## ConfigMap Demo
Step1: create a configmap
	```kubectl create cm myapp-config --from-literal=running_env=prod```

Step2: create pod that uses configmap
	```kubectl create -f configmap-pod.yaml```

## secret demo
Step1: Create a secret 
	```kubectl create secret generic db-secret --from-literal=username=testuser --from-literal=password=testpasswd```

Step2: Decoding a secret
```	kubectl get secret db-secret -o yaml
	echo 'dGVzdHBhc3N3ZA==' | base64 --decode```

Step3: create the pod
	```kubectl create -f secret-pod.yaml```

Step4: Reading a secret in pod
	login to pod and check the path where you have stored the secrets

## pod-affinity demo
Step1: Create a deployment with 2 pods
  ```	
  kubectl run nginx --image=nginx --replicas=2
	root@kmaster:~# kubectl get po 
	NAME                     READY   STATUS    RESTARTS   AGE
	nginx-7bb7cd8db5-bqq26   1/1     Running   0          7s
	nginx-7bb7cd8db5-d74g9   1/1     Running   0          7s
  ```

Step2: assign a label to one of the pod
```
	root@kmaster:~# kubectl label po nginx-7bb7cd8db5-bqq26 team=qa
	pod/nginx-7bb7cd8db5-bqq26 labeled
	root@kmaster:~# kubectl label po nginx-7bb7cd8db5-d74g9 team=dev
	pod/nginx-7bb7cd8db5-d74g9 labeled
	root@kmaster:~# kubectl get po --show-labels
	NAME                     READY   STATUS    RESTARTS   AGE   LABELS
	nginx-7bb7cd8db5-bqq26   1/1     Running   0          69s   pod-template-hash=7bb7cd8db5,run=nginx,team=qa
	nginx-7bb7cd8db5-d74g9   1/1     Running   0          69s   pod-template-hash=7bb7cd8db5,run=nginx,team=dev
 ```
 
Step3: create a pod with pod-affinity & anti-affinity
```
	root@kmaster:~# kubectl create -f pod-affinity.yaml 
	pod/pod-affinity-pod created
```

Step4: verify pod assignment is proper
```
	root@kmaster:~/demos/demos/module5/podaffinity# kubectl get po --show-labels -o wide
	NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES   LABELS
	nginx-7bb7cd8db5-bqq26   1/1     Running   0          2m25s   10.244.1.91   knode1   <none>           <none>            pod-template-hash=7bb7cd8db5,run=nginx,team=qa
	nginx-7bb7cd8db5-d74g9   1/1     Running   0          2m25s   10.244.2.70   knode2   <none>           <none>            pod-template-hash=7bb7cd8db5,run=nginx,team=dev
	pod-affinity-pod         1/1     Running   0          19s     10.244.1.92   knode1   <none>           <none>            <none>
```	

Step5: Now delete the pod and also change/delete labels on nginx pods
```
	root@kmaster:# kubectl delete -f pod-affinity.yaml 
	pod "pod-affinity-pod" deleted

	root@kmaster:# kubectl label po nginx-7bb7cd8db5-bqq26 team-
	pod/nginx-7bb7cd8db5-bqq26 labeled
	root@kmaster:# kubectl label po nginx-7bb7cd8db5-d74g9 --overwrite team=qa
	pod/nginx-7bb7cd8db5-d74g9 labeled
	root@kmaster:# kubectl get po --show-labels
	NAME                     READY   STATUS    RESTARTS   AGE     LABELS
	nginx-7bb7cd8db5-bqq26   1/1     Running   0          6m25s   pod-template-hash=7bb7cd8db5,run=nginx
	nginx-7bb7cd8db5-d74g9   1/1     Running   0          6m25s   pod-template-hash=7bb7cd8db5,run=nginx,team=qa
```

Step6: Create pod again
```
  root@kmaster:# kubectl create -f pod-affinity.yaml 
	pod/pod-affinity-pod created
```

