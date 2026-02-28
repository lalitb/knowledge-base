# kubectl Cheatsheet

> Quick-reference for managing Kubernetes clusters from the command line.

## 🔥 Most Used

| Command | What it does | Example |
|---|---|---|
| `kubectl get pods` | List pods in current namespace | `kubectl get pods -o wide` |
| `kubectl get all` | List all resources | `kubectl get all -n my-ns` |
| `kubectl apply -f <file>` | Apply manifest | `kubectl apply -f deploy.yaml` |
| `kubectl delete -f <file>` | Delete resources from manifest | `kubectl delete -f deploy.yaml` |
| `kubectl logs <pod>` | View pod logs | `kubectl logs -f --tail=100 mypod` |
| `kubectl exec -it <pod> -- sh` | Shell into pod | `kubectl exec -it mypod -- /bin/bash` |
| `kubectl describe pod <pod>` | Detailed pod info + events | `kubectl describe pod mypod` |
| `kubectl port-forward <pod> H:C` | Forward local port to pod | `kubectl port-forward svc/api 8080:80` |
| `kubectl config use-context <ctx>` | Switch cluster context | `kubectl config use-context prod` |
| `kubectl get events --sort-by='.lastTimestamp'` | Recent cluster events | `kubectl get events -n default` |

## Cluster Info

| Command | What it does | Example |
|---|---|---|
| `kubectl cluster-info` | Cluster endpoint info | `kubectl cluster-info` |
| `kubectl get nodes` | List nodes | `kubectl get nodes -o wide` |
| `kubectl top nodes` | Node resource usage | `kubectl top nodes` |
| `kubectl config get-contexts` | List all contexts | `kubectl config get-contexts` |
| `kubectl config current-context` | Show active context | `kubectl config current-context` |
| `kubectl config use-context <ctx>` | Switch context | `kubectl config use-context staging` |
| `kubectl api-resources` | List all resource types | `kubectl api-resources --namespaced=true` |

## Pods

| Command | What it does | Example |
|---|---|---|
| `kubectl get pods` | List pods | `kubectl get pods -l app=api` |
| `kubectl get pod <pod> -o yaml` | Pod full spec | `kubectl get pod mypod -o yaml` |
| `kubectl describe pod <pod>` | Pod details + events | `kubectl describe pod mypod` |
| `kubectl delete pod <pod>` | Delete pod | `kubectl delete pod mypod --grace-period=0` |
| `kubectl run <name> --image=<img>` | Run one-off pod | `kubectl run debug --image=busybox -it --rm -- sh` |
| `kubectl top pods` | Pod resource usage | `kubectl top pods --sort-by=memory` |
| `kubectl get pods --field-selector status.phase=Failed` | Find failed pods | |

## Deployments

| Command | What it does | Example |
|---|---|---|
| `kubectl get deploy` | List deployments | `kubectl get deploy -n prod` |
| `kubectl apply -f <file>` | Create/update deployment | `kubectl apply -f deploy.yaml` |
| `kubectl scale deploy <d> --replicas=N` | Scale replicas | `kubectl scale deploy api --replicas=3` |
| `kubectl rollout status deploy <d>` | Watch rollout progress | `kubectl rollout status deploy api` |
| `kubectl rollout history deploy <d>` | Rollout history | `kubectl rollout history deploy api` |
| `kubectl rollout undo deploy <d>` | Rollback to previous | `kubectl rollout undo deploy api` |
| `kubectl rollout undo deploy <d> --to-revision=N` | Rollback to specific rev | `kubectl rollout undo deploy api --to-revision=2` |
| `kubectl set image deploy/<d> ctr=img:tag` | Update image | `kubectl set image deploy/api api=app:v2` |
| `kubectl rollout restart deploy <d>` | Restart all pods | `kubectl rollout restart deploy api` |

## Services

| Command | What it does | Example |
|---|---|---|
| `kubectl get svc` | List services | `kubectl get svc -o wide` |
| `kubectl expose deploy <d> --port=P` | Create service for deployment | `kubectl expose deploy api --port=80 --target-port=8080` |
| `kubectl describe svc <svc>` | Service details + endpoints | `kubectl describe svc api` |
| `kubectl get endpoints <svc>` | IPs backing a service | `kubectl get endpoints api` |
| `kubectl delete svc <svc>` | Delete service | `kubectl delete svc api` |

## Logs & Debug

| Command | What it does | Example |
|---|---|---|
| `kubectl logs <pod>` | Pod logs | `kubectl logs mypod` |
| `kubectl logs -f <pod>` | Follow logs | `kubectl logs -f mypod` |
| `kubectl logs <pod> -c <ctr>` | Specific container logs | `kubectl logs mypod -c sidecar` |
| `kubectl logs <pod> --previous` | Logs from crashed container | `kubectl logs mypod --previous` |
| `kubectl logs -l app=api` | Logs by label selector | `kubectl logs -l app=api --all-containers` |
| `kubectl exec -it <pod> -- sh` | Shell into pod | `kubectl exec -it mypod -- /bin/bash` |
| `kubectl cp <pod>:/path ./local` | Copy from pod | `kubectl cp mypod:/app/log.txt ./log.txt` |
| `kubectl cp ./local <pod>:/path` | Copy to pod | `kubectl cp ./config.toml mypod:/app/` |
| `kubectl port-forward <pod> H:C` | Port-forward to pod | `kubectl port-forward mypod 8080:80` |
| `kubectl port-forward svc/<s> H:C` | Port-forward to service | `kubectl port-forward svc/api 8080:80` |
| `kubectl debug <pod> -it --image=busybox` | Ephemeral debug container | `kubectl debug mypod -it --image=nicolaka/netshoot` |

