# Kubeadm ile Multi-Node Kubernetes Cluster Kurulumu - Giriş ve VM Kurulumları

---

## Giriş

Bu doküman, Kubernetes cluster kurulumu için temel ortamı hazırlamak amacıyla VM (Virtual Machine) kurulumu ve yapılandırması hakkında bilgi verir. Kubernetes kurulumu için genellikle fiziksel makineler veya bulut ortamları tercih edilir ancak yerel test ve geliştirme ortamı için VM kullanmak pratik ve yaygındır.  

Bu rehberde, Windows ve macOS kullanıcıları için **Multipass** aracı kullanılarak Ubuntu tabanlı VM'lerin nasıl kurulacağı detaylandırılmıştır. Bu dokümanda kullanacağımız VM olan Multipass, hafif ve hızlı VM yönetimi sağlayan bir araçtır. Ubuntu sanal makineleri oluşturup yönetmeyi kolaylaştırır.

---

## 1. Multipass Nedir?

- Ubuntu VM'lerini hızlıca ayağa kaldırmak için CLI ve GUI arayüzü sunan cross platform bir araçtır.
- Windows, macOS ve Linux üzerinde çalışır.
- CLI üzerinden kolay VM yönetimi sağlar.
- Kubernetes node'larınızı (master, worker) oluşturmak için ideal bir ortamdır.

---

## 2. Multipass Kurulumu

### Windows İçin

1. [Multipass Windows Installer](https://github.com/canonical/multipass/releases/latest) sayfasından en son sürümü indirin.  
2. İndirilen `.exe` dosyasını çalıştırın ve kurulum sihirbazını takip edin.  
3. Kurulum tamamlandıktan sonra PowerShell veya CMD açarak multipass komutlarının çalıştığını test edin:

```powershell
multipass version
```

### macOS İçin

1. Homebrew kullanarak kurulum yapabilirsiniz:

```bash
brew install --cask multipass
```

2. Kurulum sonrası terminal açarak multipass sürümünü kontrol edin:

```bash
multipass version
```

Alternatif olarak [Multipass macOS Installer](https://github.com/canonical/multipass/releases/latest) sayfasından .dmg dosyasını indirip yükleyebilirsiniz.

---

## 3. Ubuntu VM Oluşturma

Multipass ile Ubuntu 22.04 veya daha yeni bir sürümü kullanarak VM oluşturabilirsiniz. Örnek komut:

```bash
multipass launch --name kube-vm --mem 4G --disk 20G --cpus 2 22.04
```

- `--name`: VM ismi (ör: kube-vm, worker1, worker2)  
- `--mem`: VM için RAM miktarı  
- `--disk`: VM disk boyutu  
- `--cpus`: VM'ye atanacak CPU sayısı  
- `22.04`: Ubuntu sürümü  

---

## 4. VM'ye Bağlanma

Oluşturulan VM'ye bağlanmak için:

```bash
multipass shell kube-vm
```

---

## 5. VM Yönetimi Komutları

- VM listesini görmek:

```bash
multipass list
```

- VM durdurmak:

```bash
multipass stop kube-vm
```

- VM başlatmak:

```bash
multipass start kube-vm
```

- VM silmek:

```bash
multipass delete kube-vm
multipass purge
```

---

## 6. Notlar

- VM kaynak ayarlarını (RAM, CPU) ihtiyaçlarınıza göre ayarlayın.  
- Multipass hızlı test ortamları için uygundur, üretim için gerçek sunucular veya bulut platformları tercih edilmelidir.  
- VM'lerde port yönlendirme ve firewall ayarlarını kontrol etmeyi unutmayın.



---

## 7. Sonraki Adımlar

Bu VM'ler üzerinde Kubernetes kurulumu için aşağıdaki dokümanları kullanabilirsiniz:

- [Master Node Kurulumu](./master-node-setup/README.md)  
- [Worker-1 Node Kurulumu](./worker-1-node-setup/README.md)  
- [Worker-2 Node Kurulumu](./worker-2-node-setup/README.md)  

  
---



