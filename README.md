#### Delete the cluster
```sh
k3d cluster delete test-cluster
```

#### Create cluster
```sh
k3d cluster create test-cluster --k3s-server-arg --no-deploy --k3s-server-arg traefik
kubectl config use-context k3d-test-cluster
kubectl cluster-info
kubectl get nodes
```
#### Install k8s dashboard (Takes approx 5 minutes)
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

#### Run the proxy (In other terminal)
```sh
kubectl proxy
```




#### open dashboard, wait for a minute or two proxy not always start right away
Visit `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`

```sh
echo "$(kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}")"
```
#### install istio
```sh
kubectl create namespace istio-system
helm install istio-base    -n istio-system ~/Desktop/argo/istio-1.10.1/manifests/charts/base
helm install istiod        -n istio-system ~/Desktop/argo/istio-1.10.1/manifests/charts/istio-control/istio-discovery
helm install istio-ingress -n istio-system ~/Desktop/argo/istio-1.10.1/manifests/charts/gateways/istio-ingress
helm install istio-egress  -n istio-system ~/Desktop/argo/istio-1.10.1/manifests/charts/gateways/istio-egress
kubectl label namespace default istio-injection=enabled
```

#### install Argo CD v1.8.1
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.8.1/manifests/install.yaml
kubectl port-forward svc/argocd-server -n argocd 8443:443
```

#### change argo cd password
```sh
INGRESS_HOST=127.0.0.1
PASS=$(kubectl --namespace argocd get pods --selector app.kubernetes.io/name=argocd-server --output name | cut -d'/' -f 2)
argocd context test-argocd --delete
argocd login --name test-argocd --insecure --username admin --password $PASS 127.0.0.1:8443
argocd account update-password --current-password $PASS --new-password admin
```
#### install Argo events & Argo workflows
```sh
kubectl create namespace argo
kubectl apply -n argocd -f argo-systems.yaml
```

#### Get token
```sh
echo "Bearer $(kubectl -n argo get secret $(kubectl -n argo get sa/argo-server -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}")"
```