### 🔧 Debug a Crashing Pod Workflow

```bash
# 1. Check pod status and restarts
kubectl get pods | grep mypod

# 2. Look at events for scheduling/pull/crash reasons
kubectl describe pod mypod | tail -20

# 3. Check logs (current attempt)
kubectl logs mypod

# 4. Check logs from previous crash
kubectl logs mypod --previous

# 5. If pod is in CrashLoopBackOff, override entrypoint to inspect
kubectl run debug --image=myapp:v1 --restart=Never -it --rm \
  --command -- /bin/sh

# 6. Check resource limits / OOMKilled
kubectl get pod mypod -o jsonpath='{.status.containerStatuses[0].lastState}'

# 7. Attach ephemeral debug container (K8s 1.23+)
kubectl debug mypod -it --image=busybox --target=mycontainer
```

## ConfigMaps & Secrets

| Command | What it does | Example |
|---|---|---|
| `kubectl create cm <name> --from-file=<f>` | ConfigMap from file | `kubectl create cm app-cfg --from-file=config.toml` |
| `kubectl create cm <name> --from-literal=K=V` | ConfigMap from literal | `kubectl create cm app-cfg --from-literal=ENV=prod` |
| `kubectl get cm` | List ConfigMaps | `kubectl get cm -n my-ns` |
| `kubectl describe cm <name>` | Show ConfigMap data | `kubectl describe cm app-cfg` |
| `kubectl create secret generic <n> --from-literal=K=V` | Create secret | `kubectl create secret generic db-cred --from-literal=pass=s3cret` |
| `kubectl get secrets` | List secrets | `kubectl get secrets` |
| `kubectl get secret <n> -o jsonpath='{.data.key}'` | Get encoded value | `kubectl get secret db-cred -o jsonpath='{.data.pass}' \| base64 -d` |

## Namespaces

| Command | What it does | Example |
|---|---|---|
| `kubectl get ns` | List namespaces | `kubectl get ns` |
| `kubectl create ns <name>` | Create namespace | `kubectl create ns staging` |
| `kubectl config set-context --current --namespace=<ns>` | Set default namespace | `kubectl config set-context --current --namespace=prod` |
| `kubectl get all -n <ns>` | All resources in namespace | `kubectl get all -n kube-system` |
| `kubectl delete ns <name>` | Delete namespace (+ everything in it) | `kubectl delete ns staging` |

## RBAC

| Command | What it does | Example |
|---|---|---|
| `kubectl get roles,rolebindings` | List roles in namespace | `kubectl get roles -n prod` |
| `kubectl get clusterroles` | List cluster-wide roles | `kubectl get clusterroles` |
| `kubectl auth can-i <verb> <resource>` | Check your permissions | `kubectl auth can-i create pods` |
| `kubectl auth can-i <verb> <resource> --as=<user>` | Check another user's perms | `kubectl auth can-i get secrets --as=dev-user` |
| `kubectl create role <r> --verb=get --resource=pods` | Create role | `kubectl create role pod-reader --verb=get,list --resource=pods` |
| `kubectl create rolebinding <rb> --role=<r> --user=<u>` | Bind role to user | `kubectl create rolebinding dev-pods --role=pod-reader --user=dev` |

## Operators & CRDs

| Command | What it does | Example |
|---|---|---|
| `kubectl get crd` | List installed CRDs | `kubectl get crd \| grep greeting` |
| `kubectl explain <crd>.spec` | Show CR schema fields | `kubectl explain greetingservices.spec` |
| `kubectl get <cr-plural>` | List custom resources | `kubectl get greetingservices -n greeting` |
| `kubectl get <shortname>` | List custom resources via shortname | `kubectl get gs -n greeting` |
| `kubectl describe <shortname> <name>` | Detailed custom resource info | `kubectl describe gs my-greeter -n greeting` |
| `kubectl get <shortname> <name> -o yaml` | Inspect spec + status | `kubectl get gs my-greeter -n greeting -o yaml` |
| `kubectl logs -l app=<operator-label>` | View operator logs | `kubectl logs -n greeting -l app=greeting-operator --tail=100` |

## CI/CD + RBAC Debug (Fast Path)

```bash
# CI deploy says "forbidden"?
kubectl auth can-i apply deployments -n greeting
kubectl auth can-i create secrets -n greeting
kubectl auth can-i patch services -n greeting

# Confirm what service account CI job is using (in-cluster jobs)
kubectl auth can-i --list -n greeting

# Dry-run apply before merge
kubectl apply --dry-run=server -f k8s/

# Rollout status after deployment
kubectl rollout status deployment/greeting-service -n greeting
kubectl get events -n greeting --sort-by='.lastTimestamp' | tail -20
```

## Quick Troubleshooting

| Symptom | Commands to Run |
|---|---|
| Pod stuck `Pending` | `kubectl describe pod <p>` → check Events for scheduling failures |
| `ImagePullBackOff` | `kubectl describe pod <p>` → check image name, pull secrets |
| `CrashLoopBackOff` | `kubectl logs <p> --previous` → check app error / OOMKilled |
| Service not reachable | `kubectl get endpoints <svc>` → empty = selector mismatch |
| Node `NotReady` | `kubectl describe node <n>` → check conditions, kubelet |
| DNS not resolving | `kubectl run -it --rm dns-test --image=busybox -- nslookup <svc>` |

---

*See [Kubernetes 101](../../tutorials/rust-k8s-operator/README.md#part-3-kubernetes-101--deploy-with-kubectl) for detailed walkthrough.*
