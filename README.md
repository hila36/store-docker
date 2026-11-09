
# Assignment 1: Local CI/CD Pipeline with GitHub Actions & Docker Desktop
## Objective
In this assignment, you will build an automated CI/CD pipeline that builds Docker images for **each service** of a web application (e.g., frontend and backend) and pushes them to a local container registry. 

To simulate a real DevOps environment without relying on cloud infrastructure, you will execute this GitHub Actions workflow entirely locally on your machine using Docker Desktop and the `act` CLI tool. These images will be used in your next assignment when we cover Docker Compose.

To simulate a real DevOps environment without relying on cloud infrastructure, you will execute this GitHub Actions workflow entirely locally on your machine using Docker Desktop and the `act` CLI tool. This image will be used in your next assignment when we cover Docker Compose.

## Prerequisites
Before starting, ensure your local development environment has the following installed and running:
* **Docker Desktop** (Make sure the Docker engine is running)
* **VS Code** 
* **`act` CLI** (A tool for running GitHub Actions locally)
* A basic application codebase that includes a valid `Dockerfile`

---

## Stage 1: Provision the Local Registry
Before you can push an image, you need a place to store it. Your first task is to host your own container registry locally.

* **Your Task:** Using Docker Desktop, research how to run the official `registry:2` Docker image as a background container. 
* **Requirement:** Expose and map it to port `5000` on your host machine.
* **Goal:** You should have a running container acting as your private, local image repository.

## Stage 2: Design the GitHub Actions Workflow
Now, you will write the automation script that builds and pushes your application.

* **Your Task:** Inside your project repository, create the standard directory structure required by GitHub Actions and create a workflow YAML file (e.g., `build-and-push.yml`).
* **Requirements:** 
  1. Set the workflow to trigger on a `push` event into the 
  `master` branch.
  2. Define a job that runs on an Ubuntu runner.
  3. **Step 1:** Use the standard GitHub Action to check out your repository's code.
  4. **Step 2:** Write a step that executes the `docker build` command. You must tag the image specifically so that Docker knows it belongs to your local registry (Hint: you will need to prefix the tag with `localhost:5000/my-app`).
  5. **Step 3:** Write a step that executes the `docker push` command to send your newly built image to your local registry.

## Stage 3: Execute the Workflow Locally
With your workflow file written and your local registry running, it is time to test the automation.

* **Your Task:** Do not commit and push your code to GitHub. Instead, open your VS Code terminal and use the `act` CLI to trigger your workflow locally.
* **Goal:** Watch the terminal output as `act` provisions a local runner, checks out your code, builds the Docker image, and pushes it to the registry running in Docker Desktop. Troubleshoot any YAML syntax or Docker CLI errors that arise during execution.

## Stage 4: Verification
An automation is only successful if you can prove the outcome. 

* **Your Task:** Verify that your image actually made it into the local registry.
* **Validation Method 1:** Research how to query the local registry's HTTP API (specifically the `/_catalog` endpoint) using `curl` or your browser to see a list of stored repositories.
* **Validation Method 2:** Delete the image from your local Docker Desktop cache, and attempt to `docker pull` the image directly from your `localhost:5000` registry. 

---

### Deliverables
Once you have successfully pulled your image back from the local registry, take a screenshot of your VS Code terminal showing the successful `act` execution and your verification step. Submit this along with your `build-and-push.yml` file.---

# Assignment 2 - Deploying the "STORE" Application with Docker Compose

## Objective
In Assignment 1, you successfully built Docker images for the services of your "STORE" application and pushed them to your local registry (`localhost:5000`). 

In this assignment, you will write a `docker-compose.yml` file to spin up your entire application architecture at once, including the database.

## Stage 1: The Docker Compose File
1. In the root of your "STORE" project repository, create a new file named `docker-compose.yml`.
2. Define the necessary services for your architecture. This should include:
   - **Your App Services:** For each of your custom services (e.g., client, server, etc.), **do not use the `build` context.** Instead, tell Docker Compose to pull the images you created in Assignment 1 from your local registry (e.g., `image: localhost:5000/store-<service_name>:v1`).
   - **Database Service:** Pull the official MongoDB image from Docker Hub (`image: mongo:latest`).

## Stage 2: Configuration & Environment
To make these independent containers talk to each other, they need the right configuration.
1. **Environment Variables:** Pass the correct environment variables to your services so they know how to connect to the database container. (Hint: The connection string inside the Docker network will use the service name, e.g., `mongodb://<db_service_name>:27017/store-db`).
2. **Ports:** Expose the correct ports for your services so you can access them from your host machine browser. 
3. **Dependencies:** Add `depends_on` to your services that rely on the database so they wait for it to start up first.

## Stage 3: Run the App
1. Run `docker-compose up -d` in your terminal to start the application.
2. Verify that you can access your application and that the services successfully communicate with each other and the database.
3. ⚠️ *Make sure to run `docker-compose down` to clean up your containers when you are done with this stage.*

---

# 🎯 Assignment 3 (Bonus) - Single Entrypoint with Nginx Reverse Proxy

## Objective
In the previous assignment, we used docker-compose to deploy our application. We exposed the different services on different ports. 

What if we wanted to expose our application on a **single port**? For example, we want our users to simply visit `http://localhost` (port 80) to see the user interface, and we want any API requests sent to `http://localhost/api` to be automatically routed to the server behind the scenes.

In this assignment, we will set up an Nginx Reverse Proxy to do exactly that.

## Stage 1: Nginx Configuration
1. In your project directory, create a new folder called `nginx`.
2. Inside the `nginx` folder, create a new file called `default.conf`.
3. In the `default.conf` file, write the following configuration (Replace the placeholders `<SERVICE_NAME>` and `<PORT>` with the actual names and internal ports from your `docker-compose.yml`):

```nginx
server {
    listen 80;

    # Route /api requests to the API/server service
    location /api {
        proxy_pass http://<API_SERVICE_NAME>:<API_SERVICE_PORT>/api;
    }

    # Route all other requests to the client/UI service
    location / {
        proxy_pass http://<CLIENT_SERVICE_NAME>:<CLIENT_SERVICE_PORT>;
    }
}
```

## Stage 2: Update Docker Compose
Now, you need to add Nginx to your architecture and lock down the other services.

1. Open your `docker-compose.yml` file and add a new service called `nginx`, based on the `nginx:latest` image from Docker Hub.
2. **Security:** Remove the `ports` mapping from your custom app services. They should no longer be accessible directly from your host machine. They will only communicate internally on the Docker network.
3. **Expose Nginx:** Expose port `80` on the `nginx` service (e.g., `80:80`).
4. **Volume Mount:** Mount the configuration file you created into the Nginx container. Add this to your `nginx` service:
   ```yaml
   volumes:
     - ./nginx:/etc/nginx/conf.d
   ```
5. **Dependencies:** Add `depends_on` to the `nginx` service so it starts only after your app services are ready.

## Stage 3: Test the Single Entrypoint
1. Run `docker-compose up -d` to start the updated architecture.
2. Go to `http://localhost` in your browser. You should see your application interface.
3. Perform an action that calls the API (which now goes through `http://localhost/api`). Ensure everything works seamlessly through the single port!

ℹ️ You need to make sure all your backend routes start with `/api` or edit the `default.conf` file to match your routes.
