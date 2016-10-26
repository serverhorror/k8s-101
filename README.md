## Kubernetes 101

We'll be doing a small showcase

### Starting a development environment with minikube

* [Upstream Documentation](http://kubernetes.io/docs/getting-started-guides/minikube/)

Let's prepare the prerequisites

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.12.0/minikube-darwin-amd64 \
  && chmod +x minikube \
  && sudo mv minikube /usr/local/bin/
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/darwin/amd64/kubectl \
  && chmod +x kubectl \
  && sudo mv kubectl /usr/local/bin/
```

Let's start our cluster, also let's discover our IP to use for services

```
minikube start

# take not of this
# We'll need this IP to talk to our (exposed) services
minikube ip
```

### **local** _development_

Let's create a way to monitor metrics from all around our cluster.
Especially interesting to us is the hardware side as well as generic
Kubernetes metrics.

* Service
  Exposes IP addresses either to cluster internal use only or make it publicly available for consumption.
* Deployment
  This will be our "_container_" resource that will take care of:
  * Pods
    The atomic unit of deployment in Kubernetes. Think of it as a collection of containers that **always** can talk to each via `localhost`.
  * Replica Sets
    Keeps a defined number of Pods running at all Times
  * DaemonSets
  * Volumes
    This is used to configure a service as well as to provide a scratch space for some services.
* ConfigMap
  Basically, a descriptive way to expose configuration files to services that require them.


```
kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/01-grafana/01-service.yaml
kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/01-grafana/02-deployment.yaml

kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/02-prometheus/01-service.yaml
kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/02-prometheus/02-NodePort-service.yaml
kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/02-prometheus/03-configmap.yaml
kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/02-prometheus/04-deployment.yaml

kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/01-grafana/01-service.yaml # multiple k8s resources
kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/01-grafana/02-deployment.yaml

kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/03-node_exporter/01-service.yaml
kubectl apply -f https://raw.githubusercontent.com/serverhorror/k8s-101/master/03-node_exporter/02-daemonset.yaml
```


### **Production**_-ish_ Deployment

* [Upstream Documentation](http://kubernetes.io/docs/getting-started-guides/kubeadm/)

#### Master Deployment

```
cat <<EOS | /bin/bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get -qq update
apt-get install -qq -y docker.io
apt-get install -qq -y kubelet kubeadm kubectl kubernetes-cni
echo done
EOS
```

```
kubeadm init
```

```
# Magic networking! (Overlay Network for nodes)
kubectl apply -f https://git.io/weave-kube
```
The above configures our overlay network. It let's the pods running talk to each other if necessary (think: front ed talks to back end)


```
# We want the nice dashboard UI
# This won't start until we have actual nodes
kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
```

#### Node Deployment

```
cat <<EOS | /bin/bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get -qq update
apt-get install -qq -y docker.io
apt-get install -qq -y kubelet kubeadm kubectl kubernetes-cni
echo done
EOS

kubeadm join --token ...
```

#### Let's get to work with our new cluster

Let's grab a configuration file so we can use the dashboard in a somewhat secure way.

```
scp root@45.55.35.158:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin proxy
```