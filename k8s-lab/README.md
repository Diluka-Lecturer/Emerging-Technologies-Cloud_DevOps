# Kubernetes Lab

A ready-to-use, throwaway Kubernetes cluster (1 control-plane + 2 workers, via kind)
inside a dev container. No installs beyond Docker and VS Code.

## Student setup (about 5 minutes, once)

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine on Linux) and start it.
2. Install [VS Code](https://code.visualstudio.com/) and the **Dev Containers** extension.
3. Clone this repo, open it in VS Code, and choose **Reopen in Container** when prompted.
4. Wait for the terminal to print the node list. Check with `kubectl get nodes`.

No Docker, or a weak laptop? Open the repo on GitHub and choose **Code > Codespaces > Create**.
This gives the same environment in the browser.

## Useful commands

| Command | Purpose |
|---|---|
| `k9s` | Terminal UI for the cluster |
| `bash scripts/cluster-reset.sh` | Wipe and recreate the cluster |
| `bash labs/<lab>/verify.sh` | Check a lab |

`k` is aliased to `kubectl`.

## Requirements

Docker with at least 2 CPUs and 4 GB RAM allocated. First start downloads about 1 GB of images, so
instructors should ask students to do setup **before** the session (the campus Wi-Fi will thank you).

## Troubleshooting

- **"Cannot connect to the Docker daemon":** start Docker Desktop and wait until it says it is running.
- **`accessing specified distro mount service ... ubuntu.sock` on container start (Windows with WSL Ubuntu):**
  in VS Code settings, untick `dev.containers.mountWaylandSocket`, then reopen in the container.
- **Cluster is broken:** run `bash scripts/cluster-reset.sh`.

## Labs

- `labs/01-first-deployment`: Deployments, pods, scheduling
- `labs/02-services-and-scaling`: Services, scaling, rolling updates
- `labs/03-config-and-branding`: ConfigMaps, Secrets, serving a branded IFS x NSBM page
- `labs/04-persistent-storage`: PersistentVolumeClaims
- `labs/05-health-and-self-healing`: probes, crash loops, resource limits
- `labs/06-helm`: charts, upgrades, rollbacks

Labs 01-02 are the core; 03-06 can be done in any order. Reset between labs with
`bash scripts/cluster-reset.sh` if needed.

Instructor: add `ifs.png` and `nsbm.png` to `labs/03-config-and-branding/assets/` before sharing.

## Instructor notes

- Windows students need WSL2 enabled in Docker Desktop (the default).
- Apple Silicon and x86 both work. The install script picks the right architecture.
- Add labs as new folders under `labs/`, each with a README and a `verify.sh`.
