---
sidebar_position: 2
---

## Use Terraform Deploy Project To GKE
* 使用 Terraform 把專案部署到 GKE 上

參考網站：https://medium.com/@minghunghsieh/day-27-terraform-gcp%E5%AF%A6%E6%88%B0-%E4%BD%BF%E7%94%A8-terraform-%E5%89%B5%E5%BB%BA-gcp-%E9%81%8B%E7%AE%97%E6%9C%8D%E5%8B%99-kubernetes-gke-a3be135547e2

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

## 第二步驟 - 新增 main.tf, outputs.tf, provider.tf, variables.tf

分別來介紹一下，這幾個檔案是用來做啥的～

* provider.tf
這個檔案主要用於配置 Terraform 的提供者（provider），以確保 Terraform 可以正確地與 Google Cloud Platform（GCP）進行交互

1. 定義了要使用的提供者（如 Google Cloud Provider）
2. 包含提供者的配置，如項目 ID、區域等
3. 有時也包含提供者的版本限制

* variables.tf
這個檔案用於定義 Terraform 使用的變數，以實現更高度的可配置性和重用性

1. 定義了在其他文件中使用的變量
2. 可以包含變量的默認值、描述和類型
3. 使配置更加靈活和可重用
4. 允許在不修改主要配置的情況下更改值

* main.tf

這個檔案是主要的 Terraform 代碼，用於創建 GKE 會用到的資料

1. 這是主要的配置文件
2. 定義了要創建的所有資源（例如 GKE 集群、節點池、Kubernetes 部署等）
3. 包含資源之間的關係和依賴
4. 通常是最長和最複雜的文件

* outputs.tf
Terraform 配置中的資訊輸出為易於讀取和使用的格式，以便後續的操作、監視或報告

1. 定義了 Terraform 運行後要輸出的值
2. 這些輸出可以是創建的資源的屬性，如 IP 地址、資源名稱等
3. 對於查看創建的資源的關鍵信息很有用
4. 也可用於將信息傳遞給其他 Terraform 配置或腳本



### 新增四個檔案

* 檔案架構

```md
| FASTAPI_Docker
  |  -- DBoperator         
        | -- **init**.py        
        | -- app.py        
        | -- database.py   
        | -- models.py     
        | -- base.py
  |  -- deployment                -> 新增此檔案
        | -- main.tf              -> 新增此檔案
        | -- provider.tf          -> 新增此檔案
        | -- variables.tf         -> 新增此檔案
        | -- outputs.tf           -> 新增此檔案                
  |  Dockerfile
  |  pyproject.toml  
  |  main.py
  |  docker-compose.yml
  |  requirements.txt
  |  app.yaml
  |  env
```

### provider.tf

* provider "google" {...}: 這段配置用來設定 Google Cloud Provider，它告訴 Terraform 如何與 Google Cloud Platform (GCP) 進行交互
* provider "kubernetes": 這段配置用來設定 Kubernetes Provider，它告訴 Terraform 如何與 Kubernetes 集群進行交互
 
```tf
provider "google" {
  project = "fastapi-429704"     # 這是在 GCP 的 projectID
  region  = var.region           # 這是在 projectID 的 region
}

provider "kubernetes" {
  config_path = "~/.kube/config"  # 或者是其他正確的 kubeconfig 文件路徑
}
```
### variables.tf

* 此資料夾就單純是存放變數的地方

```tf
variable "project_id" {
  description = "GCP project ID"
  type        = string
  default     = "fastapi-429704"
}

variable "region" {
  description = "GCP region"
  type        = string
  default     = "asia-east1"
}

variable "cluster_name" {
  description = "GKE cluster name"
  type        = string
  default     = "fastapi-cluster-version3"
}

variable "gke_node_count" {
  description = "Number of nodes in the GKE cluster"
  type        = number
  default     = 1
}

variable "gke_node_machine_type" {
  description = "Machine type for GKE nodes"
  type        = string
  default     = "e2-medium"
}

variable "cloud_sql_host" {
  description = "Cloud SQL Host"
  type        = string
  default     = "35.201.232.244"
}

variable "db_user" {
  description = "Cloud SQL User"
  type        = string
  default     = "root"
}

variable "db_password" {
  description = "Cloud SQL Password"
  type        = string
  default     = "00000"
}
```
### outputs.tf

* output "load_balancer_ip": 這是輸出變量的名稱。當你在終端執行 terraform apply 後，這個名稱會被用來顯示該輸出變量的值
* value: 這部分指向一個 Kubernetes 資源，該資源是由 Terraform 管理的 Kubernetes Service，名稱為 fastapi，並且找尋裡面的 ip 資料

