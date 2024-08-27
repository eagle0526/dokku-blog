---
sidebar_position: 2
---

## Google Kubernetes Engine
* 此教學為如何 deploy 自己的專案到 GKE 上面

## 第一步驟 - 準備檔案
首先一樣先把這份檔案給拉下來：

### 目前檔案架構
```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- **init**.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
        | -- base.py
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
  |  requirements.txt
  |  app.yaml
  |  env
```

## 第二步驟 - 新增 deployment.yaml 和 service.yaml 檔案

分別來介紹一下，這兩份檔案目的和作用是什麼

* deployment.yaml

deployment.yaml 檔案的作用是定義應用程式的部署規則和配置，主要包括以下幾個功能：

1. 定義 Pod 的設定：指定 Pod 的映像檔、環境變數、啟動命令、掛載的卷等細節。
2. 管理副本數量：確保在 Kubernetes 叢集中同時運行的 Pod 數量，這裡可以指定要運行多少個 Pod（如 replicas: 3）。
3. 滾動更新與回滾：當您更新部署時，Deployment 控制器會逐步替換舊的 Pod，確保無中斷的更新過程。如果發生問題，也可以輕鬆回滾到之前的版本。
4. 自動修復：如果某些 Pod 異常崩潰，Deployment 會自動重新啟動它們，以保持所需的副本數量。

* service.yaml

Service 是 Kubernetes 中的一種資源，主要用來定義一組 Pod 的網路服務。service.yaml 檔案的作用是將一組相關的 Pod 暴露給外部或集群內的其他服務，主要功能包括：

1. 負載均衡：Service 會自動將流量分配到後端的多個 Pod 上，實現負載均衡。
2. 固定 IP 地址：Pod 是短暫的，它們的 IP 地址可能會隨著重啟而改變，但 Service 提供了一個固定的 Cluster IP 或外部 IP 來穩定地訪問這些 Pod。
3. 多種訪問類型：您可以定義不同類型的 Service，例如 ClusterIP（內部訪問）、NodePort（通過節點 IP 訪問）、LoadBalancer（通過雲提供商的負載均衡器訪問）等。

### 新增這兩個檔案

* 檔案架構

```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- **init**.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
        | -- base.py
  |  -- k8s                        -> 新增此檔案
        | -- deployment.yaml       -> 新增此檔案
        | -- service.yaml          -> 新增此檔案
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
  |  requirements.txt
  |  app.yaml
  |  env
```

### deployment.yaml

這邊有很多參數要新增，像是 `Pod 的映像檔、環境變數、啟動命令、掛載的卷等細節`，還有名稱之類的設定，`containers` 下面那邊就全都是 `docker` 的設定

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-deployment
  namespace: default  # 確保這裡是正確的 namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fastapi
  template:
    metadata:
      labels:
        app: fastapi
    spec:
      containers:
      - name: fastapi
        image: gcr.io/fastapi-429704/fastapi:latest
        ports:
        - containerPort: 8000
        env:
        - name: USE_CLOUD_SQL
          value: "true"
        - name: CLOUD_SQL_HOST
          value: "35.201.232.244"
        - name: CLOUD_SQL_USER
          valueFrom:
            secretKeyRef:
              name: cloudsql-db-credentials
              key: username
        - name: CLOUD_SQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: cloudsql-db-credentials
              key: password
        - name: CLOUD_SQL_DB
          value: "test"
```

### service.yaml

這邊要設定 `port` 和 `LoadBalancer` 這些的參數

```yaml
apiVersion: v1
kind: Service
metadata:
  name: fastapi-service
spec:
  type: LoadBalancer
  selector:
    app: fastapi
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
```


## 第三步驟 - 新增 GKE 帳號

首先確認自己在 GCP 裡面有沒有創建專案，我現在已經創建好了， `project-id` 是 `fastapi-429704`，並且我的 `gcloud` 也已經在本機端設定好了 (已經連進該 Project 裡面)，因此我可以直接輸入以下指令：

```shell
$ gcloud container clusters create fastapi-cluster \
    --zone asia-east1-a \
    --num-nodes 3 \
    --machine-type e2-medium
