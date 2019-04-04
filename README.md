# Kubernetes Cheat Sheet

 A cheat sheet for Kubernetes commands.

## Kubectl Aliases and functions

Linux
```
alias k=kubectl

# print or change current namespace:
function kns() { 
  [ $# -eq 0 ] \
    && kubectl config get-contexts | grep '^\*' | awk '{print $5}' \
    || kubectl config set-context $(kubectl config current-context) --namespace=$1;
}
```

Windows
```
Set-Alias -Name k -Value kubectl
```

## Cluser Info

- Get clusters
```
kubectl config get-clusters
NAME
docker-for-desktop-cluster
foo
```

- Get cluster info.
```
kubectl cluster-info
Kubernetes master is running at https://172.17.0.58:8443
```

## Contexts

A context is a cluster, namespace and user.

- Get a list of contexts.
```
kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          docker-desktop       docker-desktop               docker-desktop
*         foo                  foo                          foo                  bar
```

- Get the current context.
```
kubectl config current-context
foo
```

- Switch current context.
```
kubectl config use-conext docker-desktop
```

## Get Commands

```
kubectl get all
kubectl get namespaces
kubectl get configmaps
kubectl get nodes
kubectl get pods
kubectl get rs
kubectl get svc kuard
kubectl get endpoints kuard
```

Additional switches that can be added to the above commands:

- `-o wide` - Show more information.
- `-w` - watch for changes.

## Namespaces

- `--namespace` - Get a resource for a specific namespace.

You can set the default namespace for the current context like so:

```
kubectl config set-context $(kubectl config current-context) --namespace=my-namespace
```

## Labels

- Get pods showing labels.
```
kubectl get pods --show-labels
```

- Get pods by label.
```
kubectl get pods -l environment=production,tier!=frontend
kubectl get pods -l 'environment in (production,test),tier notin (frontend,backend)'
```

## Describe Command

```
kubectl describe nodes [id]
kubectl describe pods [id]
kubectl describe rs [id]
kubectl describe svc kuard [id]
kubectl describe endpoints kuard [id]
```

## Delete Command

```
kubectl delete nodes [id]
kubectl delete pods [id]
kubectl delete rs [id]
kubectl delete svc kuard [id]
kubectl delete endpoints kuard [id]
```

## Create vs Apply

`kubectl create` can be used to create new resources while `kubectl apply` inserts or updates resources while maintaining any manual changes made like scaling pods.

- `--record` - Add the current command as an annotation to the resource.

## Create Pod

```
kubectl run kuard --generator=run-pod/v1 --image=gcr.io/kuar-demo/kuard-amd64:1 -o yaml --dry-run > kuard-pod.yml
kubectl apply -f kuard-pod.yml
```

## Create Deployment

```
kubectl run kuard --image=gcr.io/kuar-demo/kuard-amd64:1 -o yaml --dry-run > kuard-deployment.yml
kubectl apply -f kuard-deployment.yml
```

## Create Service

```
kubectl expose deployment kuard --port 8080 --target-port=8080 -o yaml --dry-run > kuard-service.yml
kubectl apply -f kuard-service.yml
```

## Export YAML for New Pod

```
kubectl run my-cool-app —-image=me/my-cool-app:v1 -o yaml --dry-run > my-cool-app.yaml
```

## Export YAML for Existing Object

```
kubectl get deployment my-cool-app -o yaml --export > my-cool-app.yaml
```

## Logs

- Get logs.
```
kubectl logs -l app=kuard
```

- Get logs for previously terminated container.
```
kubectl logs POD_NAME --previous
```

- Watch logs in real time.
```
kubectl attach POD_NAME
```

- Copy files out of pod (Requires `tar` binary in container).
```
kubectl cp POD_NAME:/var/log .
```

You can also install and use [kail](https://github.com/boz/kail).

## Port Forward

```
kubectl port-forward deployment/kuard 8080:8080
```

## Scaling

- Update replicas.
```
kubectl scale deployment nginx-deployment --replicas=10
```

## Autoscaling

- Set autoscaling config.
```
kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80
```

## Rollout

- Get rollout status.
```
kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

- Get rollout history.
```
kubectl rollout history deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment --revision=2
```

- Undo a rollout.
```
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

- Pause/resume a rollout
```
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deploy/nginx-deployment
```

## Pod Example

```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
 ```

## Deployment Example

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: my-namespace
  labels:
    - environment: production,
    - teir: frontend
  annotations:
    - key1: value1,
    - key2: value2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

## Dashboard

- Install

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/grafana.yaml
```

- Enable proxy

```
kubectl proxy
```

# Azure Kubernetes Service

[List of az aks commands](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest)

## Get Credentials
```
az aks get-credentials --resource-group <Resource Group Name> --name <AKS Name>
```

## Show Dashboard

Secure the dashboard like [this](http://blog.cowger.us/2018/07/03/a-read-only-kubernetes-dashboard.html). Then run:
```
az aks browse --resource-group <Resource Group Name> --name <AKS Name>
```

## Upgrade

Get updates
```
az aks get-upgrades --resource-group <Resource Group Name> --name <AKS Name>
```
# Docker Cheat Sheet

List all tags of the debian image (thanks to [StackOverflow](https://stackoverflow.com/questions/28320134/how-to-list-all-tags-for-a-docker-image-on-a-remote-registry) pointing to [fordone.com](https://fordodone.com/2015/10/02/docker-get-list-of-tags-in-repository/))
```
IMAGE=debian
curl -s https://registry.hub.docker.com/v1/repositories/${IMAGE}/tags  | sed -e 's/[][]//g' -e 's/"//g' -e 's/ //g' | tr '}' '\n'  | awk -F: '{print $3}'
```
