# Exercise 3: Minikube Scaling Flask App with ReplicaSets

> The requested reference file, `3-Minikube-Scaling-Flask-App-with-Replicasets.md`, was not present in the workspace or tracked repository. This guide follows the assignment requirements in the request and the existing application/manifests.

## Objective

Run the Big Sale flash-sale Flask app in a Kubernetes ReplicaSet, scale from three to five replicas, and observe Kubernetes replace a deleted Pod. A ReplicaSet maintains a desired count of matching Pods; a Service provides a stable in-cluster endpoint and distributes requests among ready matching Pods.

This exercise uses a single-node Minikube profile, so all five replicas are expected to run on the same node. The ReplicaSet provides replica-level availability, not multi-node fault tolerance in this cluster.

## Existing project and files

The existing folder is named `ex3-MiniKube_Scaling` (capital `K`). It was reused without creating another exercise folder.

```text
ex3-MiniKube_Scaling/
├── DockerFile
├── README.md
├── app.py
├── build-output (2).png
├── flash-scale-Replicaset.yaml
├── replicaset-output.png
├── requirements.txt
├── scale-output.png
└── selfheal-output.png
```

- `app.py` already implemented `/`, `/buy`, and `/health`; it was reused, with `/buy` corrected to return its chosen value under the requested `product` key.
- `DockerFile` already used Gunicorn and the correct `app:app` module; it was updated to install the pinned requirements and expose port 5000.
- `flash-scale-Replicaset.yaml` already defines `flashsale-rs`, three replicas, matching labels, `flashsale:1.0`, local-image pull policy, probes, resource requests/limits, and the `flashsale-svc` ClusterIP Service. It was reviewed and kept unchanged.
- `requirements.txt` pins Flask and Gunicorn.
- The four PNGs existed before this change and were preserved; they are not evidence of a deployment performed now.

## Prerequisites

Install Docker Desktop (or another supported Minikube driver), Minikube, kubectl, and Python 3.10+ with pip for local checks. Check the tools:

```powershell
docker version
minikube version
kubectl version --client
```

Inspect existing Minikube profiles and resources before starting or changing anything:

```powershell
minikube profile list
minikube status
kubectl config current-context
kubectl get pods -A
```

Do not delete or reset an existing profile for this exercise. If a profile is running and shared, inspect its workloads before deploying.

## Run the application locally

From the repository root in PowerShell:

```powershell
Push-Location '.\ex3-MiniKube_Scaling'
py -m venv .venv
& .\.venv\Scripts\python.exe -m pip install -r requirements.txt
& .\.venv\Scripts\python.exe -c "from app import app; c=app.test_client(); print(c.get('/').get_json()); print(c.get('/buy?user=demo').get_json()); print(c.get('/health').get_json())"
Pop-Location
```

The root response includes a Big Sale welcome message, hostname, and timestamp. `/buy?user=demo` returns a successful purchase, a randomly selected Smartphone/Shoes/Headphones/Laptop product, the requested user, serving hostname, and time. `/health` returns healthy status and hostname.

## Start Minikube safely

Only start a profile after inspecting whether it already exists and confirming that using it will not disrupt other workloads:

```powershell
minikube profile list
minikube start
minikube status
kubectl get nodes
```

The start command was not run during this repository update because the existing profile's state could not be established. Do not run `minikube delete` or stop a shared cluster as troubleshooting shortcuts.

## Build and load the image

The ReplicaSet expects the exact image `flashsale:1.0`. In PowerShell, build with the existing case-sensitive filename from the project directory:

```powershell
Push-Location '.\ex3-MiniKube_Scaling'
docker build -t flashsale:1.0 -f DockerFile .
Pop-Location
```

The preferred local workflow for Windows/Docker Desktop is to load that image into Minikube:

```powershell
minikube image load flashsale:1.0
minikube image ls | Select-String 'flashsale:1.0'
```

The manifest uses `imagePullPolicy: Never`, so the image must be present in Minikube's node runtime before creating Pods. Do not assume the host Docker image is automatically visible in Minikube.

On Linux, another option is to build directly into Minikube's Docker daemon:

```sh
eval "$(minikube docker-env)"
cd ex3-MiniKube_Scaling
docker build -t flashsale:1.0 -f DockerFile .
```

`eval $(minikube docker-env)` is a Unix-shell command, not a PowerShell command. Use one image workflow consistently. If you choose Docker Hub instead, tag and push `flashsale:1.0`, update the manifest to reference the pushed registry image and an appropriate pull policy, then configure registry credentials only if the registry is private.

## Deploy and inspect

Apply the existing manifest from the repository root:

```powershell
kubectl apply -f '.\ex3-MiniKube_Scaling\flash-scale-Replicaset.yaml'
```

