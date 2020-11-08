# Bootstrapping Control Plane

We can bootstrap Kubernetes in any of the control plane nodes.

Let's pick one control plane node, say `k8s-master0`, to bootstrap.

## Pick one control plane to bootstrap

> Note: In `k8s-master0`

```sh
# Define a kubeadm config file
mkdir -p /etc/kubernetes/kubeadm
cat > /etc/kubernetes/kubeadm/kubeadm-config.yaml <<EOF
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: vehl48.8msy2evs9g4ycfej
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 172.21.0.3
  bindPort: 6443
nodeRegistration:
  criSocket: /run/containerd/containerd.sock
  name: master0
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - localhost
  - 127.0.0.1
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: k8s-lb0:6443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.19.3
networking:
  dnsDomain: cluster.local
  podSubnet: 10.32.0.0/12
  serviceSubnet: 10.96.0.0/12
scheduler: {}
EOF

# Bootstrap kubernetes
kubeadm init \
    --config=/etc/kubernetes/kubeadm/kubeadm-config.yaml \
    --ignore-preflight-errors=all \
    --upload-certs \
    --v=6
```

> SAMPLE OUTPUT:

```
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-lb0:6443 --token r4ctec.u9nts6d7uf1nkbir \
    --discovery-token-ca-cert-hash sha256:1d070523f5f342240913c761f1519c78e6f69b8afcf70c4f10cecb5980464e34 \
    --control-plane --certificate-key 945a8f58211b8bc02f0f1feb6c27b163074611c32681064bc92d3245fe11234b

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-lb0:6443 --token r4ctec.u9nts6d7uf1nkbir \
    --discovery-token-ca-cert-hash sha256:1d070523f5f342240913c761f1519c78e6f69b8afcf70c4f10cecb5980464e34
```

Next: [05-join-nodes](05-join-nodes.md)
