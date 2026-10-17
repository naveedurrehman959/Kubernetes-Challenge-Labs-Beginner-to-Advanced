kind create cluster --name k8s-lab --wait 120s

kubectl config current-context
kubectl get nodes
kubectl get namespaces
kubectl get pods -A

kubectl apply -f manifests/demo-deployment.yaml
kubectl rollout status deployment/demo-web
kubectl get deployments,replicasets,pods

kubectl port-forward deployment/demo-web 8080:80
# In a second terminal:
curl -I http://localhost:8080
curl http://localhost:8080

kubectl logs deployment/demo-web
kubectl describe deployment demo-web
kubectl delete pod -l app=demo-web
kubectl get pods -w

# After completing the lab:
kubectl delete -f manifests/demo-deployment.yaml
kind delete cluster --name k8s-lab
