
# CI/CD Pipeline Workflow

This GitHub Actions workflow automates the continuous integration and deployment (CI/CD) process for a project. It triggers on push events to the `master` branch. The workflow consists of environment variables, jobs, and steps to checkout code, build and push a Docker image, and deploy the application by SSH into a server.

## Workflow Name

`name: CI/CD Pipeline`

## Trigger

```yaml
on:
  push:
    branches:
      - master
```

Triggers the workflow on push events to the `master` branch.

## Environment Variables

```yaml
env:
  VITE_API_URL: ${{ secrets.API_URL }}
```

Defines environment variables using GitHub secrets for API URL.

## Jobs

### Build

Runs on the latest Ubuntu runner and includes steps for:

1. **Checkout code:** Checks out the repository code.
2. **Docker login:** Logs into Docker using secrets for Docker username and password.
3. **Build and push Docker image:** Uses `docker/build-push-action@v2` to build and push the Docker image. It tags the image and passes the API URL as a build argument.
4. **SSH into server:** Uses `appleboy/ssh-action@master` to SSH into the server, stops any running containers, pulls the latest Docker image, and runs the new image.

### Steps

Each step is defined under `jobs.build.steps` and performs specific actions such as checking out code, logging into Docker, building and pushing the Docker image, and deploying the application via SSH.

## Deployment

The deployment step stops any currently running containers on the server, logs into Docker, pulls the latest Docker image tagged as `web-latest`, and runs the new Docker image, exposing it on port 80.

---

This workflow provides a basic but comprehensive CI/CD pipeline setup suitable for Docker-based projects, automating build, push, and deployment processes.


# Dockerfile Overview

This Dockerfile creates a multi-stage build for a Node.js application with an Nginx server to serve the built static files. It's optimized for minimal size and efficient caching during the build process.

## Stages of Build

### Stage 1: Node.js Build Environment

- **Base Image:** `FROM node:16-alpine`
  - Starts with a lightweight Node.js 16 Alpine image.
- **Working Directory:** `WORKDIR /app`
  - Sets the working directory for the Docker image.
- **Arguments and Environment Variables:**
  - `ARG VITE_API_URL`
    - Declares a build-time argument `VITE_API_URL`.
  - `ENV VITE_API_URL=$VITE_API_URL`
    - Sets an environment variable `VITE_API_URL` that persists in the built image.
- **Dependencies Installation:**
  - `COPY package*.json ./`
    - Copies `package.json` and `package-lock.json` (if available) to the Docker image.
  - `RUN npm install --force`
    - Runs `npm install` to install dependencies, using `--force` to ensure installation if there are conflicts.
- **Build Application:**
  - `COPY . .`
    - Copies the application source code into the image.
  - `RUN npm run build`
    - Executes the build script defined in `package.json`, typically generating a `dist` folder with static files.

### Stage 2: Nginx Server Setup

- **Base Image:** `FROM nginx:1.23.3-alpine`
  - Switches to a lightweight Nginx Alpine image for serving the static files.
- **Static Files:**
  - `COPY --from=0 /app/dist /usr/share/nginx/html`
    - Copies the `dist` folder from the Node.js build stage into the Nginx server's html directory.
- **Nginx Configuration:**
  - `COPY nginx.conf /etc/nginx/conf.d/default.conf`
    - Copies a custom `nginx.conf` file into the default Nginx configuration directory. This configuration is essential for customizing Nginx behavior, such as routing and caching rules.
- **Expose Port 80:**
  - `EXPOSE 80`
    - Indicates that the container listens on port 80 at runtime. Useful for documentation, though does not actually publish the port.
- **Start Nginx:**
  - `CMD ["nginx", "-g", "daemon off;"]`
    - Defines the default command to run Nginx in the foreground, which is required for the Docker container to stay running.

## Summary

This Dockerfile is designed for applications that need a build step (with Node.js) and are served via an Nginx server. It uses multi-stage builds to keep the final image size down and to separate concerns between building the application and serving the static files.

# nginx.conf Overview

This `nginx.conf` configuration sets up a simple web server for serving static files, ideal for single-page applications (SPAs), static sites, or as a front for a more complex application served by a different backend. It's configured to serve files over HTTP on port 80 and to handle requests to a specific domain.

## Configuration Breakdown

### Server Block

- **Listen Directive:** `listen 80;`
  - Configures Nginx to listen on port 80, the default port for HTTP traffic.
- **Server Name Directive:** `server_name domain.com;`
  - Specifies the domain name for the server. This should be replaced with your actual domain name.
- **Location Block:**
  - Defines behavior for the root URL path (`/`).
  - **Root Directive:** `root /usr/share/nginx/html;`
    - Sets the root directory for requests, pointing to where Nginx should look to serve static files.
  - **Index Directive:** `index index.html index.htm;`
    - Specifies the index files to be served when directory paths are requested. If a directory is requested, Nginx will try to serve `index.html` or `index.htm` from that directory.
  - **Try Files Directive:** `try_files $uri $uri/ /index.html;`
    - Tries to serve the requested URI as is, if it doesn't exist, tries to serve the URI as a directory, and finally falls back to `/index.html` if none of those files exist. This directive is particularly useful for SPAs where you might want all requests to be served by a single HTML file.

## Purpose and Use Cases

This configuration is typically used for hosting static websites or as the frontend for web applications where the frontend is completely separated from the backend (e.g., a React or Vue app that talks to an API). The `try_files` directive ensures that the application can use client-side routing without getting 404 errors from the server, by always falling back to `index.html` where the routing is handled by JavaScript.

---

This simple yet effective configuration is crucial for serving static content efficiently while providing a fallback mechanism for SPA routing, ensuring a smooth user experience without server errors for routes handled client-side.


## About us
### Ready to see what's beyond? Check out our website to discover more projects and learn how we're making a difference: [Discover More](https://lnoks.com).
<img src="https://api.lnoks.com/api/files/gpdni4nbyo5aj5b/ckzlbh3iiyt5n83/logo_purple_pure_hqhbjiEXbL.svg?token=" alt="drawing" width="200" height="100"/>
