##### DR(Disaster Recovery, 재해 복구)
- **목적**: 재해 발생 시 서비스 중단을 최소화하고 빠른 복구
- 주요 요인: 자연재해(지진, 홍수), 하드웨어 장애, 사이버 공격 등
- 구성 요소: 데이터 소산(원격지 저장), 이중화, 백업 시스템 등
- 관련 개념:
    - HA (High Availability, 고가용성): 시스템이 멈추지 않고 지속적으로 운영되는 기능
    - 백업(Backup): 데이터를 복사하여 보관하는 것

# Azure Load Balancer
___
- 실습 내용: Azure Cloud 기반 RHEL 9 VM 3대 + Standard Public Load Balancer(80) + Azure Bastion(SSH)를 Azure CLI + cloud-init

- VM 3대는 Public IP 없음(완전 Private) 
- 외부 웹은 Standard Public Load Balancer의 Public IP:80 으로만 접근 
- SSH 접속은 Azure Bastion 통해서만 
- cloud-init로 Nginx 자동 설치/기동 + VM별 고유 페이지(호스트명/사설IP 표시)
## 사전 준비
___
### 1) Azure ssh 확장 모듈 설치
```bash
az extension add -n ssh
az extension list -o table | egrep -i 'bastion|ssh' # 설치 확인
```

### 2) Azure Bastion 확장 모듈 설치
```bash
az extension add -n bastion
az extension list -o table | grep bastion # 설치 확인
```

### 3)확장 자동 설치 설정(강력 권장)
`교육/실습 환경에서는 이 설정을 꼭 해두세요.`
```bash
az config set extension.use_dynamic_install=yes_without_prompt
az config set extension.dynamic_install_allow_preview=true
```
___
### 1) cloud-init: vi cloud-init-rhel9-nginx.yml
`VM 3대에 공통 적용됩니다(호스트명/사설IP를 페이지에 출력).`
```bash
#cloud-config
package_update: true
package_upgrade: true
packages:
  - nginx
  - firewalld 

runcmd:
  - [ bash, -lc, "systemctl enable --now firewalld" ]
  - [ bash, -lc, "systemctl enable --now nginx" ]
  - [ bash, -lc, "firewall-cmd --permanent --add-service=http" ]
  - [ bash, -lc, "firewall-cmd --permanent --add-service=ssh" ]
  - [ bash, -lc, "firewall-cmd --reload" ]
  
  - [ bash, -lc, "cat > /usr/share/nginx/html/index.html <<'EOF'\n<html>\n<head><title>RHEL9 Nginx via cloud-init</title></head>\n<body>\n  <h1>OK - Nginx is running</h1>\n  <p><b>Hostname:</b> __HOSTNAME__</p>\n  <p><b>Private IP:</b> __PRIVATE_IP__</p>\n  <p><b>Time:</b> __TIME__</p>\n</body>\n</html>\nEOF\nchmod 0644 /usr/share/nginx/html/index.html\nchown root:root /usr/share/nginx/html/index.html" ]

  - [ bash, -lc, "HN=$(hostname -f 2>/dev/null || hostname); sed -i \"s/__HOSTNAME__/${HN}/g\" /usr/share/nginx/html/index.html" ]
  - [ bash, -lc, "IP=$(ip -4 -o addr show scope global | awk '{print $4}' | head -n1 | cut -d/ -f1); sed -i \"s/__PRIVATE_IP__/${IP}/g\" /usr/share/nginx/html/index.html" ]
  - [ bash, -lc, "TS=$(date -Is); sed -i \"s/__TIME__/${TS}/g\" /usr/share/nginx/html/index.html" ]

  - [ bash, -lc, "systemctl reload nginx || systemctl restart nginx" ]
```

