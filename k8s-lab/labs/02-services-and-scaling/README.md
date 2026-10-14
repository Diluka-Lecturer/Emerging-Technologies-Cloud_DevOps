# Lab 02: Services and scaling

Requires lab 01's `web` Deployment.

1. Create a NodePort Service: `kubectl apply -f service.yaml`
2. Open http://localhost:8080 (forwarded from the cluster). You should see the nginx welcome page.
3. Scale up: `kubectl scale deploy web --replicas=5`. Check with `kubectl get endpoints web`.
4. Rolling update: `kubectl set image deploy/web nginx=nginx:1.27-alpine`
   then `kubectl rollout status deploy/web`. Roll it back with `kubectl rollout undo deploy/web`.
5. Check your work: `bash verify.sh`