```tf
output "load_balancer_ip" {
  description = "The IP address of the load balancer"
  value       = kubernetes_service.fastapi.status[0].load_balancer[0].ingress[0].ip
}
```


#### 印出 service ip
```shell
$ kubectl get service fastapi-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

------
35.229.161.76
```

### main.tf

* resource "google_container_cluster" "primary": 定義一個 GKE 的 cluster
* resource "google_container_node_pool" "primary_nodes": 定義了 GKE 叢集中的節點池
* resource "kubernetes_secret" "cloudsql_db_credentials": 定義了一個 Kubernetes Secret，用於存儲敏感信息，比如數據庫的用戶名和密碼。
* resource "kubernetes_deployment" "fastapi": 定義了一個 Kubernetes 部署 (Deployment)，這個部署管理 fastapi 應用的容器
* resource "kubernetes_service" "fastapi": 定義了一個 Kubernetes 服務 (Service)，用來公開 fastapi 應用，並允許外部訪問

```tf
resource "google_container_cluster" "primary" {
  name     = var.cluster_name                         # cluster 名稱
  location = var.region                               # cluster 所在區域

  initial_node_count = var.gke_node_count             # GKE 初始 node 數量
  remove_default_node_pool = true                     # 加上這段可以把 default 的 node 給刪掉，只留等等我們要新建的 node_pool

  node_config {
    machine_type = var.gke_node_machine_type          # node 的機器類型 (machine_type)
  }
}

resource "google_container_node_pool" "primary_nodes" {
  cluster    = google_container_cluster.primary.name      # 指定這個節點池屬於哪個叢集，這裡引用了前面定義的 
  node_count = var.gke_node_count                         # 節點池中包含的節點數量

  node_config {                                           # 指定節點的配置
    machine_type = var.gke_node_machine_type
    disk_type    = "pd-standard"                          # 使用標準 HDD 而非 SSD
    disk_size_gb = 10                                     # 減少每個節點的磁碟大小
  }
}

resource "kubernetes_secret" "cloudsql_db_credentials" {
  metadata {                                              # 設定 Secret 的名稱 cloudsql-db-credentials 和命名空間 default
    name      = "cloudsql-db-credentials"
    namespace = "default"
  }

  data = {                                                # 儲存的實際資料
    username = var.db_user        
    password = var.db_password
  }
}

resource "kubernetes_deployment" "fastapi" {
  metadata {                                              # 指定部署的名稱 fastapi-deployment 和命名空間 default
    name      = "fastapi-deployment"    
    namespace = "default"
  }

  spec {                                                  # 部署的規範，包括副本數 (replicas) 和選擇器 (selector)
    replicas = 3

    selector {
      match_labels = {
        app = "fastapi"
      }
    }

    template {                                            # 定義 Pod 的模板，包括容器配置 (container) 和環境變數 (env)
      metadata {
        labels = {
          app = "fastapi"
        }
      }

      spec {
        container {                                       # 容器設定
          name  = "fastapi"
          image = "gcr.io/fastapi-429704/fastapi:latest"

          env {
            name  = "USE_CLOUD_SQL"
            value = "true"
          }

          env {
            name  = "CLOUD_SQL_HOST"
            value = var.cloud_sql_host
          }

          env {
            name = "CLOUD_SQL_USER"
            value_from {
              secret_key_ref {
                name = kubernetes_secret.cloudsql_db_credentials.metadata[0].name
                key  = "username"
              }
            }
          }

          env {
            name = "CLOUD_SQL_PASSWORD"
            value_from {
              secret_key_ref {
                name = kubernetes_secret.cloudsql_db_credentials.metadata[0].name
                key  = "password"
              }
            }
          }

          env {
            name  = "CLOUD_SQL_DB"
            value = "test"
          }

          port {
            container_port = 8000
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "fastapi" {
  metadata {                                      # 指定服務的名稱 fastapi-service 和命名空間 default
    name      = "fastapi-service"    
    namespace = "default"
  }

  spec {                                          # 服務的規範，包括選擇器 (selector) 和服務類型 (type)
    selector = {
      app = "fastapi"
    }

    type = "LoadBalancer"

    port {
      port        = 80
      target_port = 8000
    }
  }
}
```

