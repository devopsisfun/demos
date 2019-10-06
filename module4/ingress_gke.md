Ingress controller in GKE

1. Visit the Kubernetes Engine page in the Google Cloud Platform Console.
2. Create or select a project.
3. Create a kubernetes cluster: gcloud container clusters create loadbalancedcluster

Now Let's do web application deployment, then create NodePort service and then deploy ingress

Step 1: Deploy a web application
	wget https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer/web-deployment.yaml
	kubectl apply -f web-deployment.yaml

Step 2: Expose your Deployment as a Service internally
	https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer/web-service.yaml
	kubectl apply -f web-service.yaml

	--> Check service
	kubectl get service web

	At this stage, no external ip is assigned

Step 3: Create an Ingress resource
	https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer/basic-ingress.yaml
	kubectl apply -f basic-ingress.yaml

Step 4: Visit your application
	kubectl get ingress basic-ingress