### 2) All-in-One 배포 스크립트: vi deploy-rhel9-3vm-lb-bastion.sh
```bash
#!/usr/bin/env bash
set -euo pipefail
 
# ===== Variables (provided style) =====
RG="prof02-rg-LB-bastion"
LOC="koreacentral"
  
VNET="vnet-lab"
SUBNET="snet-web"
NSG="nsg-web"
  
ADMINUSER="azureuser"
IMAGE="RedHat:RHEL:9-lvm-gen2:latest"
VM_SIZE="Standard_B2s"
CINIT="cloud-init-rhel9-nginx.yml"
SSH_PUBKEY="${HOME}/.ssh/id_ed25519.pub"
 
# Network CIDR
VNET_CIDR="10.10.0.0/16"
SUBNET_CIDR="10.10.10.0/24"
  
# Bastion
BASTION_SUBNET="AzureBastionSubnet" #이름은 AzureBastionSubnet으로 고정되어 있음
BASTION_SUBNET_CIDR="10.10.250.0/26"
BASTION_NAME="${VNET}-bastion"
BASTION_PIP="pip-bastion"
 
# Load Balancer (Standard Public)
LB_NAME="lb-web"
LB_PIP="pip-lb-web"
LB_FE="fe-web"
LB_BE="be-web"
LB_PROBE="probe-http"
LB_RULE="rule-http-80"
  
# VM naming (3대)
VM_PREFIX="rhel9-web"
VM_COUNT=3
 
echo "[0/9] Ensure SSH key exists: ${SSH_PUBKEY}"
if [[ ! -f "${SSH_PUBKEY}" ]]; then
  ssh-keygen -t ed25519 -f "${HOME}/.ssh/id_ed25519" -N ""
fi
  
echo "[1/9] Create RG"
az group create -n "$RG" -l "$LOC" -o none
  
echo "[2/9] Accept Marketplace terms (RHEL image) (idempotent)"
URN="$IMAGE"
az vm image terms accept --urn "$URN" -o none || true
  
echo "[3/9] Create VNet/Subnets (web + bastion)"
az network vnet create \
  -g "$RG" -n "$VNET" -l "$LOC" \
  --address-prefix "$VNET_CIDR" \
  --subnet-name "$SUBNET" --subnet-prefix "$SUBNET_CIDR" -o none
  
# Bastion subnet must be named exactly AzureBastionSubnet
az network vnet subnet create \
  -g "$RG" --vnet-name "$VNET" \
  -n "$BASTION_SUBNET" --address-prefixes "$BASTION_SUBNET_CIDR" -o none
  
WEB_SUBNET_ID=$(az network vnet subnet show -g "$RG" --vnet-name "$VNET" -n "$SUBNET" --query id -o tsv)
BASTION_SUBNET_ID=$(az network vnet subnet show -g "$RG" --vnet-name "$VNET" -n "$BASTION_SUBNET" --query id -o tsv)
  
echo "[4/9] Create NSG (Bastion SSH only, LB HTTP only)"
az network nsg create -g "$RG" -n "$NSG" -l "$LOC" -o none
  
# Allow SSH(22) only from Bastion subnet
az network nsg rule create \
  -g "$RG" --nsg-name "$NSG" -n "allow-ssh-from-bastion" \
  --priority 1000 --access Allow --direction Inbound --protocol Tcp \
  --source-address-prefixes "$BASTION_SUBNET_CIDR" --source-port-ranges "*" \
  --destination-address-prefixes "*" --destination-port-ranges 22 -o none
 

# Allow HTTP(80) only from Azure Load Balancer (service tag)
az network nsg rule create \
  -g "$RG" --nsg-name "$NSG" -n "allow-http-from-lb" \
  --priority 1010 --access Allow --direction Inbound --protocol Tcp \
  --source-address-prefixes "AzureLoadBalancer" --source-port-ranges "*" \
  --destination-address-prefixes "*" --destination-port-ranges 80 -o none
  
# NSG에 80을 Internet로 허용 추가
az network nsg rule create \
  -g "$RG" --nsg-name "$NSG" -n allow-http-from-internet \
  --priority 1020 --access Allow --direction Inbound --protocol Tcp \
  --source-address-prefixes Internet --source-port-ranges "*" \
  --destination-address-prefixes "*" --destination-port-ranges 80
  
# (선택) 운영 점검용: VNet 내부에서 80 허용하고 싶으면 주석 해제
# az network nsg rule create \
#   -g "$RG" --nsg-name "$NSG" -n "allow-http-from-vnet" \
#   --priority 1020 --access Allow --direction Inbound --protocol Tcp \
#   --source-address-prefixes "VirtualNetwork" --source-port-ranges "*" \
#   --destination-address-prefixes "*" --destination-port-ranges 80 -o none
  
echo "[5/9] Create Standard Public IP for Load Balancer"
az network public-ip create \
  -g "$RG" -n "$LB_PIP" -l "$LOC" \
  --sku Standard --allocation-method Static -o none
  
echo "[6/9] Create Standard Public Load Balancer (frontend + backend + probe + rule)"
az network lb create \
  -g "$RG" -n "$LB_NAME" -l "$LOC" \
  --sku Standard \
  --public-ip-address "$LB_PIP" \
  --frontend-ip-name "$LB_FE" \
  --backend-pool-name "$LB_BE" -o none
  
az network lb probe create \
  -g "$RG" --lb-name "$LB_NAME" -n "$LB_PROBE" \
  --protocol Tcp --port 80 -o none
  
az network lb rule create \
  -g "$RG" --lb-name "$LB_NAME" -n "$LB_RULE" \
  --protocol Tcp --frontend-port 80 --backend-port 80 \
  --frontend-ip-name "$LB_FE" \
  --backend-pool-name "$LB_BE" \
  --probe-name "$LB_PROBE" \
  --idle-timeout 4 --enable-tcp-reset true -o none
  
echo "[7/9] Create 3 NICs (no Public IP) + attach to NSG + LB backend pool"
for i in $(seq 1 "$VM_COUNT"); do
  NIC_NAME="nic-${VM_PREFIX}-${i}"
  
  az network nic create \
    -g "$RG" -n "$NIC_NAME" -l "$LOC" \
    --subnet "$WEB_SUBNET_ID" \
    --network-security-group "$NSG" -o none
  
  # Add NIC IP config to LB backend pool
  az network nic ip-config address-pool add \
    -g "$RG" --nic-name "$NIC_NAME" \
    --ip-config-name "ipconfig1" \
    --lb-name "$LB_NAME" \
    --address-pool "$LB_BE" -o none
done
  
echo "[8/9] Create 3 RHEL9 VMs with cloud-init (no Public IP)"
for i in $(seq 1 "$VM_COUNT"); do
  VM_NAME="${VM_PREFIX}-${i}"
  NIC_NAME="nic-${VM_PREFIX}-${i}"
  
  az vm create \
    -g "$RG" -n "$VM_NAME" -l "$LOC" \
    --image "$IMAGE" \
    --size "$VM_SIZE" \
    --admin-username "$ADMINUSER" \
    --ssh-key-values "$SSH_PUBKEY" \
    --nics "$NIC_NAME" \
    --custom-data "$CINIT" \
    --public-ip-address "" \
    -o none
done
  
echo "[9/9] Create Azure Bastion (Standard) for SSH access"
az network public-ip create \
  -g "$RG" -n "$BASTION_PIP" -l "$LOC" \
  --sku Standard --allocation-method Static -o none
  
az network bastion create \
  -g "$RG" -n "$BASTION_NAME" -l "$LOC" \
  --vnet-name "$VNET" \
  --public-ip-address "$BASTION_PIP" \
  --sku Standard \
  --enable-tunneling true \
  -o none
  
LB_PUBLIC_IP=$(az network public-ip show -g "$RG" -n "$LB_PIP" --query ipAddress -o tsv)
echo
echo "===== DONE ====="
echo "Load Balancer Public IP: ${LB_PUBLIC_IP}"
echo "Web URL: http://${LB_PUBLIC_IP}"
echo
echo "Bastion name: ${BASTION_NAME}"
echo "VMs:"

for i in $(seq 1 "$VM_COUNT"); do
  echo "  - ${VM_PREFIX}-${i}"
done

echo ===========================================
echo ===== RHEL 9 VM 서버 접속 방법
echo ===== rhel9-web-1 VM 이름만 변경하여 재 접속
echo ===========================================
export RG="prof02-rg-LB-bastion"
export VM_PREFIX="rhel9-web"
export VNET="vnet-lab"
export BASTION_NAME="${VNET}-bastion"
export ADMINUSER="azureuser"
export VMID=$(az vm show -g "$RG" -n "${VM_PREFIX}-1" --query id -o tsv)
  
cat <<EOF
# 1) 대상 VM 리소스 ID 조회
VMID=\$(az vm show -g "$RG" -n "${VM_PREFIX}-1" --query id -o tsv)
  
# 2) Bastion 통해 SSH 접속 (VM 이름만 -1/-2/-3로 변경)
az network bastion ssh \\
  -g "$RG" \\
  -n "$BASTION_NAME" \\
  --target-resource-id "\$VMID" \\
  --auth-type ssh-key \\
  --username "$ADMINUSER" \\
  --ssh-key "${HOME}/.ssh/id_ed25519"
EOF
```

