# Assignment: Local CI/CD Pipeline with GitHub Actions & Docker Desktop

## Objective
In this assignment, you will build an automated CI/CD pipeline that builds a Docker image of a web application and pushes it to a local container registry. 

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
Once you have successfully pulled your image back from the local registry, take a screenshot of your VS Code terminal showing the successful `act` execution and your verification step. Submit this along with your `build-and-push.yml` file.
