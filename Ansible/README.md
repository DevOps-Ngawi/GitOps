# Setup Ansible GitOps Lab

Setup ini dipakai dari VM control untuk melakukan provisioning ke VM managed host AlmaLinux aarch64. Target default dibuat untuk VM managed host 8 GB RAM dan 6 core.

## Topologi

- VM control: menjalankan Ansible dan playbook.
- VM managed host: target provisioning Kubernetes single-node dengan Docker, Minikube, kubectl, Helm, dan Argo CD.
- Arsitektur: aarch64/arm64.

## 1. Siapkan VM control

Install Ansible di VM control:

```bash
sudo dnf install -y ansible-core python3
```

Buat SSH key jika belum ada:

```bash
ssh-keygen -t ed25519 -C "ansible-control"
```

Copy key ke VM managed host:

```bash
ssh-copy-id RevanArturito@192.168.56.2
```

Pastikan user target punya akses sudo:

```bash
ssh RevanArturito@192.168.56.2 "sudo whoami"
```

## 2. Sesuaikan inventory

Edit `inventory.ini` jika IP atau user VM managed host berbeda:

```ini
[managed]
k8s-node ansible_host=192.168.56.2 ansible_user=RevanArturito ansible_become=true
```

## 3. Jalankan provisioning

Dari folder `Ansible`:

```bash
ansible all -m ping
ansible-playbook playbook.yml
```

Playbook akan mengalokasikan Minikube dengan default:

- CPU: 4 core
- Memory: 6144 MB
- Driver: Docker

Nilai ini disimpan di `group_vars/all.yml` dan aman untuk VM target 8 GB/6 core karena masih menyisakan resource untuk OS host.

## Override cepat

Contoh jika ingin memberi resource lebih besar:

```bash
ansible-playbook playbook.yml -e minikube_cpus=5 -e minikube_memory=7168
```

Contoh jika Docker/Minikube sudah terpasang dan hanya ingin bootstrap ulang Kubernetes/Argo CD:

```bash
ansible-playbook playbook.yml -e skip_minikube_install=true
```

## Setelah selesai

Cek cluster dari VM managed host:

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get applications -n argocd
```
