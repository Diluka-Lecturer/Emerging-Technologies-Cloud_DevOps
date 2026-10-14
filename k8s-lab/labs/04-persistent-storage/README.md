# Lab 04: Persistent storage

Containers are disposable. Data should not be.

1. Create a claim and a pod that appends a timestamp to a file every 5 seconds:
   ```bash
   kubectl apply -f pvc.yaml -f logger.yaml
   kubectl get pvc,pv
   ```
2. Read the file: `kubectl exec deploy/logger -- cat /data/log.txt`
3. Delete the pod: `kubectl delete pod -l app=logger`. Wait for the new one, then read the file again.
   Is the earlier data still there? Compare with an `emptyDir` volume (change `logger.yaml` and try).
4. Which node holds the data? (`kubectl get pv -o yaml`, look at `nodeAffinity`.) What does that mean
   for scheduling, and why does it not work for real multi-node clusters without network storage?
5. Check your work: `bash verify.sh`