```

這個指令可以創建一個名叫 `fastapi-cluster` 的 `GKE`，並且設定時區為 `asia-east1-a`，打開 `三個 node`，機器的類型為 `e2-medium`。



## 第四步驟 - 與 GKE 建立憑證

* container clusters get-credentials：這個子命令用於取得指定 Kubernetes 叢集的憑證和配置，並自動配置 kubectl 工具。
* fastapi-cluster：你所要連接的 GKE 叢集的名稱。
* --zone asia-east1-a：指定叢集所在的區域，這是 GKE 叢集部署的地理位置。

```shell
$ gcloud container clusters get-credentials fastapi-cluster --zone asia-east1-a
```
執行此行指令後，就可以使用 `kubectl` 指令跟 `fastapi-cluster` 進行互動，例如查看 Pod、部署應用程式、更新資源等。



## 第五步驟 - 建立 docker image

接著我們就來建立一個 image 檔案，等等要推上 GKE 用的，不過目前這個版本的 `image` 會有個小問題，因為 `GKE` 使用的架構是 `amd64`，但是我們目前 `python:3.9-slim` 預設的架構是 `arm64`。
```shell
$ docker build -t gcr.io/fastapi-429704/fastapi:latest .
```

因此我們修改一下指令，建立一個 `amd64` 的 image 檔案
```shell
$ docker build --platform linux/amd64 -t gcr.io/fastapi-429704/fastapi:latest . 
```


## 第六步驟 - 把 image 推上 GKE

接著就可以來把該 image 給推上雲端 (GKE) 了！
```shell
$ docker push gcr.io/fastapi-429704/fastapi:latest
```

## 第七步驟 - 建立 secret

我們需要在 `Kubernetes` 中建立一個 `secret` 來保存 `Cloud SQL` 的使用者名稱和密碼。

* YOUR_DB_USERNAME 請換成自己的 cloudSQL 的使用者名稱
* YOUR_DB_PASSWORD 請換成自己的 cloudSQL 的使用者密碼

```shell
$ kubectl create secret generic cloudsql-db-credentials \
   --from-literal=username=YOUR_DB_USERNAME \
   --from-literal=password=YOUR_DB_PASSWORD
```

### 確認 secret 是否建立成功

```shell
$ kubectl get secrets

------
NAME                      TYPE     DATA   AGE
cloudsql-db-credentials   Opaque   2      6d------
```


### 檢查剛剛建立的 secret 內容

這邊可以注意到 `data` 裡面有兩個參數，一個是 `password`，一個是 `username`，這兩個後面的值就是我們一開始用 `kubectl create secret` 指令時創建出來的值，不過可能會覺得很奇怪，看起來跟你當初輸入的值怎麼完全不一樣，沒錯，因為裡面的值是已經被轉碼過的。
```shell
$ kubectl get secret cloudsql-db-credentials -o yaml

------
apiVersion: v1
data:
  password: MDAwMDA=
  username: cm9vdA==
kind: Secret
metadata:
  creationTimestamp: "2024-08-08T06:37:36Z"
  name: cloudsql-db-credentials
  namespace: default
  resourceVersion: "101109"
  uid: 8beb2b67-f2de-4381-b2d9-6d6bd5567a45
type: Opaque
```

## 第八步驟 - 把 deployment 和 service 部署上去

把 `deployment.yaml` 和 `service.yaml` 檔案 push 上 `GKE`
```shell
$ kubectl apply -f k8s/deployment.yaml
$ kubectl apply -f k8s/service.yaml
```

## 第九步驟 - 確認部署狀況

在前面，我們已經開始部署，並且也已經把 `image` 推上去了，接著我們開始確認一下，是否部署正常，並且看一下 `pod` 有沒有正常的運作

### 查看 pod 狀態

這邊會發現我們部署發生了錯誤，因為 pod 沒有正常的運作
```shell
$ kubectl get pods

------
NAME                                  READY   STATUS             RESTARTS         AGE
fastapi-deployment-6d6595c8b6-6bqt9   0/1     CrashLoopBackOff   365 (112s ago)   32h
fastapi-deployment-6d6595c8b6-ffttb   0/1     CrashLoopBackOff   366 (87s ago)    32h
fastapi-deployment-6d6595c8b6-jspvv   0/1     CrashLoopBackOff   365 (24s ago)    32h
```


### 查看 pods 的 log 狀態

用以下指令，可以看看在部署的時間，發生了哪些問題

```shell
$ kubectl logs fastapi-deployment-6d6595c8b6-6bqt9

