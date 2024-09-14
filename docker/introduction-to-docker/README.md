## Introduction to Docker

### Overview
Docker is an open platform for developing, shipping, and running applications. With Docker, you can separate your applications from your infrastructure and treat your infrastructure like a managed application. Docker helps you ship code faster, test faster, deploy faster, and shorten the cycle between writing code and running code.

In this lab, you will learn the essentials of Docker, building the skillset needed to start developing Kubernetes and containerized applications.

### Objectives
- Build, run, and debug Docker containers.
- Pull Docker images from Docker Hub and Google Artifact Registry.
- Push Docker images to Google Artifact Registry.

### Prerequisites
This is an introductory-level lab. Familiarity with Cloud Shell and the command line is suggested but not required.

### Setup and Requirements
- Access to a standard internet browser (Chrome recommended).
- Use an Incognito or private browser window to run this lab.
- Do not use your personal Google Cloud account to avoid incurring charges.

### Lab Instructions

#### Task 1: Hello World
1. **Run a Hello World Container**:
   ```bash
   docker run hello-world
   ```
   Output will display the steps Docker performed to run the container.

2. **List Docker Images**:
   ```bash
   docker images
   ```

3. **Run the Container Again**:
   ```bash
   docker run hello-world
   ```

4. **View Running Containers**:
   ```bash
   docker ps
   ```

5. **View All Containers** (including exited ones):
   ```bash
   docker ps -a
   ```

#### Task 2: Build a Docker Image
1. **Create and Switch into a Folder**:
   ```bash
   mkdir test && cd test
   ```

2. **Create a Dockerfile**:
   ```bash
   cat > Dockerfile <<EOF
   # Use an official Node runtime as the parent image
   FROM node:lts

   # Set the working directory in the container to /app
   WORKDIR /app

   # Copy the current directory contents into the container at /app
   ADD . /app

   # Make the container's port 80 available to the outside world
   EXPOSE 80

   # Run app.js using node when the container launches
   CMD ["node", "app.js"]
   EOF
   ```

3. **Create a Node Application**:
   ```bash
   cat > app.js << EOF
   const http = require("http");

   const hostname = "0.0.0.0";
   const port = 80;

   const server = http.createServer((req, res) => {
       res.statusCode = 200;
       res.setHeader("Content-Type", "text/plain");
       res.end("Hello World\n");
   });

   server.listen(port, hostname, () => {
       console.log("Server running at http://%s:%s/", hostname, port);
   });

   process.on("SIGINT", function () {
       console.log("Caught interrupt signal and will exit");
       process.exit();
   });
   EOF
   ```

4. **Build the Docker Image**:
   ```bash
   docker build -t node-app:0.1 .
   ```

5. **List Docker Images**:
   ```bash
   docker images
   ```

#### Task 3: Run the Docker Image
1. **Run the Container**:
   ```bash
   docker run -p 4000:80 --name my-app node-app:0.1
   ```

2. **Test the Server**:
   ```bash
   curl http://localhost:4000
   ```

3. **Stop and Remove the Container**:
   ```bash
   docker stop my-app && docker rm my-app
   ```

4. **Run the Container in the Background**:
   ```bash
   docker run -p 4000:80 --name my-app -d node-app:0.1
   ```

5. **List Running Containers**:
   ```bash
   docker ps
   ```

#### Task 4: Modify and Rebuild the Application
1. **Edit `app.js`** to change "Hello World" to "Welcome to Cloud".

2. **Build the New Image**:
   ```bash
   docker build -t node-app:0.2 .
   ```

3. **Run the New Container**:
   ```bash
   docker run -p 8080:80 --name my-app-2 -d node-app:0.2
   ```

4. **Test the Containers**:
   ```bash
   curl http://localhost:8080
   curl http://localhost:4000
   ```

#### Task 5: Debug
1. **View Logs of a Container**:
   ```bash
   docker logs [container_id]
   ```

2. **Start an Interactive Bash Session in the Running Container**:
   ```bash
   docker exec -it [container_id] bash
   ```

3. **Exit the Bash Session**:
   ```bash
   exit
   ```

4. **Inspect the Container Metadata**:
   ```bash
   docker inspect [container_id]
   ```

#### Task 6: Publish to Google Artifact Registry
1. **Create an Artifact Registry Repository**:
   ```bash
   gcloud artifacts repositories create my-repository --repository-format=docker --location=us-west1 --description="Docker repository"
   ```

2. **Configure Docker to Authenticate with Artifact Registry**:
   ```bash
   gcloud auth configure-docker us-west1-docker.pkg.dev
   ```

3. **Tag the Docker Image**:
   ```bash
   docker build -t us-west1-docker.pkg.dev/[PROJECT_ID]/my-repository/node-app:0.2 .
   ```

4. **Push the Image to Artifact Registry**:
   ```bash
   docker push us-west1-docker.pkg.dev/[PROJECT_ID]/my-repository/node-app:0.2
   ```

5. **Simulate a Fresh Environment** by removing containers and images:
   ```bash
   docker stop $(docker ps -q)
   docker rm $(docker ps -aq)
   docker rmi us-west1-docker.pkg.dev/[PROJECT_ID]/my-repository/node-app:0.2
   docker rmi node:lts
   docker rmi -f $(docker images -aq)
   ```

6. **Pull the Image and Run It**:
   ```bash
   docker run -p 4000:80 -d us-west1-docker.pkg.dev/[PROJECT_ID]/my-repository/node-app:0.2
   ```

7. **Test the Container**:
   ```bash
   curl http://localhost:4000
   ```

### Conclusion
In this lab, you learned how to:
- Run containers based on public images from Docker Hub.
- Build your own container images.
- Push images to Google Artifact Registry.
- Debug running containers.
