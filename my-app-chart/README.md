# My App Chart - Helm Chart สำหรับเรียนรู้การ Deploy จาก Private Docker Hub

Helm chart นี้สร้างขึ้นเพื่อการเรียนรู้การ deploy แอปพลิเคชันจาก private Docker Hub registry

## ข้อกำหนดเบื้องต้น

- Kubernetes cluster ที่ใช้งานได้
- Helm 3.x ติดตั้งแล้ว
- Docker Hub account และ private repository

## การใช้งาน

### 1. แก้ไขไฟล์ `values.yaml`

ก่อนการ deploy คุณจะต้องแก้ไข `values.yaml` ให้ตรงกับข้อมูลของคุณ:

```yaml
image:
  repository: "your-dockerhub-username/your-app-name"
  tag: "latest"

dockerhub:
  username: "your-dockerhub-username"
  password: "your-dockerhub-password"
  email: "your-email@example.com"
```

### 2. Install Chart

```bash
# Install chart
helm install my-app ./my-app-chart

# หรือ upgrade ถ้ามีอยู่แล้ว
helm upgrade --install my-app ./my-app-chart
```

### 3. ตรวจสอบสถานะ

```bash
# ดู pods
kubectl get pods

# ดู services
kubectl get services

# ดู deployment
kubectl get deployments
```

### 4. การทดสอบ

```bash
# Port forward เพื่อทดสอบ
kubectl port-forward service/my-app-my-app-chart 8080:80

# เปิด browser ไปที่ http://localhost:8080
```

## โครงสร้างไฟล์

```
my-app-chart/
├── Chart.yaml              # ข้อมูล metadata ของ chart
├── values.yaml             # ค่าเริ่มต้นและการตั้งค่า
├── templates/
│   ├── _helpers.tpl        # Template helpers
│   ├── deployment.yaml     # Kubernetes Deployment
│   ├── service.yaml        # Kubernetes Service
│   └── secret.yaml         # Docker registry secret
└── README.md              # คำแนะนำการใช้งาน
```

## การปรับแต่ง

### Environment Variables

สามารถเพิ่ม environment variables ใน `values.yaml`:

```yaml
env:
  - name: NODE_ENV
    value: "production"
  - name: DATABASE_URL
    value: "your-database-url"
```

### Resource Limits

ปรับ CPU และ Memory limits:

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

### Scaling

เปิดใช้งาน auto-scaling:

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

## การ Uninstall

```bash
helm uninstall my-app
```

## 🔒 Security - การจัดการ Docker Hub Credentials

**⚠️ สำคัญ**: อย่าเก็บ Docker Hub credentials ใน `values.yaml` ในระบบ production!

### วิธีที่ปลอดภัย (แนะนำ):

```bash
# สร้าง secret แยกด้วย kubectl
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=YOUR_USERNAME \
  --docker-password=YOUR_PASSWORD \
  --docker-email=YOUR_EMAIL@example.com

# ตั้งค่าใน values.yaml
dockerhub:
  createSecret: false  # ใช้ secret ที่มีอยู่แล้ว
```

ดูรายละเอียดเพิ่มเติมใน [SECURITY.md](./SECURITY.md)

## หมายเหตุสำคัญ

1. **Security**: ใช้วิธีการที่ปลอดภัยตามที่อธิบายข้างต้น
2. **Image Tags**: ใช้ specific tags แทน `latest` ใน production
3. **Health Checks**: ปรับ liveness และ readiness probes ให้เหมาะกับแอปพลิเคชันของคุณ

## การเรียนรู้เพิ่มเติม

- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Hub Private Repositories](https://docs.docker.com/docker-hub/repos/#private-repositories)