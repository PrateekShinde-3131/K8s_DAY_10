# Kubernetes Pod-to-Pod Communication and Service Exposure Example

This README outlines the steps to create Kubernetes namespaces, deployments, and services. We will demonstrate pod-to-pod communication, service exposure, and DNS resolution using Fully Qualified Domain Names (FQDN).

## Steps Overview
1. Create two namespaces: `ns1` and `ns2`.
2. Create deployments with the `nginx` image and a single replica in each namespace.
3. Get the IP addresses of the pods.
4. Test pod-to-pod communication by curling the IP address of a pod in a different namespace.
5. Scale the deployments to 3 replicas.
6. Create services to expose the deployments.
7. Test service-to-service communication using pod exec and curl.
8. Resolve communication errors using the FQDN of the services.
9. Clean up by deleting the namespaces.

## Steps To Do 
1. Create two namespaces ns1 and ns2
```
kubectl create namespace ns1
kubectl create namespace ns2
```
2. Create a deployment in each namespace with a single replica and nginx image
For ns1:
```
kubectl create deployment deploy-ns1 --image=nginx --replicas=1 -n ns1
```
For ns2:
```
kubectl create deployment deploy-ns2 --image=nginx --replicas=1 -n ns2
```
3. Get the IP address of each pod
You can list the pods in each namespace and retrieve their IP addresses:

```
kubectl get pods -o wide -n ns1
kubectl get pods -o wide -n ns2
```
4. Exec into the pod of deploy-ns1 and curl the IP of the pod from deploy-ns2
First, exec into the deploy-ns1 pod:

```
kubectl exec -it <pod-name-of-deploy-ns1> -n ns1 -- /bin/sh
```
Then, within the pod:
```
curl <pod-ip-of-deploy-ns2>
```
You should get a successful response if pod-to-pod communication is working.

5. Scale both deployments from 1 to 3 replicas
For deploy-ns1:
```
kubectl scale deployment deploy-ns1 --replicas=3 -n ns1
```
For deploy-ns2:
```
kubectl scale deployment deploy-ns2 --replicas=3 -n ns2
```
6. Create services to expose the deployments
For deploy-ns1:
```
kubectl expose deployment deploy-ns1 --type=ClusterIP --name=svc-ns1 -n ns1
```
For deploy-ns2:
```
kubectl expose deployment deploy-ns2 --type=ClusterIP --name=svc-ns2 -n ns2
```
7. Exec into each pod and curl the service IP of the other namespace
For deploy-ns1 pods:
```
kubectl exec -it <pod-name-of-deploy-ns1> -n ns1 -- /bin/sh
curl <svc-ns2-clusterIP>
```
For deploy-ns2 pods:
```
kubectl exec -it <pod-name-of-deploy-ns2> -n ns2 -- /bin/sh
curl <svc-ns1-clusterIP>
```
This should work, and you should get responses from each service.

8. Curl the service name instead of the IP (This might fail)
From within the deploy-ns1 pod:
```
curl svc-ns2
```
This likely won't work due to DNS resolution, and you'll get a host not found error.

9. Curl using the FQDN of the service
The FQDN format is <service-name>.<namespace>.svc.cluster.local.
For example, from deploy-ns1:
```
curl svc-ns2.ns2.svc.cluster.local
```
Similarly, from deploy-ns2:
```
curl svc-ns1.ns1.svc.cluster.local
```
This should successfully resolve the DNS and return a response.

10. Delete both namespaces
Finally, delete the namespaces (which will remove the services and deployments):
```
kubectl delete namespace ns1
kubectl delete namespace ns2
```
This will clean up all the resources in ns1 and ns2, including the deployments and services.
