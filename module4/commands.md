## Command reference for labels

# Check labels assigned to a pod
kubectl get po --show-labels

# add labels to a pod
kubectl label pod label-pod env=dev

# add more than 1 label to a pod
kubectl label pod new-nginx-5cd8c45c86-257lm env=dev owner=devops

# remove a label from a pod
kubectl label pod label-pod env-

# overwrite an existing label
kubectl label pod label-pod --overwrite status=beta

# show all pods with label , operator based selection
kubectl get po -l status=alpha
kubectl get po -l status!=alpha

# set based selection
kubectl get po -l 'status in (ga,beta)'

## Command reference for replica sets, replication controller, deployment

# scale a replicaset
kubectl scale --replicas=2 rs/web

# create deployment without yaml file
kubectl create test-deployment --image=nginx --replicas=2

# expose a deployment
kubectl expose deployment deploymentName –type=NodePort –port=80

# record history of deployment
kubectl create -f deployment.yaml --record 

# change image of a deployment
kubectl set image deployment webserver-deployment frontend=httpd:2.4 --record

# check change history
kubectl rollout history deployment webserver-deployment

# rollback to last revision or undo
kubectl rollout undo deployment webserver-deployment

# rollback to specific revision 
kubectl rollout undo deployment webserver-deployment --to-revision=1

# scale deployment
kubectl scale deployment webserver-deployment --replicas=6

kubectl scale deployment webserver-deployment --replicas=3

# autoscale deployment
kubectl autoscale deployment webserver-deployment --min=6 --max=10 --cpu-percent=70


