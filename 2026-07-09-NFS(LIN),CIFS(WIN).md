```bash
# rocky9-1~4
dnf install -y nfs-utils
```

```bash
# rocky9-1(master)
mkdir /nfs-server

vi /etc/exports
/nfs-server 10.0.0.0/24(rw,sync,no_root_squash)

systemctl start nfs-server
exportfs -v
```

```bash
# rocky9-2(node)
mkdir /nfs-client

mount -t nfs 10.0.0.11:/nfs-server /nfs-client

mount # 확인
touch /nfs-client/test.txt
```

```bash
# win10
제어판 - 프로그램 및 기능 - Windows 기능 켜기/끄기 - NFS용 서비스 - NFS용 클라이언트

# cmd
mount -o anon 10.0.0.11:/nfs-server z:

#파일 탐색기에서 확인 후 텍스트 파일 생성하려하면 안됨 그러면
#9-1에서 chmod 757 /nfs-server
```

```bash
kubectl api-resources | grep per*

vi pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: sgm-pv
spec:
  storageClassName: sgm
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /web
  
kubectl apply -f pv.yml
-----------------------------------------------
vi pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sgm-pvc
spec:
  storageClassName: sgm
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
      
kubectl apply -f pvc.yml
-----------------------------------------------
vi nginx.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: n1
    image: nginx
    imagePullPolicy: Never
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html/
      name: sgm-stor
  volumes:
  - name: sgm-stor
    persistentVolumeClaim:
      claimName: sgm-pvc

kubectl apply -f nginx.yml
```

```bash
curl
```