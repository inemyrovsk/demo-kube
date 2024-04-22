1. create cluster
2. install nginx-ingress
3. deploy application
4. install dashboard

## Prerequisite 
docker
kind
kubectl

1. we are using `kind` to create kubernetes local cluster, (guide)[https://kind.sigs.k8s.io/docs/user/ingress/]
```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

`docker ps` to check nodes

`kubectl cluster-info --context kind-kind` to check cluster API

2. install nginx-ingress controller:
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`

check status:
```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

apply kubernetes manifests:
```
git clone https://github.com/inemyrovsk/demo-kube.git
kubectl apply -f demo-kube
```

check:
`kubectl get pods`


login into pod:
`kubectl exec -it CHANGE-pod-name -n default -- bash`

check networking inside kuberenets:
`kubectl exec -it CHANGE-pod-name -n default -- curl service-name.namespace.svc.cluster.local`


describe pod configs:
`kubectl describe pod ingress-nginx-controller-55bbd74b5f-sdj4g -n ingress-nginx -- bash`


install k9s dashboard:
`brew install derailed/k9s/k9s`

or you can also use k8sLens


remove cluster:
`kind delete cluster -n kind`
