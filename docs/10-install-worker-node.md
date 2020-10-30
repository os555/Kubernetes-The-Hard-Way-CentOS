# ຕີດຕັ້ງ Kubernetes Worker Nodes
kubelet และ kube-proxy ເປັນສ່ວນປະກອບ 2 ສ່ວນຂອງ Kubernetes ຕິດຕັ້ງທີ່ Worker Node ທີ່ໃຊ້ໃນການຈັດການສຶ່ງຕ່າງໆທີເກີດຂຶ້ນໃນ Cluster ແລະຍັງຕ້ອງຕິດຕໍ່ກັບ master node ຕະຫຼອດເວລາ ນອກເໜືອຈາກ ນັ້ນກໍ່ມີ Container Runtime ອີກ 
## ຕ້ຽມ Kubernetes Worker Node Binaries [all worker node]
```
mkdir -p \
 /etc/cni/net.d \
 /opt/cni/bin \
 /var/lib/kubelet \
 /var/lib/kube-proxy \
 /var/lib/kubernetes \
 /var/run/kubernetes

dnf install -y wget

wget --show-progress https://dl.k8s.io/v1.19.0/kubernetes-node-linux-amd64.tar.gz

tar xvfz kubernetes-node-linux-amd64.tar.gz
cd kubernetes/node/bin/
mv kubectl kube-proxy kubelet /usr/local/bin/
cd
```
## ຕ້ຽມຂໍ້ມູນທີ່ໃຊ້ໃນການເຮັດວຽກຂອງ kubelet [all worker node]
```
cp ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
cp ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
cp ca.crt /var/lib/kubernetes/

cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/etc/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}.key"
EOF
```
## ສ້າງ kubelet.service ສຳລັບ systemd [all worker node]
```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
## ຕ້ຽມຂໍ້ມູນທີ່ໃຊ້ໃນການເຮັດວຽກຂອງ kube-proxy [all worker node]
```
cp kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.0.0.0/16"
EOF
```
## ສ້າງ kube-proxy.service ສຳລັບ systemd [all worker node]
```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
## ເລື່ມການເຮັດວຽກຂອງ kubelet และ kube-proxy [all worker node]
```
systemctl daemon-reload
systemctl enable kubelet kube-proxy --now
```
## ທົດສອບ ແລະ ຜົນການທົດສອບ [master0]
```
kubectl get nodes --kubeconfig admin.kubeconfig
```
>ຜົນການທົດສອບ
```
NAME    STATUS     ROLES    AGE    VERSION
node0   NotReady   <none>   10m    v1.19.0
node1   NotReady   <none>   6m7s   v1.19.0
```
> ຜົນທີ່ໄດ້ ສະຖານະຂອງ node ຈະຍັງເປັນ Not Ready ເນື່ອງຈາກ ໃນສ່ວນ Network ຍັງບໍ່ໄດ້ຖືກກຳນົດຄ່າ

**Next>** [Kubernetes Object Authorization](11-kubernetes-object-authorization.md)

**<Prev** [ຕິດຕັ້ງ Loadbalancer สำหรับ master node](09-loadbalancer.md)
