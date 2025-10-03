# วิธีการจัดการ Docker Hub Credentials แบบปลอดภัย

## 🔒 วิธีที่ 1: สร้าง Secret แยกด้วย kubectl (แนะนำสำหรับการเรียนรู้)

### สร้าง Secret
```bash
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=YOUR_USERNAME \
  --docker-password=YOUR_PASSWORD \
  --docker-email=YOUR_EMAIL@example.com
```

### ตั้งค่า values.yaml
```yaml
dockerhub:
  createSecret: false  # ไม่สร้าง secret ใน chart
```

### Deploy
```bash
helm install my-app ./my-app-chart
```

---

## 🔐 วิธีที่ 2: ใช้ Helm values แยกไฟล์

### สร้างไฟล์ secrets.yaml (เก็บแยกและไม่ commit ใน git)
```yaml
dockerhub:
  createSecret: true
  username: "your-real-username"
  password: "your-real-password"
  email: "your-real-email@example.com"
```

### Deploy ด้วย values แยก
```bash
helm install my-app ./my-app-chart -f secrets.yaml
```

### เพิ่ม secrets.yaml ใน .gitignore
```
secrets.yaml
*.secret.yaml
```

---

## 🚀 วิธีที่ 3: ใช้ External Secrets Operator (Production Level)

### 1. ติดตั้ง External Secrets Operator
```bash
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets-system --create-namespace
```

### 2. สร้าง SecretStore (ตัวอย่างใช้กับ AWS Secrets Manager)
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
```

### 3. สร้าง ExternalSecret
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: dockerhub-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: dockerhub-secret
    creationPolicy: Owner
    template:
      type: kubernetes.io/dockerconfigjson
      data:
        .dockerconfigjson: |
          {
            "auths": {
              "https://index.docker.io/v1/": {
                "username": "{{ .username }}",
                "password": "{{ .password }}",
                "email": "{{ .email }}",
                "auth": "{{ printf "%s:%s" .username .password | b64enc }}"
              }
            }
          }
  data:
  - secretKey: username
    remoteRef:
      key: dockerhub-credentials
      property: username
  - secretKey: password
    remoteRef:
      key: dockerhub-credentials
      property: password
  - secretKey: email
    remoteRef:
      key: dockerhub-credentials
      property: email
```

---

## 📝 วิธีที่ 4: ใช้ Sealed Secrets

### 1. ติดตั้ง Sealed Secrets Controller
```bash
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml
```

### 2. สร้าง secret และ encrypt
```bash
# สร้าง secret file
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=YOUR_USERNAME \
  --docker-password=YOUR_PASSWORD \
  --docker-email=YOUR_EMAIL@example.com \
  --dry-run=client -o yaml > dockerhub-secret.yaml

# Encrypt secret
kubeseal -f dockerhub-secret.yaml -w dockerhub-sealed-secret.yaml

# Deploy sealed secret
kubectl apply -f dockerhub-sealed-secret.yaml
```

---

## 💡 เทคนิคเพิ่มเติม

### 1. ใช้ Environment Variables
```bash
export DOCKERHUB_USERNAME="your-username"
export DOCKERHUB_PASSWORD="your-password"
export DOCKERHUB_EMAIL="your-email@example.com"

helm install my-app ./my-app-chart \
  --set dockerhub.username="$DOCKERHUB_USERNAME" \
  --set dockerhub.password="$DOCKERHUB_PASSWORD" \
  --set dockerhub.email="$DOCKERHUB_EMAIL" \
  --set dockerhub.createSecret=true
```

### 2. ใช้ HashiCorp Vault
```yaml
# vault-secret.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "external-secrets"
```

---

## ⚡ Quick Start แนะนำ

สำหรับการเรียนรู้ แนะนำใช้วิธีที่ 1:

```bash
# 1. สร้าง secret ก่อน
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=YOUR_USERNAME \
  --docker-password=YOUR_PASSWORD \
  --docker-email=YOUR_EMAIL@example.com

# 2. Deploy chart
helm install my-app ./my-app-chart

# 3. ตรวจสอบ
kubectl get secrets
kubectl get pods
```