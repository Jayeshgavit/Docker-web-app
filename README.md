# Docker Web App

A hands-on Docker learning project: package a single HTML website into an image, serve it with Nginx, and manage its container locally on Windows 11.

The focus is understanding each step, from writing a Dockerfile to inspecting logs and restarting a container. No application dependencies, Docker Compose, or cloud deployment are needed.

## What you will learn

- How a Dockerfile becomes an image and how an image becomes a container.
- Why this project uses the official `nginx:alpine` image.
- How build context, file copying, port publishing, and container names work.
- How to verify, inspect, stop, and start your website.

## Project files

```text
Docker-web-app/
├── index.html         # Website, styles, and browser-side interaction
├── Dockerfile         # Instructions for packaging the website
├── README.md          # Practical learning walkthrough
└── ARCHITECTURE.md    # Build flow, request flow, and component explanations
```

See [the architecture guide](ARCHITECTURE.md) for diagrams and interview revision notes.

## Before you begin

Use PowerShell with Git installed and Docker Desktop running in Linux container mode. Run each command separately and read its output before continuing.

If PowerShell cannot recognize `docker`, but Docker Desktop is installed at its standard location, define this helper for the current terminal session:

```powershell
function docker { & 'C:\Program Files\Docker\Docker\resources\bin\docker.exe' @args }
```

This forwards commands to the executable without changing your system PATH.

## 1. Open the project

For a fresh checkout, run:

```powershell
git clone https://github.com/Jayeshgavit/Docker-web-app.git
cd Docker-web-app
```

If you already have the project, open that folder instead. Do not clone over an existing project. For the original learning workspace:

```powershell
cd 'D:\Goal\Docker Flow'
```

## 2. Check Docker Engine

```powershell
docker version
```

Expected: both `Client` and `Server` sections. The client accepts your commands; the server builds images and runs containers. A client section alone does not prove the engine is available.

## 3. Understand the Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

`FROM` selects the official Nginx image's Alpine variant. Alpine is a minimal Linux distribution; this image supplies Nginx and the userspace it needs. A container shares its Docker host's kernel rather than running its own kernel. Docker Desktop provides the Linux environment on Windows.

`COPY` puts the local HTML into Nginx's default document directory inside the image, replacing the base image's welcome page. It does not modify your original file. Nginx's startup configuration is inherited, so this Dockerfile needs no custom `CMD`.

## 4. Build the image

First check for an existing image with the intended name:

```powershell
docker images my-first-website
```

If a matching image exists, inspect it before rebuilding: a build with the same tag moves that tag to the newly built image.

```powershell
docker build -t my-first-website .
```

- `build` creates an image using the Dockerfile.
- `-t my-first-website` names it `my-first-website:latest`.
- `.` supplies the current directory as the build context, which makes `index.html` available to `COPY`.

Expected: build steps complete and the image is named successfully. The first build may download the base image. Building does not start the website. `latest` is a default tag, not a guarantee that an image is current.

## 5. Run the website

Check for an existing container before creating one:

```powershell
docker ps -a --filter "name=^/my-first-container$"
```

If a row appears, inspect its status, image, and ports. Use the existing container if appropriate; do not delete it to resolve a naming conflict. If there is no matching container, run:

```powershell
docker run -d --name my-first-container -p 127.0.0.1:8080:80 my-first-website
```

| Argument | Purpose |
| --- | --- |
| `run` | Create and start a new container. |
| `-d` | Run in the background and print its ID. |
| `--name my-first-container` | Assign a name for subsequent commands. |
| `-p 127.0.0.1:8080:80` | Publish host loopback port 8080 to container port 80. |
| `my-first-website` | Select the image to run. |

Expected: a container ID. Check the running status next; an ID alone does not prove the website stays available. The loopback binding keeps this demo local to your computer.

## 6. Verify the website

```powershell
docker ps
```

Expected: `my-first-container` has an `Up` status and the mapping `127.0.0.1:8080->80/tcp`.

```powershell
Invoke-WebRequest -Uri 'http://localhost:8080' -UseBasicParsing
```

Expected: HTTP `200` and the website HTML in `Content`. Open [http://localhost:8080](http://localhost:8080) in your browser to check the page and click counter. The counter runs in the browser and resets on refresh.

## 7. Practice the container lifecycle

Run these one at a time and observe what changes:

| Command | What it tells you or changes |
| --- | --- |
| `docker images` | Lists locally stored images. |
| `docker ps` | Lists running containers. |
| `docker ps -a` | Includes stopped containers. |
| `docker logs my-first-container` | Prints captured startup and request logs. Visit the page first to generate a request. |
| `docker stop my-first-container` | Stops Nginx and the container without deleting either the container or image. |
| `docker start my-first-container` | Starts the same stopped container with its existing port mapping. |

After stopping, the website is unavailable and the container remains visible in `docker ps -a`. After starting, verify it again with `docker ps` and an HTTP request.

`docker run` creates a new container; `docker start` reuses an existing one. Stopped containers still reserve their names.

## Troubleshooting

| Symptom | Next step |
| --- | --- |
| `docker` is not recognized | Use the PowerShell helper above or fix PATH, then reopen the terminal. |
| Engine connection fails | Open Docker Desktop, wait for its engine, and retry `docker version`. |
| Container name is already in use | Inspect with `docker ps -a --filter "name=^/my-first-container$"`. Do not repeat `run`. |
| Port 8080 is already allocated | Inspect `docker ps` and identify the listener before stopping anything. Do not terminate an unrelated service. |
| Container exits immediately | Read `docker logs my-first-container` and inspect its status with `docker ps -a`. |
| HTML edits do not appear | `COPY` takes a snapshot at build time. Rebuild and create a new container from the updated image; restarting the old container does not update it. |

## Scope and next experiments

This is a local learning demo, not a production deployment. There is no database, persistent application storage, TLS setup, or cloud hosting. The `alpine` tag can change over time; production projects should choose deliberate versioning and update policies.

Next experiments: add a CSS asset and update `COPY`, learn bind mounts and volumes, then package a frontend and backend as separate services. Keep credentials out of images and source control.

## Official references

- [Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/)
- [Running a container](https://docs.docker.com/reference/cli/docker/container/run/)
- [Publishing ports](https://docs.docker.com/engine/network/port-publishing/)
