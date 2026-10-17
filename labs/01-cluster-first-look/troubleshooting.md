Symptom First diagnostic step
Docker unavailable      docker info
Cluster creation fails  kind get clusters and docker ps -a
Node not ready  kubectl describe node
Pod pending     kubectl describe pod -l app=demo-web
Image pull failure      kubectl describe pod -l app=demo-web
Rollout timeout kubectl rollout status deployment/demo-web
Port already in use     ss -ltnp
HTTP request fails      Check port-forward output and kubectl get pods
Avoid using sudo as a blanket fix for Docker permissions. Diagnose the specific error first.

