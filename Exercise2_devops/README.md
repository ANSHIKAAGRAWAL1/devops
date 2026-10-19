# Exercise 2: Deploy Flask on Minikube Using kubectl and YAML

> The requested notes file, `2-Minikube-Kubectl-Flask.md`, was not present in the workspace or tracked repository when this was completed. This guide follows the instructions supplied in the exercise request.

## Objective

Build and deploy the existing Flask application as a Kubernetes Deployment, expose it with a NodePort Service, then access the response from a browser or `curl`. The Flask endpoint returns `Hello from Flask on Kubernetes!` on port 15000. This is the small web-service pattern behind a storefront such as Zepto: the Pod runs the application and a Service gives clients a stable way to reach it.

## Existing project files

```text
Exercise2_devops/
├── Dockerfile
├── README.md
├── app.py
├── build-output.png
├── deploy-output.png
├── flask-deployment.yaml
├── flask-output.png
└── requirements.txt
```

`app.py` was already present and already implements the required `/` response and `0.0.0.0:15000` listener; it was reused unchanged. The Dockerfile was updated to use Python 3.12 and install the pinned Flask dependency. The combined YAML defines the `flask-app` Deployment and `flask-app-service` NodePort Service. The existing PNG files were preserved and not regenerated; they are not treated as proof of the current cluster state.

## Prerequisites

Install Docker Desktop or another driver supported by Minikube, Minikube, kubectl, and Python 3.10+ with pip for local checks. Verify the tools:

```powershell
docker version
minikube version
kubectl version --client
```

Before starting Minikube, inspect existing profiles and workloads. Do not reset or delete an existing cluster as part of this lab.

## Start Minikube

```powershell
minikube profile list
minikube start
minikube status
kubectl cluster-info
kubectl get nodes
```

Use an existing profile only after checking its status and resources. If Docker is the driver, ensure Docker Desktop is running before starting Minikube.

## Build the Flask image

From a PowerShell session at the repository root, build the image from the existing exercise folder:

```powershell
Push-Location '.\Exercise2_devops'
docker build -t flask-app:latest .
Pop-Location
```

This guide uses the Minikube image-load method, which is convenient when Docker and Minikube use separate image stores:

```powershell
minikube image load flask-app:latest
minikube image ls | Select-String 'flask-app'
```

The Deployment specifies `imagePullPolicy: Never`, so the image must be present in the Minikube node runtime before applying the manifest. Confirm the image listing before deployment.

On a supported Linux shell, an alternative is to build directly into Minikube's Docker daemon:

```sh
eval "$(minikube docker-env)"
docker build -t flask-app:latest .
```

Run those commands from `Exercise2_devops` in the same shell that will access that Docker environment. `eval $(minikube docker-env)` is a Unix-shell command; it is not a PowerShell command. When using this alternative, the build goes directly into Minikube's image store, so do not also treat a separate host Docker image as sufficient. The deployment's `Never` pull policy remains appropriate for the locally built image.

## Deploy with YAML

From the repository root:

```powershell
kubectl apply -f '.\Exercise2_devops\flask-deployment.yaml'
```

The file contains both the Deployment and Service. Verify the rollout, Pod, and Service:

```powershell
kubectl get deployments
kubectl get pods -l app=flask-app
kubectl describe deployment flask-app
kubectl get services
kubectl get endpoints flask-app-service
```

Wait for the Deployment to show `1/1` available and its Pod to show `Running` and `1/1` Ready before testing. If you prefer applying from inside `Exercise2_devops`, use `kubectl apply -f flask-deployment.yaml` instead.

## Imperative alternative

The imperative equivalent is:

```powershell
kubectl create deployment flask-app --image=flask-app:latest
kubectl patch deployment flask-app --type=json -p '[{"op":"add","path":"/spec/template/spec/containers/0/ports","value":[{"containerPort":15000,"protocol":"TCP"}]},{"op":"add","path":"/spec/template/spec/containers/0/imagePullPolicy","value":"Never"}]'
kubectl expose deployment flask-app --name=flask-app-service --type=NodePort --port=15000 --target-port=15000
```

