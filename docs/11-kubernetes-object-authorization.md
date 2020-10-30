# ກຳນົດຄ່າ Kubernetes Object Authorization
ພາຍໃນ Kubernetes Cluster ເອງກໍ່ມີການຄອບຄຸມການເຂົ້າຖິງໃນແຕ່ລະສ່ວນດ້ວຍແນວຄິດ RBAC (Role-Based Access Control) ດັ່ງນັ້ນໃນການໃຊ້ວຽກຈຳເປັນຕ້ອງກຳນົດໃຫ້ object ໂຕໃດ ສາມາດໃຊ້ວຽກ object  ໂຕໃດໄດ້ແນ່ ໃນເອກະສານນີ້ຈະກຳນົດໃຫ້ Kubernetes API Server ສາມາດເຂົ້າເຖິງຂໍ້ມູນບາງສ່ວນຂອງ Kubelet ໃນບົດບາດຂອງ system:kubelet-api-admin
##  ກຳນົດໃຫ້ຜູ່ໃຊ້ຊື່ kubernetes ມີສິດໃນບົດບາດ  system:kubelet-api-admin [master0]
```
{
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: apiserver-kubelet-api-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kubelet-api-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: kubernetes
EOF
}
```
**Next>** [ສ້າງ kubeconfig สำหรับ remote access](12-kubectl-remote-access.md)

**<Prev** [ຕິດຕັ້ງ Kubernetes Worker Nodes](10-install-worker-node.md)
