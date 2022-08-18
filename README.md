# Kubernetes playground

For testing stuff outside of production for a change.

# Create cluster

## Kind

```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: playground
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

_See docs: https://kind.sigs.k8s.io/docs/user/ingress/_

# Install components

## Nginx Ingress

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`

_For creating local cert see: https://github.com/FiloSottile/mkcert_

`mkcert "*.local.gd"`

`kubectl create secret tls local-tls --key ./nginx/local.gd+3-key.pem --cert ./nginx/local.gd+3.pem -n ingress-nginx`

```
kubectl patch deployment "ingress-nginx-controller" \
    -n "ingress-nginx" \
    --type "json" \
    --patch '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--default-ssl-certificate=ingress-nginx/local-tls"}]'
```

## ArgoCD

`helm repo add argo https://argoproj.github.io/argo-helm`

`helm install argocd argo/argo-cd -n argocd --values ./argocd/values.yaml --create-namespace`

`kubectl apply -f ./argocd/app-of-apps.yaml`
