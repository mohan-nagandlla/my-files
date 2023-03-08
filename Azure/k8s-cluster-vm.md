```
sudo apt-get update
sudo apt install docker.io -y
sudo groupadd docker
sudo gpasswd -a $USER docker
source ~/.bash_profile
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind create cluster --name k8s

```
