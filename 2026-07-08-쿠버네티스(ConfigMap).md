```bash
---------------------------------------
vi mysqlconf

MYSQL_ROOT_PASSWORD=It12345!
MYSQL_DATABASE=word
MYSQL_USER=sgm
MYSQL_PASSWORD=It12345!
---------------------------------------

kubectl create configmap mysqlenv --from-env-file mysqlconf

kubectl get configmaps
kubectl get configmaps mysqlenv -o yaml
---------------------------------------
vi mysql.yml

apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: mysql
    env: devel
spec:
  containers:
  - name: m1
    image: mysql:8.0
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 3306
    envFrom:
    - configMapRef:
        name: mysqlenv

kubectl apply -f mysql.yml

kubectl exec -it mysql -- mysql -uroot -p'It12345!' -e "SELECT user, host FROM mysql.user;"

kubectl expose --name svc-mysql pod mysql --port 3306
```

```bash
---------------------------------------
vi wordconf

WORDPRESS_DB_HOST=svc-mysql
WORDPRESS_DB_NAME=word
WORDPRESS_DB_USER=sgm
WORDPRESS_DB_PASSWORD=It12345!

kubectl get configmaps
kubectl get configmaps wordenv -o yaml
---------------------------------------
vi word.yml

apiVersion: v1
kind: Pod
metadata:
  name: word
  labels:
    app: wordpress
    env: devel
spec:
  containers:
    - name: w1
      image: wordpress
      imagePullPolicy: Never
      ports:
      - containerPort: 80
      envFrom:
        - configMapRef:
            name: wordenv
```

```yaml
---------------------------------------
vi mysqlcon.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: mysqlenv
data:
  MYSQL_ROOT_PASSWORD: "It12345!"
  MYSQL_DATABASE: "word"
  MYSQL_USER: "sgm"
  MYSQL_PASSWORD: "It12345!"
  
kubectl apply -f mysqlcon.yml

---------------------------------------
vi wordcon.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: wordenv
data:
  WORDPRESS_DB_HOST: "svc-mysql"
  WORDPRESS_DB_NAME: "word"
  WORDPRESS_DB_USER: "sgm"
  WORDPRESS_DB_PASSWORD: "It12345!"

kubectl apply -f wordcon.yml

---------------------------------------
kubectl get configmaps
```

```bash
kubectl delete pods --all
```


---
# Kubernetes ConfigMap + Deployment + Service 실습

## 문제

1. configmap 파일 2개를 작성합니다. (yaml)
   1.1. wordenv(wordpress 환경변수), mysqlenv(mysql 환경변수)를 각각 작성합니다.
2. mysql pod를 생성해서 clusterip로 노출합니다.
3. wordpress를 deployment로 복제본을 5개 생성합니다.
4. wordpress는 nodeport로 외부에 공개합니다. 30000번 사용합니다.
5. 실제 PC에서 각각의 hostip:nodeport로 접속해 봅니다.

## 답

### 1. ConfigMap 2개 작성

**mysqlenv.yml**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysqlenv
data:
  MYSQL_ROOT_PASSWORD: "It12345!"
  MYSQL_DATABASE: "word"
  MYSQL_USER: "sgm"
  MYSQL_PASSWORD: "It12345!"
```

**wordenv.yml**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordenv
data:
  WORDPRESS_DB_HOST: "svc-mysql"
  WORDPRESS_DB_NAME: "word"
  WORDPRESS_DB_USER: "sgm"
  WORDPRESS_DB_PASSWORD: "It12345!"
```

```bash
kubectl apply -f mysqlenv.yml
kubectl apply -f wordenv.yml
kubectl get configmaps
```

### 2. MySQL Pod 생성 + ClusterIP 노출

**mysql.yml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: mysql
    env: devel
spec:
  containers:
  - name: m1
    image: mysql:8.0
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 3306
    envFrom:
    - configMapRef:
        name: mysqlenv
```

```bash
kubectl apply -f mysql.yml
kubectl expose --name svc-mysql pod mysql --port 3306
kubectl get svc
```

### 3. WordPress Deployment 생성 (복제본 5개)

**word.yml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 5
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: w1
        image: wordpress
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: wordenv
```

```bash
kubectl apply -f word.yml
kubectl get deployment
kubectl get po -o wide       # Pod 5개가 여러 노드에 분산됐는지 확인
```

### 4. WordPress NodePort 노출 (30000번)

**word-svc.yml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-word
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000
```

```bash
kubectl apply -f word-svc.yml
kubectl get svc -o wide
```

### 5. 실제 PC에서 접속

```bash
kubectl get nodes -o wide    # 각 노드 IP 확인
```

| 노드 | IP |
|------|-----|
| master | 10.0.0.11 |
| node1 | 10.0.0.12 |
| node2 | 10.0.0.13 |
| node3 | 10.0.0.14 |

실제 PC 브라우저에서 아무 노드 IP로 접속:
```
http://10.0.0.12:30000
```


## 삭제
```bash
kubectl delete pod --all
kubectl delete deployment --all
kubectl delete svc --all
kubectl delete configmap --all
```

---
```yaml
vi indexdata.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: index
data:
  babo.html: |
    <html>
    <body>
    <h1>SGM-CONFIGMAP-BABO</h1>
    </body>
    </html>
  coco.html: |
    <html>
    <body>
    <h1>SGM-CONFIGMAP-APACHE</h1>
    </body>
    </html>

kubectl apply -f indexdata.yml
---
vi npod.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: n1
    image: nginx
    imagePullPolicy: Never
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html/index.html
      subPath: babo.html
      name: sgm-vol
  volumes:
  - name: sgm-vol
    configMap:
      name: index
---
vi apod.yml

apiVersion: v1
kind: Pod
metadata:
  name: apache
spec:
  containers:
  - name: h1
    image: httpd
    imagePullPolicy: Never
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/local/apache2/htdocs/index.html
      subPath: coco.html
      name: sgm-vol
  volumes:
  - name: sgm-vol
    configMap:
      name: index

kubectl apply -f npod.yml
kubectl apply -f apod.yml
kubectl get po -o wide
lynx <nginx-pod-ip>
lynx <apache-pod-ip>
```



---
```yaml
vi sqlsec.yml

apiVersion: v1
kind: Secret
metadata:
  name: sqlsec
data:
  MYSQL_ROOT_PASSWORD: It12345!
  MYSQL_PASSWORD: It12345!
```