Step7: This time, you can see that pod is assigned on knode2
```
  root@kmaster:# kubectl get po -o wide
	NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
	nginx-7bb7cd8db5-bqq26   1/1     Running   0          8m23s   10.244.1.91   knode1   <none>           <none>
	nginx-7bb7cd8db5-d74g9   1/1     Running   0          8m23s   10.244.2.70   knode2   <none>           <none>
	pod-affinity-pod         1/1     Running   0          107s    10.244.2.71   knode2   <none>           <none>
```

### Taint and toleration demo
Step1: taint a node
```
	root@kmaster:# kubectl taint nodes knode2 scanner-app=true:NoSchedule
```
	verify by running kubectl describe no knode2

Step2: Now run a deployment with 2 replicas and verify that none of the pod is assigned to knode2
```
	root@kmaster:# kubectl run nginx --image=nginx --replicas=2
	kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
	deployment.apps/nginx created
	root@kmaster:# kubectl get po -o wide
	NAME                     READY   STATUS              RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
	nginx-7bb7cd8db5-rtrmm   0/1     ContainerCreating   0          4s    <none>   knode1   <none>           <none>
	nginx-7bb7cd8db5-w94hl   0/1     ContainerCreating   0          4s    <none>   knode1   <none>           <none>
```

Step3: Now run a pod with toleration to above taint
```
	root@kmaster:# kubectl create -f toleration-pod.yaml 
	pod/toleration-pod created
	root@kmaster:~# kubectl get po -o wide
	NAME                     READY   STATUS              RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
	nginx-7bb7cd8db5-rtrmm   1/1     Running             0          74s   10.244.1.94   knode1   <none>           <none>
	nginx-7bb7cd8db5-w94hl   1/1     Running             0          74s   10.244.1.93   knode1   <none>           <none>
	toleration-pod           0/1     ContainerCreating   0          3s    <none>        knode2   <none>           <none>
```	
	We can see that toleration-pod was assigned to knode2

Step4: Finally remove the taint
```
	root@kmaster:# kubectl taint nodes knode2 scanner-app:NoSchedule-
	node/knode2 untainted
```
	verify with kubectl describe

### Persistent volume demo
Step1: First create a PV
```
	root@kmaster:~/demos/demos/module5/persistentvolumes# kubectl create -f pv-volume.yaml 
	persistentvolume/pv-volume created
	root@kmaster:# kubectl get pv
	NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
	pv-volume   5Gi        RWO            Retain           Available           manual                  7s
```
Step2: Create a PV claim
```
	root@kmaster:# kubectl create -f pv-claim.yaml 
	persistentvolumeclaim/pv-claim created
	root@kmaster:# kubectl get pvc
	NAME       STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
	pv-claim   Bound    pv-volume   5Gi        RWO            manual         5s
	root@kmaster:# kubectl create -f pv-pod.yaml 
	pod/pv-pod created
```
Step3: Create a pod using pvc
```
	root@kmaster:# kubectl create -f pv-pod.yaml 
	pod/pv-pod created
	root@kmaster:# kubectl get po -o wide
	NAME     READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
	pv-pod   1/1     Running   0          2m    10.244.2.76   knode2   <none>           <none>
```
Step4: Since pod is running on knode2, so go to knode2 and then creat an index.html file at /mnt/data/index.html
```
	root@knode2:~# sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
```	
Step5: Curl to pod IP on knode2 and you can see that index.html data is read
```
	root@knode2:~# curl 10.244.2.76
	Hello from Kubernetes storage
```
Step6: Login to pod and make some changes in index.html
```
	root@kmaster:# kubectl exec -i -t pv-pod -- bash
	root@pv-pod:/# echo -e "bye" >> /usr/share/nginx/html/index.html
```
	Verify these changes are reflected on knode2 /mnt/data/index.html

Step7: Delete pv-pod, pv-claim and pv-vol
```
	root@kmaster:# kubectl delete -f pv-pod.yaml 
	pod "pv-pod" deleted
	root@kmaster:# kubectl delete -f pv-claim.yaml 
	persistentvolumeclaim "pv-claim" deleted
	root@kmaster:# kubectl delete -f pv-volume.yaml 
	persistentvolume "pv-volume" deleted
```

	
