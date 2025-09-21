---
title: "Docker Volumes and Bind Mounts Explained"
date: 2025-09-21T00:00:00Z
description: "A practical guide to Docker volumes, anonymous and named volumes, and bind mounts for persistent and shared data."
draft: false
---

When you first start working with Docker, it feels magical. You don’t need to worry about dependencies, installed software, or other setup details. Docker allows you to run, test, develop, and deploy applications in a consistent environment.

![Docker process example](https://dummyimage.com/600x200/cccccc/000000&text=Docker+process+example)

A container is still just a process inside Linux — not magic, just a powerful technology. For example:

```sh
docker run -d node sleep 1000
ps aux | grep node
```
Output might look like:

```
ubersholder 92774 0.0 0.1 41 24624 s000 S+ 0:0.04 docker run -it node sleep 100
```
This shows that Docker containers are processes on your host system.

## Why Volumes Are Needed
By default, containers are ephemeral — any data inside disappears when the container stops or is removed. Docker volumes solve this by providing a way to persist data beyond the container’s lifecycle.

Think of it this way:

- A container is like a running process.
- A volume is like a hard drive where your process saves its files.

Volumes allow you to:

- Persist data across container restarts or removals.
- Share data between multiple containers.
- Decouple storage from the container image, keeping images small and portable.

## Named Volumes
The simplest way to attach a volume is using the `-v` (or `--volume`) flag:

```sh
docker run -d --name test-container -v mydata:/app/data node:latest
```
- `mydata` → the name of the volume (Docker will create it if it doesn’t exist).
- `/app/data` → mount point inside the container.

Any files written to `/app/data` now live in the `mydata` volume on the host.

Check volumes:

```sh
docker volume ls
```
Output:

```
DRIVER    VOLUME NAME
local     mydata
```

## Anonymous Volumes
Anonymous volumes are created automatically without a name. They happen when you do not specify a volume name:

```sh
docker run -d -v /app/data node:latest sleep 1000
```
- Docker mounts an anonymous volume to `/app/data`.
- Docker assigns a random name (hash) to it.

Check volumes:

```sh
docker volume ls
```
Output might show something like:

```
DRIVER    VOLUME NAME
local     37334d8ec7e87353fbab6…
local     mydata
```

**Key points:**

- Anonymous volumes survive container removal.
- Harder to manage or reference.
- Useful for temporary caches or logs.

## Named Volume in Action
Run a container with a named volume:

```sh
docker run -d --name node-vol-demo -v mydata:/app/data node:latest sleep 1000
```
Enter the container:

```sh
docker exec -it node-vol-demo bash
```
Write some data:

```sh
echo "persistent data test" > /app/data/hello.txt
cat /app/data/hello.txt
exit
```
Remove the container:

```sh
docker rm -f node-vol-demo
```
Run a new container with the same volume:

```sh
docker run -it --rm -v mydata:/app/data node:latest cat /app/data/hello.txt
```
Output:

```
persistent data test
```
✅ Named volumes survive restarts and container removal. As long as you reference the same volume name, your data remains available.

## Small Bonus: What is a Docker Bind Mount?
A bind mount links a folder or file from your host directly into a container. Unlike Docker volumes, which Docker manages, bind mounts give you full control over the files on your host.

**Key points:**

- Changes on the host are instantly visible in the container.
- Changes inside the container are instantly reflected on the host.
- Great for development, configuration sharing, and backups.
- Host files override container files if paths overlap.

Think of a bind mount as a bridge between your computer and the container.

### Practical Bind Mount Example: Live Development
Create a host folder and file:

```sh
mkdir host-app
echo "console.log('Hello from host');" > host-app/index.js
```
Run a container and mount the folder:

```sh
docker run -it --name live-dev -v $(pwd)/host-app:/app node:latest bash
```
Run the app inside the container:

```sh
node /app/index.js
```
Output:

```
Hello from host
```
Edit the file on your host:

```sh
echo "console.log('Updated live');" >> host-app/index.js
```
Run it again inside the container:

```sh
node /app/index.js
```
Output:

```
Hello from host
Updated live
```
✅ Any changes on the host are immediately reflected inside the container — perfect for development.

Cleanup:

```sh
docker rm -f live-dev
rm -rf host-app
```

## Summary: Docker Volumes and Bind Mounts
Docker containers are ephemeral by default — any data inside disappears when the container stops or is removed. To persist data, Docker provides volumes and bind mounts.

1. **Anonymous Volumes**
   - Automatically created, no name.
   - Good for temporary or cache data.
   - Survive container removal but hard to manage.
2. **Named Volumes**
   - Created with a specific name.
   - Ideal for persistent data: databases, uploads, configs.
   - Survive restarts and container removal; reusable across containers.
3. **Bind Mounts**
   - Directly connect a host folder or file to a container.
   - Changes on the host are instantly visible in the container and vice versa.
   - Perfect for development, configuration sharing, and backups.
   - Host files override container files if paths overlap.

**Key Takeaways:**

- Anonymous volumes → temporary storage.
- Named volumes → persistent, reusable storage.
- Bind mounts → real-time host-container sync for development or file sharing.

By understanding volumes and bind mounts, you can control where your data lives, ensure persistence, and streamline development workflows.

If you like the article, drop a like, comment and feel free to checkout my FREE gumroad digital list of resources to ACE YOUR DEVOPS INTERVIEW.
