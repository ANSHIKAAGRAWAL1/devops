# Exercise 4: Docker Networking

> The reference file `4-Docker-Networking.md` was not found in the workspace or repository. This guide follows the instructions in the task.

## Objective and existing files

Build a Flask REST API and demonstrate communication between MySQL, Redis, and Flask containers on the user-defined bridge `my-bridge-net`. The API serves `GET /about` on container port 5001. The database and cache demonstrate container networking; Flask does not use them internally.

```text
Ex4-Docker-Networking/
├── DockerFile
├── README.md
├── Requirements.txt
├── app.py
├── build-output.png
├── containers-output.png
└── network-output.png
```

The existing `app.py` already returned the requested JSON and listened on `0.0.0.0:5001`; it was reused with debug mode disabled. The existing Dockerfile and capitalized `Requirements.txt` were retained and made consistent. Existing PNGs were preserved and may show a previous run, not current validation. There was no Compose file, and none is needed.

## Prerequisites

Use Docker Desktop with its engine running and Windows PowerShell. Check Docker:

```powershell
docker version
docker info
```

Inspect for resource conflicts before creating containers or the network:

```powershell
docker ps -a --filter "name=^/mysql$"
docker ps -a --filter "name=^/redis$"
docker ps -a --filter "name=^/flask$"
docker network ls
```

Inspect existing resources before reuse or removal. If names conflict with another project, select different names and update every command consistently.

## Build the Flask image

From the repository root:

```powershell
Push-Location '.\Ex4-Docker-Networking'
docker build -t flask-api:latest -f DockerFile .
Pop-Location
docker image inspect flask-api:latest
```

The Dockerfile installs the pinned Flask dependency, copies `app.py`, exposes 5001, and starts the app. The image tag is `flask-api:latest` throughout this guide.

## Create and inspect the bridge

```powershell
docker network create --driver bridge my-bridge-net
docker network ls
docker network inspect my-bridge-net
```

If the network already exists, inspect it and use it only if it belongs to this exercise. Do not delete an unknown network.

## Launch MySQL, Redis, and Flask

MySQL requires a root password to initialize. Enter a temporary, non-production password when prompted; do not commit or reuse it:

```powershell
$mysqlRootPassword = [System.Net.NetworkCredential]::new('', (Read-Host -AsSecureString 'Temporary MySQL root password')).Password
docker run -d --name mysql --network my-bridge-net --env "MYSQL_ROOT_PASSWORD=$mysqlRootPassword" mysql:8.4
Remove-Variable mysqlRootPassword
```

MySQL normally listens on container port 3306. Start Redis (normally port 6379) on the same bridge:

```powershell
docker run -d --name redis --network my-bridge-net redis:7-alpine
```

Start Flask on that network and publish its port to the host:

```powershell
docker run -d --name flask --network my-bridge-net -p 5001:5001 flask-api:latest
docker ps
docker network inspect my-bridge-net
```

The shared user-defined bridge provides container-name DNS, so peers can resolve `mysql` and `redis`. This only demonstrates connectivity; the Flask app does not connect to either service.

## Test the Flask API

From the host:

```powershell
Invoke-RestMethod http://localhost:5001/about | ConvertTo-Json
```

Expected response:

```json
{
  "name": "Simple REST API",
  "version": "1.0",
  "description": "This is a simple REST API built with Flask."
}
```

Equivalent command: `curl.exe http://localhost:5001/about`.

## Verify container-to-container connectivity

Use Python's standard-library TCP socket from the Flask container, without assuming it has `ping`, Bash, `nc`, or database clients:

```powershell
docker exec flask python -c "import socket; s=socket.create_connection(('mysql',3306),5); print('mysql:3306 reachable'); s.close()"
docker exec flask python -c "import socket; s=socket.create_connection(('redis',6379),5); print('redis:6379 reachable'); s.close()"
```

Wait for MySQL initialization and inspect `docker logs mysql` if port 3306 refuses connections. If Flask is unavailable, a temporary diagnostic container can test both ports:

```powershell
docker run --rm --network my-bridge-net python:3.12-slim python -c "import socket; s=socket.create_connection(('mysql',3306),5); print('mysql:3306 reachable'); s.close(); s=socket.create_connection(('redis',6379),5); print('redis:6379 reachable'); s.close()"
```

These checks prove TCP reachability only, not authentication or application-level use.

## Bridge, host networking, and `-p`

- **User-defined bridge:** isolated virtual network with Docker DNS for attached containers; used by this exercise.
- **Host networking:** shares the host network namespace and does not provide the same isolation model; it is platform-dependent and unnecessary here.
- **`-p hostPort:containerPort`:** publishes a container port on the host. `-p 5001:5001` makes Flask reachable at `localhost:5001`; MySQL/Redis need no published host ports for this demonstration.

## Four questions and answers

The notes were unavailable; these answers address the networking requirements:

1. **Why use a user-defined bridge?** It isolates the containers and enables name-based DNS for attached containers.
2. **Why attach all services to the same network?** Members share network connectivity and can resolve names such as `mysql` and `redis`.
3. **What does `-p 5001:5001` do?** It maps host port 5001 to Flask's container port 5001.
4. **How does bridge networking differ from host networking?** Bridge mode provides an isolated virtual network; host mode shares the host namespace and has different isolation and platform behavior.

## Troubleshooting

- **Port 5001 is busy:** inspect containers/processes; do not stop unrelated services. Use another host port, e.g. `-p 5002:5001`, and request `http://localhost:5002/about`.
- **Image missing:** build from `Ex4-Docker-Networking` and verify `docker image inspect flask-api:latest`.
- **Container exits:** inspect `docker logs flask`, `docker logs mysql`, or `docker logs redis`; verify the Flask dependency and MySQL initialization password.
- **Container name already exists:** inspect ownership first; choose another name rather than deleting an unrelated container.
- **Network connection fails:** confirm all containers are attached to `my-bridge-net`, wait for MySQL readiness, and retry the socket check. Name resolution works only on the shared user-defined network.
- **Host request fails:** confirm `docker ps` shows the port mapping and Flask binds to `0.0.0.0:5001`; check firewall/VPN settings.

## Cleanup

After confirming these containers were created for this exercise, remove only them:

```powershell
docker rm -f flask redis mysql
```

Inspect the network, then remove it only if no other project uses it:

```powershell
docker network inspect my-bridge-net
docker network rm my-bridge-net
```

Optionally remove the image if it is not used elsewhere:

```powershell
docker image rm flask-api:latest
```

Do not use broad prune commands. No containers or networks were created or removed during this update.

## Current validation status

The `/about` endpoint was checked with Flask's test client in a disposable environment. Docker cannot currently connect to its engine, so the image build, container startup, host request, and bridge connectivity checks were not run. Start Docker Desktop, inspect for conflicts, and execute the commands above to validate the live setup.
