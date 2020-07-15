# Serverless-Experiment
### Environment
```bash 
$ lsb_release -a         
Distributor ID: Ubuntu                     
Description:    Ubuntu 16.04.1 LTS         
Release:        16.04                      
Codename:       xenial

$ docker --version 
Docker version 18.09.7, build 2d0083d

$ nvidia-docker run nvidia/cuda:10.0-base nvidia-smi    
Tue Jul 14 18:20:14 2020                                                       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 410.48                 Driver Version: 410.48                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K40m          Off  | 00000000:83:00.0 Off |                    0 |
| N/A   45C    P0    68W / 235W |      0MiB / 11441MiB |    100%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

### Nvidia-Docker
- install nvidia-docker
    - [reference link](https://github.com/NVIDIA/nvidia-docker)
    - [deprecated version](https://nvidia.github.io/nvidia-docker/)
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -

distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install nvidia-docker2
sudo systemctl restart docker

```

- run nvidia docker
```
sudo nvidia-docker run nvidia/cuda:10.0-base nvidia-smi
```

### OpenFaaS + Docker Swarm
- initalize docker swarm
```
sudo docker stack rm func
sudo docker swarm leave --force
sudo docker swarm init --advertise-addr enp1s0f0
git clone https://github.com/openfaas/faas.git
cd faas
sudo ./deploy_stack.sh
```
- openfaas returns the credential info
```
admin
ae06fb68b997636611e33a438d8546cf0e1895d74609ac69b6546e8dc5c690e2
```
- deploy a function in openfaas
```
mkdir -p functions
cd functions
sudo faas-cli build -f ./hello-python.yml
sudo docker images | grep hello-python
update the hello-python.yml with your maverickjin88/hello-python
sudo faas-cli push -f ./hello-python.yml

sudo faas-cli login --username=admin --password=ae06fb68b997636611e33a438d8546cf0e1895d74609ac69b6546e8dc5c690e2
sudo faas-cli deploy -f ./hello-python.yml

sudo curl http://127.0.0.1:8080/function/hello-python -d "it's Sungho here"
```

### OpenFaaS + Kubernetes

- setup go and k3d
```
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
export GOROOT=/usr/local/go
export PATH=$PATH:/usr/local/go/bin
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/master/install.sh | TAG=v1.3.4 bash
k3d --version
k3d version v1.3.4
```

- setup kubernetes cluster
```
sudo k3d create
export KUBECONFIG="$(sudo k3d get-kubeconfig --name='k3s-default')"
kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.5", GitCommit:"e6503f8d8f769ace2f338794c914a96fc335df0f", GitTreeState:"clean", BuildDate:"2020-06-26T03:47:41Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2-k3s.1", GitCommit:"b8b17ba55f20e590df507fce333dfee13ab438c6", GitTreeState:"clean", BuildDate:"2019-10-16T05:17Z", GoVersion:"go1.13.3", Compiler:"gc", Platform:"linux/amd64"}
kubectl cluster-info
kubectl get nodes
```

- deploy kubernetes cluster using helm
```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
kubectl -n kube-system create sa tiller \
  && kubectl create clusterrolebinding tiller \
  --clusterrole cluster-admin \
  --serviceaccount=kube-system:tiller

helm init --skip-refresh --upgrade --service-account tiller
kubectl get pods -n kube-system
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
helm repo add openfaas https://openfaas.github.io/faas-netes/
helm repo update \
 && helm upgrade openfaas --install openfaas/openfaas \
    --namespace openfaas  \
    --set functionNamespace=openfaas-fn \
    --set generateBasicAuth=true

PASSWORD=$(kubectl -n openfaas get secret basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode)
echo "OpenFaaS admin password: $PASSWORD"
9zkXGugXwhaG
export OPENFAAS_URL=http://127.0.0.1:31112
kubectl get pods -n openfaas
kubectl port-forward -n openfaas svc/gateway 31112:8080 &
faas-cli login --username=admin --password=$PASSWORD
faas-cli version
```

- deploy a function
```
faas-cli deploy -f ./hello-python.yml
curl http://127.0.0.1:31112/function/hello-python -d "it's Sungho here"
Handling connection for 31112
Hello! You said: it's Sungho here
it's Sungho here
```

- kill the environment
```
kubectl delete namespace openfaas openfaas-fn
helm delete --purge openfaas
sudo k3d delete
```






