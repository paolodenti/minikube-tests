# Minikube tests

You need

* kubectl
* helm
* docker
* minikube

```bash
# if macOS
brew install hyperkit
DRIVER=hyperkit
# if linux
DRIVER=kvm2


minikube config set memory 8g
minikube config set cpus 8
minikube delete
minikube start --driver=${DRIVER} --kubernetes-version=v1.26.6 --cni calico --extra-config=kubelet.housekeeping-interval=10s --static-ip=192.168.200.200
minikube addons enable metallb

MINIKUBE_IP=$(minikube ip)
expect << _EOF_
spawn minikube addons configure metallb
expect "Enter Load Balancer Start IP:" { send "${MINIKUBE_IP}\\r" }
expect "Enter Load Balancer End IP:" { send "${MINIKUBE_IP}\\r" }
expect eof
_EOF_

minikube addons enable ingress
minikube addons enable metrics-server
```

## Install the helm template

```bash
helm upgrade --install \
  --set lbip="$(minikube ip)" \
  -f helm/values.yaml example ./helm
```

browse:

* `http://web1.<your minikube ip>.nip.io`
* `http://web2.<your minikube ip>.nip.io` (showing auto generated secrets)

* web2 has hpa enabled, 3 replicas, min 2, max 5
* web1 has no hpa, single replica
* postgres with username/password from secrets, db name from values
