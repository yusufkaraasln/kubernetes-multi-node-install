# Kubeadm ile Multi-Node Kubernetes Cluster Kurulumu

## Worker Node (worker1) Kurulum Dokümanı

---

### 1. Ön Koşullar
- İşletim Sistemi: Ubuntu 22.04 veya benzeri Linux dağıtımı  
- Root (sudo) yetkisi olan kullanıcı  
- Kubernetes Control Plane Node (Master) kurulmuş ve çalışır durumda  
- Control Plane IP adresi ve join komut bilgisi hazır  
- Network ve firewall ayarlarının yapılmış olması (6443 portu açık olmalı)  

---

### 2. Sistem Güncellemeleri ve Temel Paketler

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl
```

---

### 3. Kubernetes Deposu ve GPG Anahtarının Eklenmesi

```bash
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \\
gpg --dearmor | sudo tee /etc/apt/keyrings/kubernetes-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | \\
sudo tee /etc/apt/sources.list.d/kubernetes.list
```

---

### 4. Kubernetes Bileşenlerinin Kurulması

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

- **kubelet**: Node üzerinde Kubernetes podlarını çalıştıran servis  
- **kubeadm**: Cluster kurulumu ve yönetimi için araç  
- **kubectl**: Kubernetes komut satırı aracı (isteğe bağlı, yönetim için kullanılır)  

---

### 5. Sistem Parametrelerinin Ayarlanması  
Worker node’un sorunsuz çalışması için bazı kernel parametreleri ayarlanmalı:

#### 5.1. Modülün yüklenmesi:

```bash
echo "br_netfilter" | sudo tee /etc/modules-load.d/br_netfilter.conf
sudo modprobe br_netfilter
```

#### 5.2. Sysctl ayarlarının yapılması:

```bash
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

---

### 6. Container Runtime Kurulumu (containerd)  
Kubernetes için container runtime gereklidir, en yaygın kullanılan containerd'dir.

#### 6.1. containerd kurulumu

```bash
sudo apt install -y containerd
sudo systemctl enable containerd
sudo systemctl start containerd
```

#### 6.2. containerd konfigürasyonu

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

---

### 7. Node’un Cluster’a Katılması  
Control Plane (master) node’dan aldığınız kubeadm join komutunu root yetkisiyle worker node üzerinde çalıştırın:

```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash <HASH>
```

Örnek:

```bash
sudo kubeadm join 192.168.64.4:6443 --token 7comyz.kpt71hkorav1kwrt --discovery-token-ca-cert-hash sha256:b81467d2a8c931dadbcedcdfd8c38557bdc37ff05bc51e6afa8300e290625561
```

- Token: Control plane’dan `kubeadm token create` komutu ile alınabilir  
- Discovery-token-ca-cert-hash: openssl veya control plane üzerindeki kubeadm komutları ile alınır  

---

### 8. Son Kontroller  
Control Plane node’dan aşağıdaki komut ile worker node’un başarıyla katıldığını kontrol edin:

```bash
kubectl get nodes
```

Worker node’un durumunun `Ready` olduğunu görmelisiniz.

---

### 9. Notlar ve İpuçları  
- Worker node’un ip_forward ayarının 1 olduğundan emin olun, aksi halde join sırasında hata alınabilir.  
- swap kapalı olmalı (`sudo swapoff -a`). Kalıcı kapatma için `/etc/fstab` dosyasından swap satırı yorumlanmalı.  
- Worker node üzerinde `kubectl` kullanımı opsiyoneldir, ancak kurulumu tavsiye edilir.  
- Her node’da kubelet servisi otomatik olarak başlar ve yönetilir.  


---

### 10. Kaynaklar  
- Kubernetes resmi dokümantasyonu  
- Containerd kurulumu ve konfigürasyonu  
- Network Plugin kurulumu  

---


