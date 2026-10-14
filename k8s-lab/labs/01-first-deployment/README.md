# Lab 01: Your first Deployment

1. Check the cluster: `kubectl get nodes` (expect 1 control-plane, 2 workers).
2. Apply the manifest: `kubectl apply -f deployment.yaml`
3. Watch the pods start: `kubectl get pods -o wide -w`
4. Which nodes did the pods land on? Why?
5. Delete one pod: `kubectl delete pod <name>`. What happens, and why?
6. Break it: change the image to `nginx:doesnotexist`, re-apply, then use
   `kubectl describe pod` to find the error. Fix it.
7. Check your work: `bash verify.sh`
