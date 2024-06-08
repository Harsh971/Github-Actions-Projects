To automate the process of creating a Docker image and pushing it to Docker Hub every time a change is committed to your GitHub repository, you can set up a GitHub Actions workflow. Here's a step-by-step guide to accomplish this:

### Step 1: Create a Dockerfile

Ensure you have a `Dockerfile` in the root of your GitHub repository. This `Dockerfile` will define how your Docker image is built. Here's an example `Dockerfile`:

```Dockerfile
# Use the official Node.js image as a base
FROM node:14

# Create and change to the app directory
WORKDIR /app

# Copy application files
COPY . .

# Install dependencies
RUN npm install

# Start the application
CMD ["npm", "start"]

# Expose port 3000
EXPOSE 3000
```

### Step 2: Set Up Docker Hub Credentials in GitHub Secrets

1. Go to your Docker Hub account and ensure you have a repository set up to receive the Docker images.
2. Create a Docker Hub access token (or use your password).
3. In your GitHub repository, go to `Settings` > `Security` > `Secrets and variables` > `Actions`.
4. Add the following secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username.
   - `DOCKER_PASSWORD`: Your Docker Hub password or access token.

### Step 3: Create a GitHub Actions Workflow

Create a `.github/workflows/docker-build-and-push.yml` file in your repository with the following content:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main  # Adjust this if you want to trigger on other branches

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

    - name: Extract version from package.json
      id: vars
      run: echo "VERSION=$(jq -r .version package.json)" >> $GITHUB_ENV

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/my-repo:${{ env.VERSION }}
        tags: ${{ secrets.DOCKER_USERNAME }}/my-repo:latest
```

### Explanation of the Workflow

1. **Trigger on Push:** The workflow triggers every time there is a push to the `main` branch.
2. **Checkout Repository:** The `actions/checkout` action checks out your repository.
3. **Set up Docker Buildx:** The `docker/setup-buildx-action` sets up Docker Buildx, a feature used to build images.
4. **Log in to Docker Hub:** The `docker/login-action` logs in to Docker Hub using the credentials stored in your repository’s secrets.
5. **Extract Version:** This step extracts the version number from a `package.json` file and stores it in a GitHub Actions environment variable (`VERSION`). Adjust this step if you store the version elsewhere.
6. **Build and Push Docker Image:** The `docker/build-push-action` builds and pushes the Docker image to Docker Hub. It tags the image with the version number and `latest`.

### Step 4: Commit and Push Changes

Commit and push your workflow file to GitHub:

```sh
git add .github/workflows/docker-build-and-push.yml
git commit -m "Add GitHub Actions workflow to build and push Docker image"
git push origin main
```

### Step 5: Verify the Workflow

1. After pushing the changes, the workflow will run automatically.
2. Check the `Actions` tab in your GitHub repository to monitor the progress and ensure the workflow completes successfully.
3. Verify that the Docker image is available in your Docker Hub repository with the correct tags.

You have automated the process of building a Docker image and pushing it to Docker Hub every time there is a change committed to your GitHub repository. This setup ensures that your Docker Hub repository is always up-to-date with the latest version of your application.
