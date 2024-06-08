

### Step 1: Create a Dockerfile

First, create a `Dockerfile` in your repository's root directory. This `Dockerfile` will specify how to build the Docker image for your web page.

```Dockerfile
# Use the official NGINX image from the Docker Hub
FROM nginx:latest

# Copy the index.html file to the default NGINX directory
COPY index.html /usr/share/nginx/html/

# Expose port 80 to the outside world
EXPOSE 80
```

### Step 2: Create a GitHub Actions Workflow

Next, create a GitHub Actions workflow file in your repository. This file will define the steps required to build and push your Docker image.

Create a `.github/workflows/dockerize.yml` file with the following content:

```yaml
name: Dockerize and Push

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/simple-webpage:latest
```

### Step 3: Configure Docker Hub Credentials

For the GitHub Actions workflow to push the Docker image to Docker Hub, you need to configure your Docker Hub credentials in your GitHub repository as secrets.

1. Go to your repository on GitHub.
2. Click on `Settings`.
3. In the `Security` section, click on `Secrets and variables` > `Actions`.
4. Click the `New repository secret` button.
5. Add the following secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username.
   - `DOCKER_PASSWORD`: Your Docker Hub password.

### Step 4: Push Changes to GitHub

Finally, commit and push your changes to GitHub. This will trigger the GitHub Actions workflow, which will build the Docker image and push it to Docker Hub.

```sh
git add .
git commit -m "Dockerize web page with GitHub Actions"
git push origin main
```

### Step 5: Verify the Docker Image

After the workflow runs successfully, your Docker image should be available in your Docker Hub repository. You can verify this by going to your Docker Hub account and checking for the new image.

### Running the Docker Container Locally

To run the Docker container locally, use the following command:

```sh
docker run -d -p 80:80 your_dockerhub_username/simple-webpage:latest
```

Replace `your_dockerhub_username` with your actual Docker Hub username. This command will run the Docker container and map port 80 of the container to port 80 of your localhost, making your web page accessible via `http://localhost`.

