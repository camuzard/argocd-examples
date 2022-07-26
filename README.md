# Argo CD examples

## Requirements

One or two kubernetes cluster(s)

## Using K3d (optional)

```
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
k3d cluster create k3d1 --agents 2 --api-port 6443 -p 9001:80@loadbalancer
k3d cluster create k3d2 --agents 2 --api-port 6444 -p 9002:80@loadbalancer
```

To allows traffic between the two you need to find their docker network interface id and
```
sudo iptables -I DOCKER-ISOLATION-STAGE-2 -o br-6754dd0c21b7 -i br-a10b20ae1cc8 -j ACCEPT
sudo iptables -I DOCKER-ISOLATION-STAGE-2 -o br-a10b20ae1cc8 -i br-6754dd0c21b7 -j ACCEPT
```

## Installation

```
# installing for k3d
kubectl apply -n argocd -f install/k3d.yaml
# otherwise
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl port-forward -n argocd service/argocd-server 9015:443
https://localhost:9015/argocd
```

## Deployments

```
kubectl apply projects/request-counter.yaml
kubectl apply config/op1/django.yaml
kubectl apply deploy/request-counter.yaml
```

## Register additional cluster

The default way to register a remote cluster is to use:
```
argocd cluster add <CONTEXTNAME>
```

For k3d, we will do it manually. 
First, create a service account in the target cluster for Argo CD.
```
kubectl apply --context k3d-k3d2 -f k3d-k3d2/sa.yaml
```

Replace in k3d-k3d2/register.yaml the remote <cluster_api> and <bearer_token>.
To retrieve the bearer token for the service account created, you can
```
sa=$(kubectl get --context k3d-k3d2 sa -n kube-system argocd-manager -o jsonpath='{.secrets[0].name}')
token=$(kubectl get --context k3d-k3d2 -n kube-system secret/$sa -o jsonpath='{.data.token}' | base64 --decode)
sed -i "s/<bearer_token>/$token/g" k3d-k3d2/register.yaml
```
```
kubectl apply -n argocd -f k3d-k3d2/register.yaml
```
