# Lab 05: Probes, limits and self-healing

1. Deploy: `kubectl apply -f healthy.yaml` and wait for 3 ready pods.
2. **Readiness.** Break one pod's content so nginx returns 403:
   ```bash
   POD=$(kubectl get pod -l app=healthy -o name | head -1)
   kubectl exec $POD -- rm /usr/share/nginx/html/index.html
   kubectl get pods -w
   kubectl get endpoints healthy
   ```
   The pod stays Running but becomes `0/1` Ready and leaves the Service endpoints. Why is it not restarted?
3. **Liveness.** Kill nginx in another pod and watch the restart count:
   `kubectl exec <pod> -- nginx -s stop` then `kubectl get pods`.
4. **Crash loops.** Apply `crashloop.yaml`. Use `kubectl get pods`, `kubectl logs` and
   `kubectl describe pod` to find out why it fails, and fix it.
5. **Limits.** Apply `memhog.yaml`. What status does the pod get, and what does `describe` show as the reason?
   Raise the limit so it survives.
6. Check your work: `bash verify.sh`