Inspect ReplicaSet, Pods, Service, and logs:

```powershell
kubectl get rs flashsale-rs
kubectl get pods -l app=flashsale -o wide
kubectl get services flashsale-svc
kubectl describe rs flashsale-rs
kubectl logs -l app=flashsale --tail=50
```

Wait for `3/3` Ready before scaling. In this single-node cluster, all three (and later all five) Pods should show the same node in `-o wide` output.

## Access the application

The Service is intentionally `ClusterIP`. Forward a local port to it from PowerShell:

```powershell
kubectl port-forward service/flashsale-svc 8080:80
```

Keep this terminal open and use another terminal or browser for requests:

```powershell
curl.exe http://127.0.0.1:8080/
curl.exe 'http://127.0.0.1:8080/buy?user=alice'
curl.exe http://127.0.0.1:8080/health
```

The responses should identify a serving Pod hostname. `/buy` should return `status: success`, a product from the four-item list, the supplied user, hostname, and time. The Service may send successive requests to different ready Pods.

## Scale and observe self-healing

Scale the correct ReplicaSet from three to five replicas:

```powershell
kubectl scale replicaset flashsale-rs --replicas=5
kubectl get rs flashsale-rs -w
```

In another terminal, verify the five Pods and their node placement:

```powershell
kubectl get pods -l app=flashsale -o wide
```

Delete one actual Pod name copied from that output:

```powershell
kubectl delete pod <POD_NAME_FROM_GET_PODS>
kubectl get pods -l app=flashsale -w
```

The ReplicaSet should create a replacement so the desired count returns to five. Confirm with `kubectl get rs flashsale-rs` and `kubectl get pods -l app=flashsale -o wide`. Do not delete unrelated Pods.

## Six exercise questions and answers

The original notes containing the exact question wording were not available. These answers cover the six concepts requested by the exercise objective:

1. **What does a ReplicaSet do?** It maintains the configured number of Pods matching its selector, creating or removing Pods as necessary.
2. **How do you scale the flash sale from three to five Pods?** Run `kubectl scale replicaset flashsale-rs --replicas=5` and verify `DESIRED`, `CURRENT`, and `READY` with `kubectl get rs flashsale-rs`.
3. **What happens when a managed Pod is deleted?** The ReplicaSet observes that actual replicas are below the desired count and creates a replacement Pod.
4. **Why use a Service?** Pod IPs are ephemeral; the Service provides a stable endpoint and routes traffic to ready Pods selected by `app: flashsale`.
5. **How is the application reached through the ClusterIP Service?** Use `kubectl port-forward service/flashsale-svc 8080:80`, then request `http://127.0.0.1:8080` locally.
6. **Where will five Pods run in this lab's Minikube setup?** All are expected on the same node because the cluster is single-node; scaling replicas does not add nodes.

## Troubleshooting

- **`ImagePullBackOff` / `ErrImageNeverPull`:** confirm the manifest image is `flashsale:1.0`, build with that exact tag, load it using `minikube image load flashsale:1.0`, and inspect `minikube image ls`.
- **Pods not Ready:** use `kubectl describe pod <POD_NAME>` and `kubectl logs <POD_NAME>`. Check readiness/liveness `/health`, container port 5000, and whether the app starts under Gunicorn.
- **ReplicaSet has fewer than the desired count:** inspect `kubectl describe rs flashsale-rs` and cluster capacity with `kubectl get nodes`; a single node can still run five Pods if it has sufficient resources.
- **Service access fails:** check `kubectl get endpoints flashsale-svc`, confirm the Pod label and Service selector both use `app: flashsale`, keep the `kubectl port-forward` process running, and use local port 8080.
- **Image mismatch after rebuild:** rebuild and reload `flashsale:1.0`, then restart only this ReplicaSet's Pods if needed after confirming the manifest and image are correct.

## Cleanup

After confirming these resources belong to this exercise, remove only its manifest resources:

```powershell
kubectl delete -f '.\ex3-MiniKube_Scaling\flash-scale-Replicaset.yaml'
```

This removes the `flashsale-rs` ReplicaSet and `flashsale-svc` Service, not the Minikube profile. Do not use broad deletion commands. Stopping or deleting Minikube affects the entire profile; inspect for other workloads and obtain explicit confirmation before doing so.

## Existing screenshots and verification status

The existing build/scaling/self-healing PNGs remain unchanged. Docker currently cannot connect to its engine, and the Minikube profile reports unknown state, so no new image build, ReplicaSet rollout, scale operation, replacement Pod, or HTTP request was performed. Use the commands above after Docker and the intended Minikube profile are confirmed healthy.
