[Main](../README.md)
---

# Docker

## What is Docker?

Docker is a platform that allows you to package applications with all their dependencies into containers. These containers are:
	•	Lightweight
	•	Portable
	•	Isolated

Verify Docker is running:

```bash
docker --version
docker info
```

## Run Your First Container

```bash
docker run hello-world
```

### What it does:
	•	Downloads the hello-world image if it doesn’t exist locally.
	•	Starts a container that prints a hello message.
	•	Stops and removes the container automatically after the message.


## Run an interactive Ubuntu container:

```bash
docker run -it ubuntu bash
```

Inside the container:

```bash
apt update
apt install curl
curl https://example.com
exit
```

Now list containers:

```bash
docker ps -a
```

## Image vs Container
	•	Image = Blueprint
	•	Container = Running instance of the image

## Inspect images:

```bash
docker images
docker image inspect <image>
```


## Build Your Own Docker Image

Create a Dockerfile:

```dockerfile
FROM alpine
CMD ["echo", "My first Docker image!"]
```

Build and run:

```bash
docker build -t myimage .
docker run myimage
```

## Container Lifecycle

```bash
docker run -d --name test-container nginx
docker stop test-container
docker inspect test-container
docker start test-container
docker rm test-container
```

## Dockerfile Deep Dive

A list of commonly used Dockerfile commands:

⸻

🧱 1. FROM

📄 Description:

Specifies the base image to use for the Docker image.

🔧 Example:

```dockerfile
FROM ubuntu:20.04
```

➡️ Starts building the image from the official Ubuntu 20.04 image.

⸻

📁 2. WORKDIR

📄 Description:

Sets the working directory for any subsequent instructions in the Dockerfile.

🔧 Example:
```dockerfile
WORKDIR /app
```
➡️ Creates and switches to /app directory inside the image.

⸻

📄 3. COPY

📄 Description:

Copies files and directories from the build context into the image.

🔧 Example:
```dockerfile
COPY . /app
```
➡️ Copies everything in the current directory on host into /app in the image.

⸻

🌍 4. ADD

📄 Description:

Similar to COPY, but can also:
	•	Extract tar archives
	•	Fetch remote URLs

🔧 Example:
```dockerfile
ADD my-archive.tar.gz /data/
```
➡️ Extracts the archive into /data inside the image.

⸻

🛠️ 5. RUN

📄 Description:

Executes a shell command during image build.

🔧 Example:
```dockerfile
RUN apt update && apt install -y curl
```
➡️ Installs curl during build time.

⸻

🧾 6. CMD

📄 Description:

Specifies the default command to run when a container starts (can be overridden).

🔧 Example:
```dockerfile
CMD ["node", "app.js"]
```
➡️ Executes node app.js by default when container starts.

⸻

⚙️ 7. ENTRYPOINT

📄 Description:

Sets the main command to run; arguments passed to docker run are passed to this.

🔧 Example:
```dockerfile
ENTRYPOINT ["python3"]
CMD ["app.py"]
```
➡️ Runs python3 app.py. If you run docker run image file.py, it runs python3 file.py.

⸻

💬 8. ENV

📄 Description:

Sets environment variables.

🔧 Example:
```dockerfile
ENV APP_ENV=production
```
➡️ Sets the APP_ENV environment variable to “production”.

⸻

🔒 9. EXPOSE

📄 Description:

Documents the port the container listens on (does not actually publish it).

🔧 Example:
```dockerfile
EXPOSE 80
```
➡️ Tells Docker that the app inside the container uses port 80.

⸻

👤 10. USER

📄 Description:

Sets the user that the container will use to run.

🔧 Example:
```dockerfile
USER appuser
```
➡️ Executes commands as appuser.

⸻

⌨️ 11. ARG

📄 Description:

Defines variables that are available only at build-time.

🔧 Example:
```dockerfile
ARG VERSION=1.0
RUN echo $VERSION
```
➡️ Allows passing build-time variables using --build-arg.

⸻

🔃 12. VOLUME

📄 Description:

Declares a volume mount point inside the container.

🔧 Example:
```dockerfile
VOLUME /data
```
➡️ This directory will be managed as a persistent volume.

⸻

🔚 13. LABEL

📄 Description:

Adds metadata to an image.

🔧 Example:
```dockerfile
LABEL maintainer="Zafar Saidov <zafar@example.com>"
```
➡️ Adds descriptive info for the image.

⸻

🔗 14. SHELL

📄 Description:

Defines the shell to use for RUN commands (default is ["/bin/sh", "-c"]).

🔧 Example:
```dockerfile
SHELL ["/bin/bash", "-c"]
```
➡️ Use bash instead of sh for shell commands.

⸻

Example Dockerfile with All Key Commands:
```dockerfile
FROM node:18
LABEL maintainer="Zafar Saidov <zafar@example.com>"

WORKDIR /app
COPY . /app

RUN npm install

ENV NODE_ENV=production

EXPOSE 3000

USER node

CMD ["node", "server.js"]
```

## Docker Volumes & Port Mapping

Run a web server and map ports:

```bash
docker run -d -p 8080:80 nginx
```

Create a volume:

```bash
docker volume create mydata
docker run -v mydata:/data alpine touch /data/file.txt
```

[Main](../README.md)