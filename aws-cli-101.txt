Langsung:

---

## 🧰 **Setup AWS CLI Login Awal (Manual Mode)**

### 1. **Pastikan AWS CLI udah terinstal**

Cek:

```bash
aws --version
```

Kalau belum:

```bash
brew install awscli      # Mac
sudo apt install awscli  # Debian/Ubuntu
```

---

### 2. **Dapatkan AWS credentials**

Login ke: [https://console.aws.amazon.com](https://console.aws.amazon.com)
Lalu buka:
**My Security Credentials → Access keys → Create new access key**

Lo bakal dapet:

* `AWS Access Key ID`
* `AWS Secret Access Key`

---

### 3. **Jalankan konfigurasi awal**

```bash
aws configure
```

Input:

```
AWS Access Key ID:      <masukkan access key>
AWS Secret Access Key:  <masukkan secret key>
Default region name:    ap-southeast-1      # contoh: Singapore
Default output format:  json                # bisa juga: text, table
```

---

### 4. **Cek apakah berhasil**

```bash
aws sts get-caller-identity
```

Kalau berhasil, lo akan lihat output berisi:

* Account ID
* ARN
* User ID

---

Oke, kalau tujuan lo **akses ke EKS cluster via AWS CLI dan `kubectl`**, maka selain `aws configure`, lo perlu setup tambahan supaya bisa:

---

## 🎯 Tujuan: **Akses EKS Cluster dari CLI**

### ✅ Syarat Minimal:

1. AWS CLI sudah dikonfigurasi (`aws configure`)
2. `kubectl` terinstal
3. `eksctl` (opsional tapi sangat disarankan)

---

## 🔧 Step-by-Step Setup

### 1. **Pastikan IAM user/role punya akses ke EKS**

User atau role lo harus punya permission seperti:

```json
{
  "Effect": "Allow",
  "Action": [
    "eks:DescribeCluster"
  ],
  "Resource": "*"
}
```

Kalau gak yakin, tes aja:

```bash
aws eks list-clusters
```

---

### 2. **Dapatkan nama cluster**

```bash
aws eks list-clusters
```

---

### 3. **Generate kubeconfig**

```bash
aws eks update-kubeconfig --region ap-southeast-1 --name <nama-cluster>
```

> Replace `ap-southeast-1` dan `<nama-cluster>` sesuai dengan output lo.

Ini akan mengupdate file:

```
~/.kube/config
```

Dan masukin credential EKS ke situ.

---

### 4. **Tes Akses via kubectl**

```bash
kubectl get nodes
```

Kalau berhasil, berarti lo udah bisa akses EKS cluster.

---

### 🛠️ Tools yang Bisa Lo Tambahkan:

* `kubectl`
  → CLI untuk akses cluster
* `eksctl`
  → CLI untuk create/manage EKS (paket lengkap)
  Install:

  ```bash
  brew install eksctl
  ```

---

Langsung ke perintah:

---

## 🔄 **Ganti Region**

```bash
aws configure set region ap-southeast-1
```

Atau sementara waktu (1x command):

```bash
AWS_REGION=us-west-2 aws eks list-clusters
```

Atau:

```bash
aws eks list-clusters --region ap-northeast-1
```

---

## 👤 **Ganti Profile (kalau punya banyak akun AWS)**

### 1. Buat profile baru:

```bash
aws configure --profile my-second-account
```

Lo bisa isi access key dan region berbeda dari profile default.

### 2. Gunakan profile tersebut:

```bash
aws eks list-clusters --profile my-second-account
```

### 3. Atur profile & region sementara:

```bash
AWS_PROFILE=my-second-account AWS_REGION=ap-northeast-1 aws eks list-clusters
```

---

## 🔧 Tips:

### Cek semua profile:

```bash
cat ~/.aws/credentials
```

### Ganti default profile secara global (opsional):

```bash
export AWS_PROFILE=my-second-account
```

Atau masukin ke `.bashrc` / `.zshrc` kalau mau permanen.

---

Kalau lo mau pakai profile ini untuk `kubectl` (akses EKS), pastikan lo tambahkan:

```bash
aws eks update-kubeconfig --name <cluster-name> --profile my-second-account
```

---

AWS CLI **tidak punya command native** seperti `gcloud config set account` di GCP yang langsung switch akun/profil. Tapi lo bisa **akali** dengan beberapa cara:

---

## 🔁 Opsi 1: Pakai ENV variable (cara paling cepat & fleksibel)

```bash
export AWS_PROFILE=my-second-account
```

Setelah itu semua command AWS CLI akan pakai profile itu. Lo bisa masukin ini ke alias, script, atau terminal session.

Untuk sementara (1x command):

```bash
AWS_PROFILE=my-second-account aws s3 ls
```

---

## ⚙️ Opsi 2: Alias cepat

Edit `.bashrc` / `.zshrc`:

```bash
alias aws-dev='export AWS_PROFILE=dev'
alias aws-prod='export AWS_PROFILE=prod'
```

Sekarang lo tinggal ketik:

```bash
aws-dev   # untuk pakai profile dev
aws-prod  # untuk pakai profile prod
```

---

## 🧪 Opsi 3: Tulis helper script (mirip `gcloud`)

Contoh script: `aws-switch.sh`

```bash
#!/bin/bash
PROFILE=$1
if [ -z "$PROFILE" ]; then
  echo "Usage: aws-switch <profile-name>"
  exit 1
fi
export AWS_PROFILE=$PROFILE
echo "Switched to AWS profile: $AWS_PROFILE"
```

Pakai:

```bash
source aws-switch.sh dev
```

> Gunakan `source` biar `export` berlaku di shell lo.

---

## 🧼 Opsi 4: Simulasi `gcloud config` behavior

Kalau lo pengen experience yang mirip `gcloud`, bisa pakai `awsume`:

### Install:

```bash
brew install awsume
```

### Pakai:

```bash
awsume my-profile
```

> Ini akan set env var + support MFA + clean up saat ganti profile.

---

### 📌 Kesimpulan:

| Cara                     | Kelebihan                        | Kekurangan                      |
| ------------------------ | -------------------------------- | ------------------------------- |
| `AWS_PROFILE=xxx`        | Cepat, langsung jalan            | Cuma untuk command itu saja     |
| `export AWS_PROFILE=xxx` | Efektif untuk satu shell session | Gak persist setelah close shell |
| `awsume`                 | Mirip `gcloud`, MFA support      | Perlu install tambahan          |
| Alias/script             | Customizable                     | Harus setup manual              |

Kalau lo mau, gue bisa buatin script switcher interaktif ala `fzf`. Mau sekalian?



