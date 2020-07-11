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