------
INFO:     Started server process [1]
INFO:     Waiting for application startup.
ERROR:    Traceback (most recent call last):
...省略
...省略
sqlalchemy.exc.OperationalError: (pymysql.err.OperationalError) (2003, "Can't connect to MySQL server on '35.201.232.244' (timed out)")
(Background on this error at: https://sqlalche.me/e/20/e3q8)
```

這邊就可以看出，因為我這個 `pod` 目前連不上資料庫，才導致出現錯誤




### 查看 pods 的詳細資訊

除了剛剛那個指令，我們有可以用這個指令，確認一下是哪邊卡住了，此指令非常方便，還可以確認現在你的 `image` 一些像是環境變數的資訊

```shell
$ kubectl describe pod fastapi-deployment-6d6595c8b6-6bqt9

------
Name:             fastapi-deployment-6d6595c8b6-6bqt9
Namespace:        default
Priority:         0
Service Account:  default
...省略
...省略
Events:
  Normal   Started    26s (x3 over 67s)  kubelet            Started container fastapi
  Normal   Pulled     26s                kubelet            Successfully pulled image "gcr.io/fastapi-429704/fastapi:latest" in 708ms (708ms including waiting)
  Warning  BackOff    13s (x2 over 39s)  kubelet            Back-off restarting failed container fastapi in pod fastapi-deployment-6d6595c8b6-v9vsg_default(40567dc1-874c-4978-b233-24363d15ec24)
```


:::tip
這個指令可以查到目前檔案使用的環境變數，非常方便
:::


### 查看 pods 的 External ip 

此指令可以找到每一個 pod 的外部 IP，等等我們要把這些 IP 加進 cloudSQL 的 `允許連接` IP 之中

```shell
$ kubectl get nodes -o wide 

------
NAME                                             STATUS   ROLES    AGE    VERSION               INTERNAL-IP     EXTERNAL-IP    
gke-fastapi-cluster-default-pool-15b2462e-6wcj   Ready    <none>   3d6h   v1.29.6-gke.1326000   10.140.15.203   35.229.160.224 
gke-fastapi-cluster-default-pool-15b2462e-82pg   Ready    <none>   3d5h   v1.29.6-gke.1326000   10.140.15.204   35.229.249.22  
gke-fastapi-cluster-default-pool-15b2462e-fero   Ready    <none>   3d5h   v1.29.6-gke.1326000   10.140.15.205   35.194.163.180 
```

### 取得 LoadBalancer 的外部 IP 地址

這個命令會列出所有服務的相關資訊，包括你的 FastAPI 應用程式的 EXTERNAL-IP 地址，並且對外輸入的網址就是這個 `34.80.81.185`

```shell
$ kubectl get svc

------
NAME              TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
fastapi-service   LoadBalancer   34.118.226.241   34.80.81.185   80:31950/TCP   6d3h
kubernetes        ClusterIP      34.118.224.1     <none>         443/TCP        6d4h
```


### LoadBalancer 和 Pod 的區別

* LoadBalancer 是 Kubernetes 中的一個服務，通常被用來將流量從外部引導到集群內部的 Pod。當你使用 LoadBalancer 服務時，Kubernetes 會分配一個外部 IP 地址，並將來自該 IP 的流量負載均衡到集群中的各個 Pod
* Pod 是 Kubernetes 中的最小部署單位，通常會執行應用程式的實例。每個 Pod 都會有自己的內部 IP 地址，並且這些 IP 地址通常是僅在 Kubernetes 集群內部可見的。

因此我們在設定 cloudSQL 允許連接的 ip 時，不是要新增 `LoadBalancer` 的外部 `IP`，而是要設定 `POD` 的外部 `IP` 為可以連接 `cloudSQL` 的 `IP`。  

:::tip
Pod 的 IP 地址是 Kubernetes 集群內部的，所以如果你沒有將 Pod 的外部 IP 地址也加入到 Cloud SQL 的授權網路中，Cloud SQL 將無法識別和允許來自這些 Pod 的連接。
:::

:::tip
建議 LoadBalancer 的 external IP 和 POD 所有的 external IP 都要串接，要不然如果有一天某個 pod 壞掉，會導致整個應用程式壞掉
:::


## 第十步驟 - cloudSQL 加入 pod 的外部 IP

前面我們已經找到 pod 不能正常啟動的原因，是因為資料庫沒有連接成功，因此我們現在把 pod 所有的 `external IP` 加入 `cloudSQL` 允許連接的網路之內。  
如果成功加進去之後，過個一陣子，再輸入檢查 `pod` 指令一次：

```shell
$ kubectl get pods

------
NAME
fastapi-deployment-7b9974ff89-nc94b   1/1     Running            0      3d3h
fastapi-deployment-7b9974ff89-wnr6h   1/1     Running            0      3d2h
fastapi-deployment-7b9974ff89-zv92m   1/1     Running            0      3d2h
```

可以發現 pod 都正常運作了，因此直接在網址輸入你的 `LoadBalance` 的外部 IP - `34.80.81.185`，這時候應該能正常使用你的網站。


## GKE 其他常用指令
  
### 重啟 pods

如果今天想要重新啟動 pod，可以用以下的指令

```shell
$ kubectl rollout restart deployment fastapi-deployment
```
這樣你的 `GKE` 就會重新部署，並且重新建立 `PODS`


:::tip
如果今天你修改了檔案內容，想要重新部署的話，記得要重新建立 image 並推上去
:::


### 確認 Kubernetes 集群狀態

如果沒有像下面一樣返回資訊，代表沒有正確啟動或是配置 `Kubernetes`

```shell
$ kubectl cluster-info

------
Kubernetes control plane is running at https://107.167.190.197
GLBCDefaultBackend is running at https://107.167.190.197/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
KubeDNS is running at https://107.167.190.197/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://107.167.190.197/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```


### 刪除 Kubernetes

```shell
$ gcloud container clusters delete fastapi-cluster --zone asia-east1-a
```
