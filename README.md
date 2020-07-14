# Serverless-Experiment
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

### OpenFaaS
- deploying `openfaas` with `docker swarm`
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

 
