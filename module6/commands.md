## kubectl cp commands

- Copy logs host to pod/container
```
root@kmaster:~# kubectl cp test kube-system/etcd-kmaster:/tmp/ 
```

- Copy logs from pod/container to host
```
root@kmaster:~# kubectl cp kube-system/etcd-kmaster:/tmp/test -c etcd /tmp/test
tar: removing leading '/' from member names
root@kmaster:~# ls /tmp/test 
/tmp/test
```

## etcdctl database operations

Step1: Get container name for etcd in kube-system namespace
```
root@kmaster:~# kubectl get po -n kube-system | grep etcd
etcd-kmaster                      1/1     Running   2          8d
```

Step2: Describe this pod to get the command and certificates
```
root@kmaster:~# kubectl  describe po etcd-kmaster -n kube-system 
Name:                 etcd-kmaster
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 kmaster/192.168.56.101
Start Time:           Thu, 10 Oct 2019 19:40:08 +0530
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: 4ecda28ac93d555217d49e8a8885ac11
                      kubernetes.io/config.mirror: 4ecda28ac93d555217d49e8a8885ac11
                      kubernetes.io/config.seen: 2019-10-05T12:38:26.046554455+05:30
                      kubernetes.io/config.source: file
Status:               Running
IP:                   192.168.56.101
Containers:
  etcd:
    Container ID:  docker://1a137789dade287f9c2742a63b577a19e7f3593aee84ba8508cd4d008cc64672
    Image:         k8s.gcr.io/etcd:3.3.10
    Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:17da501f5d2a675be46040422a27b7cc21b8a43895ac998b171db1c346f361f7
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.56.101:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --initial-advertise-peer-urls=https://192.168.56.101:2380
      --initial-cluster=kmaster=https://192.168.56.101:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.56.101:2379
      --listen-peer-urls=https://192.168.56.101:2380
      --name=kmaster
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Thu, 10 Oct 2019 19:40:13 +0530
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 10 Oct 2019 19:36:30 +0530
      Finished:     Thu, 10 Oct 2019 19:39:18 +0530
    Ready:          True
    Restart Count:  2
    Liveness:       exec [/bin/sh -ec ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key get foo] delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:    <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         BestEffort
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:            <none>
```

Step3: You can copy the portion of commands from liveness probes and then login to the pod
```
root@kmaster:~# kubectl exec -i -t etcd-kmaster -n kube-system -- sh
```

Step4: take backup and verify backup 
```
/ # ETCDCTL_API=3 etcdctl  snapshot save snapshotdb --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --c
ert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key 
Snapshot saved at snapshotdb
/ # ETCDCTL_API=3 etcdctl  snapshot status snapshotdb --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt -
-cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key 
27b3113f, 260108, 843, 3.0 MB
/ # 
```

Step5: Other commands for etcd are as to check health, member list etc, refer below
```
/ # ETCDCTL_API=3 etcdctl  member list  --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kub
ernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key 
181ffe2d394f445b, started, kmaster, https://192.168.56.101:2380, https://192.168.56.101:2379
/ # ETCDCTL_API=3 etcdctl endpoint health  --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/
kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key 
https://[127.0.0.1]:2379 is healthy: successfully committed proposal: took = 1.62473ms
/ # 
```

Step6:  Exit from pod and copy db snapshot on host
```
root@kmaster:~# kubectl cp kube-system/etcd-kmaster:/snapshotdb ./snapshotdb
tar: removing leading '/' from member names
root@kmaster:~# ls snapshotdb 
snapshotdb
```
	
## static pods
Step1: copy file static-pod.yaml at /etc/kubernetes/manifests/static-pod.yaml on any of the worker node.

Step2: After few seconds you will notice that pod 'static-pod' has started on the node where you have copied the file.
```
root@kmaster:~/demos/demos/module5# kubectl  get po -o wide
NAME                READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
static-pod-knode1   1/1     Running   0          10s   10.244.1.111   knode1   <none>           <none>
```

Step3: try to delete the pod and see what happens
```
root@kmaster:~/demos/demos/module5# kubectl  delete po static-pod-knode1
pod "static-pod-knode1" deleted
root@kmaster:~/demos/demos/module5# kubectl  get po 
NAME                READY   STATUS    RESTARTS   AGE
```

Step4: So, static pods can't be deleted, now remove file  /etc/kubernetes/manifests/static-pod.yaml from your worker node and then see if pod got deleted
```
root@kmaster:~/demos/demos/module5# kubectl  get po -o wide
No resources found.
```
