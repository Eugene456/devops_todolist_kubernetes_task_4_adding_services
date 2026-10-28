# INSTRUCTION

## 0. Deploy the manifests

```
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/todoapp-pod.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml
kubectl apply -f .infrastructure/busybox.yml
```

Check that both pods are `Running`/`Ready` and that the service has two endpoints:

```
kubectl get pods -n todoapp
kubectl get svc -n todoapp
kubectl get endpoints -n todoapp
```

## 1. Test via ClusterIP service DNS from the busybox pod

Open a shell inside the busybox pod:

```
kubectl exec -it busybox -n todoapp -- sh
```

From inside the pod, call the ClusterIP service by its DNS name
(`<service-name>.<namespace>.svc.cluster.local`, short form `<service-name>` also works within the same namespace):

```
curl todoapp-service.todoapp.svc.cluster.local
```

or the short form:

```
curl todoapp-service
```

A successful response (HTML of the landing page or a redirect) confirms that the ClusterIP service is correctly routing traffic to the todoapp pods.

## 2. Test the ToDo application using `kubectl port-forward`

Forward a local port to the ClusterIP service:

```
kubectl port-forward svc/todoapp-service -n todoapp 8080:80
```

Then open the application in a browser or with curl on the host machine:

```
curl http://localhost:8080
```

or open `http://localhost:8080` in a browser.

## 3. Access the application using the NodePort service

Get the IP address of a cluster node:

```
kubectl get nodes -o wide
```

(For Minikube, you can also run `minikube ip`.)

Then access the application using the node IP and the configured `nodePort` (`30007`):

```
curl http://<NODE_IP>:30007
```

or open `http://<NODE_IP>:30007` in a browser.

### Note for Docker Desktop Kubernetes

On Docker Desktop, the node IP (e.g. `172.18.0.2`) lives inside Docker's internal/WSL2 network and is usually **not** reachable from the Windows host directly (`curl http://<NODE_IP>:30007` will time out from PowerShell/CMD), and `localhost:30007` is not automatically forwarded either in every setup.

To still verify that the NodePort service itself works correctly, test it from inside the cluster instead:

```
kubectl exec -it busybox -n todoapp -- curl http://<NODE_IP>:30007
```

A successful HTML response confirms the NodePort service and kube-proxy are configured correctly, even though the port isn't reachable from the host.

If you need to actually open the app from the Windows host/browser, forward the NodePort service's port to your machine instead:

```
kubectl port-forward svc/todoapp-nodeport-service -n todoapp 30007:80
```

Then open `http://localhost:30007` in a browser.
