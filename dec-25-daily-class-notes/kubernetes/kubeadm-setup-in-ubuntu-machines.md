# Kubeadm setup in ubuntu machines

Make sure docker installed in master and nodes, make sure master has 2 cpu's

{% stepper %}
{% step %}
### Prepare both master and nodes

Execute the following commands on both the master and each node:

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

For more details: https://kubernetes.io/docs/setup/independent/install-kubeadm/
{% endstep %}

{% step %}
### Initialize the master

Run this on the master. After it completes you'll get the node join command—copy and save it. If you forget the join command you can regenerate it with `kubeadm token create --print-join-command`.

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```
{% endstep %}

{% step %}
### Configure kubectl on the master

Run these commands on the master to set up kubectl for your user:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
{% endstep %}

{% step %}
### Install a Pod network (Flannel)

On the master, apply the Flannel manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```
{% endstep %}

{% step %}
### Join each node to the cluster

On each node, run the join command you received from `kubeadm init`. Example format:

```bash
kubeadm join 10.128.0.8:6443 --token q915fe.do2ty6a8ow6qjixt \
--discovery-token-ca-cert-hash sha256:acd137106e6b763d1ca6b5a4f7c1b1538c2ee8af81e47f9ea3f385c66cd710b3
```
{% endstep %}

{% step %}
### Verify the cluster

On the master (or a machine with kubectl configured), run:

```bash
kubectl get nodes
kubectl get pods
kubectl get service
```
{% endstep %}
{% endstepper %}
