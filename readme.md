

### Setup com 07 máquinas:

* 03 Masters
* 03 Workers
* 01 HaProxy


### Links Importantes: 
* Tipos de topologias de K8s multi-master: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/
* Instalação kubeadm, kubelet e kubectl: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
* Instalação Kubernetes multi-master: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
* HAproxy: https://www.haproxy.org/

### Comandos:

HaProxy:
hostname nomedaquina
echo "nomedaquina" > /etc/hostname
bash
apt-get update && apt-get install haproxy -y

Adicionar dentro do arquivo /etc/haproxy/haproxy.cfg:
frontend kubernetes
        mode tcp
        bind 172.31.76.236:6443 #IP da máquina do haproxy (privado)
        option tcplog
        default_backend k8s-masters

backend k8s-masters
        mode tcp
        balance roundrobin
        option tcp-check
		#IP's privados dos masters
        server k8s-master-1 172.31.72.35:6443 check fall 3 rise 2
        server k8s-master-2 172.31.77.230:6443 check fall 3 rise 2
        server k8s-master-3 172.31.78.9:6443 check fall 3 rise 2
		
systemctl restart haproxy

====================================================

k8s-master (1, 2, 3) e Workers (1,2,3)
hostname nomedaquina
echo "nomedaquina" > /etc/hostname
bash

* Instalacao do Docker
curl -f fsSl https://get.docker.com | bash 

* Instalacao do Kubernetes
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

* Incluindo DNS
echo "172.31.76.236 haproxy" >> /etc/hosts

====================================================

* No Master01
kubeadm init --control-plane-endpoint "haproxy:6443" --upload-certs
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

====================================================

* No Master02 e Master03 rodar o kubeadmjoin control-plane e depois adicionar o path do kubelet
kubeadm join haproxy:6443 --token 0dcswp.djkga4qe1autjudl \
        --discovery-token-ca-cert-hash sha256:993e8ea96c1e2ed5f3bff9eba99168476fa83db73d9c58b40a07d74a858865c8 \
        --control-plane --certificate-key b79b28bc8d4b24c14cafa23d6d733ec3e4ac2169a0df04db9710a810b4891f38
 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

====================================================

* Nos Workers:
kubeadm join haproxy:6443 --token ln8jo5.zwxkyrmbi3hgcfqv --discovery-token-ca-cert-hash sha256:993e8ea96c1e2ed5f3bff9eba99168476fa83db73d9c58b40a07d74a858865c8

====================================================

* Nos Masters:

kubectl get nodes
kubectl get all --all-namespaces

====================================================


