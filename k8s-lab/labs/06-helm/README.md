# Lab 06: Helm

Helm packages Kubernetes manifests as versioned, configurable charts.

1. Scaffold a chart: `helm create mychart` and look at the files it generated
   (`Chart.yaml`, `values.yaml`, `templates/`).
2. Install it:
   ```bash
   helm install demo ./mychart --set image.tag=1.27
   helm list
   kubectl get all -l app.kubernetes.io/instance=demo
   ```
3. Change config without editing templates:
   `helm upgrade demo ./mychart --set image.tag=1.27 --set replicaCount=3`
4. Look at history and roll back: `helm history demo`, then `helm rollback demo 1`.
5. Render without installing: `helm template demo ./mychart --set replicaCount=5 | less`.
   Where does the `5` end up?
6. Edit `mychart/values.yaml` so the chart deploys `nginx:1.27-alpine` with 2 replicas by default,
   then `helm upgrade demo ./mychart --reset-values`.
   Why `--reset-values`? Values you passed with `--set` earlier override `values.yaml` and can stick
   around between upgrades. Use `helm get values demo` to see what is overriding the file.
7. Check your work: `bash verify.sh`. Clean up with `helm uninstall demo`.
