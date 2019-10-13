NOTE: This is just a suggestive list of questions to try for your CKA exam. Trying these questions will immensly help in clearing the exam:

There are no answers given as this is hands-on exam. You are expected to solve these questions on a running k8s machine

1. Create a node that has a SSD and label it as such.
2. Create a pod that is only scheduled on SSD nodes.
3. Create 2 pod definitions: the second pod should be scheduled to run anywhere the first pod is running - 2nd pod runs alongside the first pod
4. Create a deployment running nginx version 1.12.2 that will run in 2 pods 
    a. Scale this to 4 pods. 
    b. Scale it back to 2 pods. 
    c. Upgrade this to 1.13.8 
    d. Check the status of the upgrade 
    e. How do you do this in a way that you can see history of what happened? 
    f. Undo the upgrade
5. Create a daemon-set
6. Create a static pod
7. Create a busybox container without a manifest. Then edit the manifest.
8. Create a pod that uses secrets 
    a. Pull secrets from a volume 
    b. Dump the secrets out via kubectl to show it worked
9. Create a deployment with 'nginx' image and 2 replicas ,expose it using NodePort service
10. Create a deployment with 'nginx' image and 2 replicas ,expose it using NodePort service
11. Create a horizontal autoscaling group that starts with 2 pods and scales when CPU usage is over 50%.
12. Get logs for a pod
13. Deploy a pod with the wrong image name (like --image=nginy) and find the error message.
14. Get logs for kubectl
15. Get logs for the scheduler
16. Restart kubelet
17. Backup an etcd cluster
18. List the members of an etcd cluster
19. Find the health of etcd
20. Create a container with resource limit .5 CPU and 1 CPU as request and limit
21. run ```kubectl top node```
22. Verify and fix cluster node failure due to kubelet service issue
23. make an api call using curl
24. Create a pod that uses configmap 
    a. Pull configvalues from a volume 
25. Create a scratch disk and use it inside pod
26. Verify kubelet service logs 
27. Liveness probe
28. Readinesss probe
29. kubectl cp command 
30. run ```kubectl top pod```
31. create a namespace using kubectl 
32. create a replicaset with 2 replicas and then scale it to 4 replicas

  
