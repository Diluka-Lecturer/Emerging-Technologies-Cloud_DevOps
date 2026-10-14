# Lab 03: ConfigMaps, Secrets and a branded site

You will serve a custom web page (IFS x NSBM) from nginx **without building an image**, by
injecting the HTML and logos through ConfigMaps.

Put the logo files in `assets/` first (instructor: see `assets/README.md`).

1. Create the ConfigMaps from files:
   ```bash
   kubectl create configmap site-html --from-file=index.html
   kubectl create configmap site-logos --from-file=assets/ifs.png --from-file=assets/nsbm.png
   ```
   Run `kubectl get configmap site-html -o yaml` and find your HTML inside.
2. Deploy: `kubectl apply -f site.yaml`
3. Open http://localhost:8081
4. Edit `index.html` (change the heading), then update the ConfigMap:
   ```bash
   kubectl create configmap site-html --from-file=index.html --dry-run=client -o yaml | kubectl apply -f -
   ```
   Wait about a minute. Does the page change without restarting pods? Now try
   `kubectl rollout restart deploy/site` and compare. Why the difference?
5. Secrets: create one and read it back.
   ```bash
   kubectl create secret generic site-secret --from-literal=team=gamma
   kubectl get secret site-secret -o jsonpath='{.data.team}' | base64 -d
   ```
   Is base64 encryption? Who can read this in a real cluster, and how would you restrict that?
6. Check your work: `bash verify.sh`