:::tip `selector作用`
1. 管理 Pod: selector 中的 match_labels 是一組鍵值對，用於匹配 Pod 的標籤。Kubernetes 會查找符合這些標籤的 Pod，並確保這些 Pod 是由該部署管理的。  
2. 在進行滾動更新時，selector 幫助 Kubernetes 決定哪些 Pod 需要替換或更新。它會逐個替換符合這些標籤的 Pod，從而實現零停機的應用更新。  
3. 保障一致性: selector 確保只有符合指定標籤的 Pod 才會被部署所控制，從而避免部署錯誤地管理不相關的 Pod，這有助於維持系統的一致性和穩定性。  
:::



## 第三步驟 - 進行 terraform 部署

### 初始化 terraform

要執行初始化指令，要值得 cd 進 deployment 資料夾，偵測到 terraform 相關的設定檔才可以執行

```shell
$ cd deployment/
$ terraform init
```

### 查看部署計畫

初始化後，來看一下我們檔案設定好的所有細節，這個會印出你的設定檔寫的設定，顯示 Terraform 在 terraform apply 時會執行的更改，但它並不實際執行任何更改。
```shell
$ terraform plan
```

因此可以使用這個指令， `-out` 來將此計畫保存，這樣可以確保你的的計畫和生成的東西會保證一樣
```shell
$ terraform plan -out=tfplan
```

Ps. 當你不使用 `-out` 選項時，terraform plan 只是展示了一個當時的預計結果。假如在你執行 terraform apply 之前，環境或配置有任何變動，最終執行的結果可能與你最初看到的計劃略有不同。


### 連結 Kubernetes 集群

如果今天在跑 `terraform plan` 的時候，有可能會碰到連線錯誤，那可以執行以下指令，確認一下連線是否正常：

```shell
$ kubectl config current-context

----
error: current-context is not set
```
如果今天輸入此指令，出現類似資訊，就代表目前沒有連結 GKE 成功，改輸入以下指令：

```shell
$ gcloud container clusters get-credentials fastapi-cluster-version3 --region asia-east1 --project fastapi-429704
```
這個命令將配置 kubectl 使用 GKE 集群的憑據，並更新 kubeconfig 文件。這樣就可以連結成功了，之後可以輸入 `kubectl get pods` 等等指令查看是否有資料跑出來



### 執行部署

* 確認 plan 好後，就可以實際執行 plan 
```shell
$ terraform apply tfplan

------
kubernetes_secret.cloudsql_db_credentials: Creating...
kubernetes_service.fastapi: Creating...
kubernetes_secret.cloudsql_db_credentials: Creation complete after 0s [id=default/cloudsql-db-credentials]
kubernetes_deployment.fastapi: Creating...
kubernetes_service.fastapi: Still creating... [10s elapsed]
kubernetes_deployment.fastapi: Still creating... [10s elapsed]
kubernetes_deployment.fastapi: Creation complete after 16s [id=default/fastapi-deployment]
kubernetes_service.fastapi: Still creating... [20s elapsed]
kubernetes_service.fastapi: Still creating... [30s elapsed]
kubernetes_service.fastapi: Still creating... [40s elapsed]
kubernetes_service.fastapi: Creation complete after 46s [id=default/fastapi-service]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

load_balancer_ip = "35.229.161.76"
```

這邊的 `load_balancer_ip` 就是實際的 `external IP`，因此可以在網址上輸入該 `IP`，但是輸入後應該會發現，目前該 `IP` 處於無法讀取的狀態

ps. 這邊 outputs 印出的資訊，就是當初在 `outputs.tf` 檔案寫的資訊


## 第四步驟 - 打開 cloudSQL 的 public IP

剛剛我們已經透過 terraform 建置好 GKE 的 cluster, node 和 pods 了，因此我們來查看一下 `pods` 的詳情，會發現目前的 pods status 都是 crash 狀態，因此來查看一下原因

### 查看 pods 狀態
```shell
$ kubectl get pods

------
NAME                                 READY   STATUS             RESTARTS        AGE
fastapi-deployment-ddc58d6f7-r9rt5   0/1     CrashLoopBackOff   8 (3m23s ago)   21m
fastapi-deployment-ddc58d6f7-rh9qc   0/1     CrashLoopBackOff   8 (3m38s ago)   21m
fastapi-deployment-ddc58d6f7-slm65   0/1     CrashLoopBackOff   8 (4m51s ago)   21m
```

查看 pods 的 log 狀態，用以下指令，可以看看在部署的時間，發生了哪些問題

```shell
$ kubectl logs fastapi-deployment-ddc58d6f7-r9rt5

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


### cloudSQL 加入 pod 的外部 IP

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

可以發現 pod 都正常運作了，因此直接在網址輸入你的 `LoadBalance` 的外部 IP - `35.229.161.76`，這時候應該能正常使用你的網站。