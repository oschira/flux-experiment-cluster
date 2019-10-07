# flux-experiment-cluster

example of multi-tenant flux with helm-operator

## other repositories

- https://github.com/oschira/flux-experiment-apps
- https://github.com/oschira/flux-experiment-helm-repo
- https://github.com/oschira/flux-experiment-config-team1
- https://github.com/oschira/flux-experiment-config-team2

## prerequisites

- kubernetes cluster
- fluxctl

## configure tiller

/!\ if using minikube, tiller can't be installed on k8s v1.16.
A cluster with v1.14 can be created like that: `minikube start --kubernetes-version v1.14.7`

```bash
kubectl -n kube-system create sa tiller

kubectl create clusterrolebinding tiller-cluster-rule \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller

helm init --skip-refresh --upgrade --service-account tiller
```

## install cluster

```bash
kubectl apply -k ./install/
fluxctl identity --k8s-fwd-ns=flux-system
# copy key to github
fluxctl identity --k8s-fwd-ns=team1
# copy key to github
fluxctl identity --k8s-fwd-ns=team2
# copy key to github
```

## play time

### initial state

#### flux-system namespace

```bash
kubectl get all -n flux-system

NAME                                      READY   STATUS    RESTARTS   AGE
pod/flux-c8bc9d896-cflhc                  1/1     Running   0          2m46s
pod/flux-helm-operator-545df5ccdf-s9cgr   1/1     Running   1          2m47s
pod/flux-memcached-694555cb97-w6znp       1/1     Running   0          2m47s

NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
service/flux-memcached   ClusterIP   10.99.80.52   <none>        11211/TCP   2m47s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/flux                 1/1     1            1           2m47s
deployment.apps/flux-helm-operator   1/1     1            1           2m47s
deployment.apps/flux-memcached       1/1     1            1           2m47s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/flux-c8bc9d896                  1         1         1       2m46s
replicaset.apps/flux-helm-operator-545df5ccdf   1         1         1       2m47s
replicaset.apps/flux-memcached-694555cb97       1         1         1       2m47s
```

#### team1 namespace

```bash
kubectl get all -n team1

NAME                         READY   STATUS    RESTARTS   AGE
pod/app-a-559d548746-4527k   1/1     Running   0          40s
pod/flux-6c6856f9-zhhrh      1/1     Running   0          90s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/app-a   ClusterIP   10.101.76.234   <none>        5000/TCP   40s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-a   1/1     1            1           40s
deployment.apps/flux    1/1     1            1           90s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/app-a-559d548746   1         1         1       40s
replicaset.apps/flux-6c6856f9      1         1         1       90s
```

#### team2 namespace

```bash
kubectl get all -n team2

NAME                        READY   STATUS    RESTARTS   AGE
pod/app-b-8848847bd-tr29l   1/1     Running   0          18s
pod/flux-7d69c76499-gzpdt   1/1     Running   0          101s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/app-b   ClusterIP   10.111.52.161   <none>        5000/TCP   18s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-b   1/1     1            1           18s
deployment.apps/flux    1/1     1            1           101s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/app-b-8848847bd   1         1         1       18s
replicaset.apps/flux-7d69c76499   1         1         1       101s
```

### deploy a new app

On team2 config repository, activate app-c (by removing the .off extension), then commit & push. As a consequence, flux should detect the new release and deploy it.

#### new state of team2 namespace

#### team2 namespace

```bash
kubectl get all -n team2

NAME                         READY   STATUS    RESTARTS   AGE
pod/app-b-8848847bd-tr29l    1/1     Running   0          7m13s
pod/app-c-6b6df9bb8b-gdb2z   1/1     Running   0          4s
pod/flux-7d69c76499-gzpdt    1/1     Running   0          8m36s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/app-b   ClusterIP   10.111.52.161   <none>        5000/TCP   7m13s
service/app-c   ClusterIP   10.102.85.9     <none>        5000/TCP   4s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-b   1/1     1            1           7m13s
deployment.apps/app-c   1/1     1            1           4s
deployment.apps/flux    1/1     1            1           8m36s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/app-b-8848847bd    1         1         1       7m13s
replicaset.apps/app-c-6b6df9bb8b   1         1         1       4s
replicaset.apps/flux-7d69c76499    1         1         1       8m36s
```
