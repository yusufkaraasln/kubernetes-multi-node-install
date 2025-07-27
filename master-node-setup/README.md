# Kubeadm ile Multi-Node Kubernetes Cluster Kurulumu

## Master Node (Control Plane) Kurulum Dokümanı

---

### 1. Ön Koşullar
- İşletim Sistemi: Ubuntu 22.04 veya benzeri Linux dağıtımı  
- Root (sudo) yetkisi olan kullanıcı  
- Network ve firewall ayarlarının yapılmış olması (6443, 2379-2380 portları açık olmalı)  
- Hostname doğru ayarlanmış ve DNS çalışıyor olmalı  

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
- **kubectl**: Kubernetes komut satırı aracı (yönetim için kullanılır)  

---

### 5. Sistem Parametrelerinin Ayarlanması  
Master node’un sorunsuz çalışması için bazı kernel parametreleri ayarlanmalı:

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

### 7. Kubernetes Control Plane Kurulumu

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

- `--pod-network-cidr`: Flannel veya başka bir CNI kullanıyorsanız, CNI'nin pod ağ aralığına uygun ayarlanmalı.

---

### 8. Kubectl Kullanıcı Ayarları

Control plane node üzerinde kubectl komutlarını kullanabilmek için aşağıdaki adımları yapın:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### 9. Pod Network Eklentisinin (CNI) Kurulması

Örnek olarak Flannel CNI kurulumu:

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

---

### 10. Worker Node Join Token ve Komutu Alma

Worker node’ların cluster’a katılması için gerekli token ve join komutunu alın:

```bash
kubeadm token create --print-join-command
```

Bu komut size worker node’da çalıştırılacak komutu verir.

---

### 11. Son Kontroller

```bash
kubectl get nodes
```

Control plane ve worker node’ların durumunun `Ready` olduğunu görmelisiniz.

---

### 12. Notlar ve İpuçları  
- Master node’da swap kapalı olmalı (`sudo swapoff -a`). Kalıcı kapatma için `/etc/fstab` dosyasından swap satırı yorumlanmalı.  
- Worker node’lar için de benzer sistem parametreleri ayarlanmalıdır (bkz. Worker node dokümanı).  
- Master node üzerinde kubelet servisi otomatik olarak başlar ve yönetilir.  
- Network eklentisi kurulmadan podlar çalışmaz, mutlaka CNI kurun.  


---

### 13. Kaynaklar  
- [Kubernetes Resmi Dokümantasyonu](https://kubernetes.io/docs/home/)  
- [Containerd Kurulumu ve Konfigürasyonu](https://containerd.io/docs/)  
- [Network Plugin Kurulumu](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

---