Because the locally built image is used, ensure it is loaded into Minikube first. The patch declares the container port and sets `imagePullPolicy: Never`; without it, Kubernetes normally tries to pull a `:latest` image. The imperative and YAML approaches create resources with the same names; choose one approach rather than running both. If switching approaches, inspect existing resources and remove only the resources created for this exercise before recreating them.

## Logs and application access

Check the Service and application logs:

```powershell
kubectl get services
kubectl logs deployment/flask-app
```

If the installed kubectl does not support logs from a Deployment, get the actual Pod name and use it:

```powershell
kubectl get pods -l app=flask-app
kubectl logs <POD_NAME_FROM_THE_COMMAND_ABOVE>
```

Ask Minikube for the environment-specific Service URL:

```powershell
minikube service flask-app-service --url
```

This may keep a tunnel attached to the terminal; leave it open and run the request in another terminal if needed. Copy the URL actually returned, then test it:

```powershell
curl.exe <URL-RETURNED-BY-MINIKUBE>
```

Replace the placeholder with the full returned URL, for example the URL printed by your own Minikube instance. Do not assume the NodePort or localhost port is fixed. Expected body:

```text
Hello from Flask on Kubernetes!
```

## Troubleshooting

- **`ImagePullBackOff` or `ErrImageNeverPull`:** verify `docker images`, rebuild from `Exercise2_devops`, then run `minikube image load flask-app:latest` and verify with `minikube image ls`. `Never` means Kubernetes will not pull the image from a registry.
- **Pod is Pending:** run `kubectl describe pod <POD_NAME>` and inspect events, node readiness, and available resources with `kubectl get nodes`.
- **Pod is not Ready or keeps restarting:** inspect `kubectl logs deployment/flask-app` and `kubectl describe pod <POD_NAME>`. Confirm the app binds to `0.0.0.0:15000` and the container port is 15000.
- **Service has no endpoints:** compare the Deployment Pod label `app: flask-app` with the Service selector, and check that the Pod is Ready using `kubectl get pods --show-labels` and `kubectl get endpoints flask-app-service`.
- **Connection fails:** keep the `minikube service ... --url` terminal open if it owns the tunnel, use the exact returned URL, check that the Service and Pod are in the same namespace, and inspect host firewall/VPN settings.
- **Docker image is missing from Minikube:** the host Docker image store and Minikube's runtime may differ. Use the documented `minikube image load` method, or use the Unix `docker-env` alternative consistently.

## Service port Q&A

- **What is `port`?** The port exposed by the Kubernetes Service to its clients; this Service uses 15000.
- **What is `targetPort`?** The destination port on the selected Pod's container; this app listens on 15000. They happen to match in this lab, but they have distinct roles.
- **What does `minikube service flask-app-service --url` do?** It provides a reachable, environment-dependent URL for the NodePort Service and may establish a tunnel. Use the URL it prints rather than copying a sample port.

## Cleanup and existing screenshots

After confirming these resources belong to this exercise, remove only this manifest's Deployment and Service:

```powershell
kubectl delete -f '.\Exercise2_devops\flask-deployment.yaml'
```

This does not stop or delete the Minikube profile. `minikube stop` affects every workload in the profile; `minikube delete` destroys the profile's cluster state. Do not run either command against a shared/existing profile without explicitly confirming it is safe. The existing `build-output.png`, `deploy-output.png`, and `flask-output.png` were preserved; any screenshot may show a previous run and should not be presented as proof of a new runtime validation.

## Current verification status

The existing Python app matches the requested endpoint and port. The manifest structure and dependency/build changes were reviewed. Docker is currently unable to connect to its engine, and the active Minikube profile reports unknown state, so image building, manifest application, Pod readiness, and endpoint access were not verified in this session. Start/inspect Docker and Minikube, confirm the target cluster is safe to use, then follow the steps above.