### 두 파일 작성 후
```bash
chmod +x deploy-rhel9-3vm-lb-bastion.sh
./deploy-rhel9-3vm-lb-bastion.sh

배포 후 브라우저에서:
- `http://<LB_PUBLIC_IP>` 접속
새로고침할 때마다 `Hostname/Private IP`가 바뀌면 LB 분산이 정상
```

### 로드밸런싱 되는지 확인
##### 1)curl로 새 연결 강제 (가장 정확)
```bash
for i in {1..20}; do
  curl -s http://20.194.33.19 | grep -E "Hostname|Private"
  sleep 1
done
```

##### 2) SSH 로그인 방법
`vi ssh-connect.sh`
```bash
#!/usr/bin/env bash

set -euo pipefail
  
export RG="prof02-rg-LB-bastion"
export VM_PREFIX="rhel9-web"
export BASTION_NAME="vnet-lab-bastion"
export ADMINUSER="azureuser"
  
SSH_KEY="${HOME}/.ssh/id_ed25519"
[[ -f "$SSH_KEY" ]] || { echo "ERROR: SSH key not found: $SSH_KEY" >&2; exit 1; }
  
for i in 1 2 3; do
  VMNAME="${VM_PREFIX}-${i}"
  echo "▶ VMNAME=$VMNAME"
 
  VMID="$(az vm show -g "$RG" -n "$VMNAME" --query id -o tsv)"
  echo "▶ Connecting to $VMNAME"
  
  az network bastion ssh \
    -g "$RG" \
    -n "$BASTION_NAME" \
    --target-resource-id "$VMID" \
    --auth-type ssh-key \
    --username "$ADMINUSER" \
    --ssh-key "$SSH_KEY"
