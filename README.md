# myhelm

```
CLUSTER_ID=catsokkk7btpt1ef14gm yc managed-kubernetes cluster get --id $CLUSTER_ID --format json | jq -r .master.master_auth.cluster_ca_certificate | awk '{gsub(/\\n/,"\n")}1' > ca.pem
```

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ctsoft-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ctsoft-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: ctsoft-admin
  namespace: kube-system

```

```
kubectl create -f sa.yaml
SA_TOKEN=$(kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep ctsoft-admin | awk '{print $1}') -o json | jq -r .data.token | base64 --d)
MASTER_ENDPOINT=$(yc managed-kubernetes cluster get --id $CLUSTER_ID --format json | jq -r .master.endpoints.external_v4_endpoint)
kubectl config set-cluster ctsoftk8s --certificate-authority=ca.pem --server=$MASTER_ENDPOINT --kubeconfig=ctsoft.kubeconfig
kubectl config set-credentials ctsoft-admin --token=$SA_TOKEN --kubeconfig=ctsoft.kubeconfig
kubectl config set-context default --cluster=ctsoftk8s --user=ctsoft-admin --kubeconfig=ctsoft.kubeconfig
kubectl config use-context default --kubeconfig=ctsoft.kubeconfig
```
+++++++++++++++++++++
```
cd project/ctsoftk8s/
helm  -n prodenv list
helm  -n prodenv history ctsoft-prod-subchart
helm upgrade --install --atomic -n prodenv ctsoft-prod-subchart ctsoft-e/ 
helm rollback ctsoft-prod-subchart -n prodenv N
```
