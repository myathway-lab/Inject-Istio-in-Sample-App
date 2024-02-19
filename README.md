## Enable Istio Injection 
## 
1) Install Istio using demo profile.

```yaml
vagrant@kindcluster-box:~$ istioctl install --set profile=demo -y
```

2) Create istio-in-action namespace & deploy Sample App in istio-in-action ns.

```yaml
vagrant@kindcluster-box:~$ kubectl create ns istio-in-action
vagrant@kindcluster-box:~$ git clone git@github.com:hellocloudio/hellocloud-native-box.git
vagrant@kindcluster-box:~$ cd hellocloud-native-box/istio-cop/1-start-istio/sample-apps
vagrant@kindcluster-box:~/hellocloud-native-box/istio-cop/1-start-istio/sample-apps$ kubectl apply -f . -n istio-in-action
```

```yaml
vagrant@kindcluster-box:~$ kubectl get po -n istio-in-action
NAME                                  READY   STATUS    RESTARTS   AGE
purchase-history-v1-8c7fdcd9d-7j2wk   1/1     Running   0          147m
recommendation-6d5786d695-k6jzq       1/1     Running   0          147m
sleep-7d9ff98856-c78pn                1/1     Running   0          147m
web-api-68fdcf9f75-867tl              1/1     Running   0          147m
```

3) Inject Istio in istio-in-action namespace. 

```yaml
vagrant@kindcluster-box:~$ kubectl label namespace istio-in-action istio-injection=enabled
namespace/istio-in-action labeled

vagrant@kindcluster-box:~/hellocloud-native-box/istio-cop/1-start-istio/sample-apps$ kubectl get po -n istio-in-action
NAME                                  READY   STATUS    RESTARTS   AGE
purchase-history-v1-8c7fdcd9d-7j2wk   1/1     Running   0          147m
recommendation-6d5786d695-k6jzq       1/1     Running   0          147m
sleep-7d9ff98856-c78pn                1/1     Running   0          147m
web-api-68fdcf9f75-867tl              1/1     Running   0          147m
```

After the ns was labeled with istio injection, we still cannot see the sidecar container was not able to inject.

4) Rollout restart the App deployment.

```yaml
vagrant@kindcluster-box:~$ kubectl rollout restart deploy purchase-history-v1 -n istio-in-action
deployment.apps/purchase-history-v1 restarted
```

Now, we can see purchase-history pod was injected with envoy proxy. 

```yaml
vagrant@kindcluster-box:~$ kubectl get po -n istio-in-action
NAME                                   READY   STATUS    RESTARTS   AGE
purchase-history-v1-74b756fcf7-8pw2q   2/2     Running   0          18s
recommendation-6d5786d695-k6jzq        1/1     Running   0          155m
sleep-7d9ff98856-c78pn                 1/1     Running   0          155m
web-api-68fdcf9f75-867tl               1/1     Running   0          155m
```
## 

<aside>
💡 Now we will enable the Istio injection before we deploy the application.

</aside>

###
1) Create test1 namespace & Label it. 

```yaml
vagrant@kindcluster-box:~$ kubectl create ns test1
vagrant@kindcluster-box:~$ kubectl label ns test1 istio-injection=enabled

vagrant@kindcluster-box:~$ kubectl get ns --show-labels
NAME                 STATUS   AGE     LABELS
default              Active   8d      kubernetes.io/metadata.name=default
**istio-in-action      Active   172m    istio-injection=enabled,kubernetes.io/metadata.name=istio-in-action**
istio-system         Active   7d2h    kubernetes.io/metadata.name=istio-system
kube-node-lease      Active   8d      kubernetes.io/metadata.name=kube-node-lease
kube-public          Active   8d      kubernetes.io/metadata.name=kube-public
kube-system          Active   8d      kubernetes.io/metadata.name=kube-system
local-path-storage   Active   8d      kubernetes.io/metadata.name=local-path-storage
metallb-system       Active   8d      kubernetes.io/metadata.name=metallb-system,pod-security.kubernetes.io/audit=privileged,pod-security.kubernetes.io/enforce=privileged,pod-security.kubernetes.io/warn=privileged
**test1                Active   3h11m   istio-injection=enabled,kubernetes.io/metadata.name=test1**
```

2) Deploy Sample App in test1 namespace.

```yaml
vagrant@kindcluster-box:~/hellocloud-native-box/istio-cop/1-start-istio/sample-apps$ kubectl apply -f . -n test1
vagrant@kindcluster-box:~$ kubectl get po -n test1
NAME                                  READY   STATUS    RESTARTS   AGE
purchase-history-v1-8c7fdcd9d-9n4ts   2/2     Running   0          77s
recommendation-6d5786d695-9njg2       2/2     Running   0          64s
sleep-7d9ff98856-sqhpt                2/2     Running   0          63s
web-api-68fdcf9f75-jfp2r              2/2     Running   0          63s
```

Now we can see the sidecar proxy was automatically added inside each pod in the test1 namespace.