done
```
### 파일 작성 후
```bash
chmod +x ssh-connect.sh
./ssh-connect.sh
```

##### 3) Bastion Tunnel (로컬 포트 포워딩)
```bash
RG="prof02-rg-LB-bastion"
BASTION_NAME="vnet-lab-bastion"
VM_PREFIX="rhel9-web"
VMID=$(az vm show -g $RG -n ${VM_PREFIX}-1 --query id -o tsv)
  
az network bastion tunnel \
  -g $RG \
  -n $BASTION_NAME \
  --target-resource-id "$VMID" \
  --resource-port 22 \
  --port 2221

# 다른 터미널에서
ssh -p 2221 -i ~/.ssh/id_ed25519 azureuser@127.0.0.1
```


##### 대상 VM 리소스 ID 조회
`VMID="$(az vm show -g "st602-rg-LB-bastion" -n "rhel9-web-1" --query id -o tsv)"`

az group delete -n [st613-busan-rg] --yes --no-wait

















---
```bash
az vm list-skus -l koreacentral \
  --resource-type virtualMachines \
  -o table
# Microsoft Azure에서 `koreacentral` 리전에 생성 가능한 가상 머신(VM) SKU 목록을 조회하여 표 형식으로 출력
```








web-3  > root > mkdir /home/data1> cd /usr/share/nginx/html > tar cvf - * | (cd /home/data1; tar xvf -) > cd /home/data1 백업확인