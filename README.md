# ARHA_install
This repository stores the necessary file of installing ARHA_Project developing in 2024.

## 安裝步驟
1. 在所有機器都安裝Docker
2. 安裝並設定K8s
3. 安裝Prometheus
4. 安裝NVIDIA-GPU-Operator
5. 安裝Controller
6. 安裝Monitor
7. 安裝AgentManager

## 安裝Docker
``` bash=
sudo apt update
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

## 安裝並設定K8s
### 在K8s Master node上執行以下操作
``` bash=
sudo apt install curl
sudo apt install openssh-server
chmod +x ./k8s_master_install.sh
sudo vim /etc/fstab
./k8s_master_install.sh
```
把腳本執行結果的輸出儲存起來替換到k8s_computing_install.sh中，然後記得更改檔案中K8s master node的使用者帳號和IP

``` bash=
kubectl edit cm -n kube-system kubelet-config
sudo kubeadm upgrade node phase kubelet-config
sudo vim /etc/kubernetes/manifests/kube-controller-manager.yaml
kubectl edit cm -n kube-flannel kube-flannel-cfg
kubectl get pods -n kube-flannel -o name | xargs kubectl delete -n kube-flannel
```
1. 將kubelet-config中的data.kubelet.healthzBindAddress更改為0.0.0.0開通K8s中所有節點的10248 port
2. 讓controller可以檢查所有節點的kubelet狀態
3. 新增kube-controller-manager的參數，在spec.containers.command中新增- --node-monitor-grace-period=15s
4. 將網路配置從vxlan更改至host-gw
5. 重新啟動所有kube-flannel的Pod

### 在Computing_Node上個別執行以下操作
``` bash=
sudo apt install curl
chmod +x ./k8s_computing_install.sh
sudo vim /etc/fstab
. /k8s_computing_install.sh
```

### 在PC1上執行以下操作
``` bash=
kubectl label nodes {K8s Master node 主機名稱} arha-node-type=controller-node
kubectl label nodes {兩台Computing Node主機名稱} arha-node-type=computing-node
```

## 安裝Prometheus
``` bash=
chmod +x helminstallprometheus.sh
./helminstallprometheus.sh
```

## 安裝NVIDIA-GPU-Operator
### 在Comupting_Node上個別執行以下操作
``` bash=
先確認有沒有裝好NVIDIA Driver，建議安裝550之前的版本
sudo apt update
sudo apt install nvidia-driver-550
chmod +x NVIDIA_container_rumtime_install.sh
./NVIDIA_container_rumtime_install.sh
```
### 在K8s Master Node上執行以下操作
``` bash=
chmod +x helminstallgpuoperator.sh
./helminstallgpuoperator.sh
```
## 安裝Controller
1. 先更改 information/serviceSpec.json的內容
2. 在PC1上執行以下指令
``` bash=
sudo mkdir -p /arha
sudo mkdir -p /arha/data
sudo mkdir -p /arha/logs
sudo cp ./logs/* /arha/logs/
sudo cp ./information/* /arha/data/
chmod +x controller_install.sh
./controller_install.sh
```

## 安裝Monitor
### 在PC1上執行以下指令
``` bash=
kubectl apply -f Monitor/monitor-deployment.yaml
kubectl apply -f Monitor/monitor-service.yaml
```
## 安裝AgentManager
### 先更改AgentManager/agentmanager-deployment.yaml的內容然後在PC1上執行以下指令
``` bash=
kubectl apply -f AgentManager/agentmanager-deployment.yaml
kubectl apply -f AgentManager/agentmanager-service.yaml
```