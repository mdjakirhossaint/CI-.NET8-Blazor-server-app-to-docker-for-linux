
# 🚀 .NET 8.0 Blazor Project with Dockerized CI/CD

This repository sets up a **Continuous Integration (CI)** pipeline for a `.NET 8.0 Blazor` project, integrating **Docker** for containerization and automating builds with **GitHub Actions**. Follow these steps to create, deploy, and verify your project’s Docker image on **Docker Hub**.

![GitHub last commit](https://img.shields.io/github/last-commit/username/repository)
![GitHub workflow status](https://img.shields.io/github/actions/workflow/status/username/repository/ci-build-and-push-image.yml?branch=main)

---

## 🛠️ Steps to Set Up CI Integration with Dockerization

### Step 1: [Create a GitHub Repository](https://github.com/new)
- Start by creating a new repository in your GitHub account. This will be the home for your `.NET 8.0 Blazor` project.

### Step 2: Push Your Project to GitHub
- Commit and push your `.NET 8.0 Blazor` project to this new repository.
- To push your local code:
  ```bash
  git add .
  git commit -m "Initial commit"
  git push origin main
  
### Step 3 : Create a Dockerfile
```
# Base image for running the app in Debug mode
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Stage for building the project
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["Service Libraries Path/your.csproj", "Service Libraries Path/"]
RUN dotnet restore "your presentation layer location/your.csproj"
COPY . .
WORKDIR "/src/your directory"
RUN dotnet build "your.csproj" -c $BUILD_CONFIGURATION -o /app/build

# Stage for publishing the project
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "your.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# Final stage for production
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "your.dll"]

### Step 4: Set Up GitHub Secrets 
To securely store your DockerHub credentials for GitHub Actions, follow these steps:
1. Go to your repository's **Settings**.
2. Navigate to **Secrets and variables** → **Actions**.
3. Add the following secrets:
   - **DOCKERHUB_USERNAME** – Your DockerHub username.
   - **DOCKERHUB_PASSWORD** – Your DockerHub password.
   ```
These secrets will be securely referenced in your GitHub Actions workflow.

## 🔄 Step 5: Create a GitHub Actions Workflow

To automate the process of building and pushing a Docker image, add a GitHub Actions workflow:

1. In your repository, create a new file at `.github/workflows/ci-build-and-push-image.yml`.
2. Add the following content to the file:

   ```yaml
   name: CI-build-and-push-image-for-dotnetapp

   on:
     push:
       branches: [ main ]
       paths-ignore:
         - README.md
         - .vscode/**
         - .gitignore
     pull_request:
       branches: [ main ]
       paths-ignore:
         - README.md
         - .vscode/**
         - .gitignore
     workflow_dispatch:

   jobs:
     docker:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout source code
           uses: actions/checkout@v3

         - name: Set up QEMU (for multi-platform builds)
           uses: docker/setup-qemu-action@v2

         - name: Set up Docker Buildx
           uses: docker/setup-buildx-action@v2

         - name: Log in to DockerHub
           uses: docker/login-action@v2
           with:
             username: ${{ secrets.DOCKERHUB_USERNAME }}
             password: ${{ secrets.DOCKERHUB_PASSWORD }}

         - name: Build and push Docker image
           uses: docker/build-push-action@v2
           with:
             context: .
             file: ./Dockerfile
             push: true
             tags: ${{ secrets.DOCKERHUB_USERNAME }}/your-docker-repo:latest, ${{ secrets.DOCKERHUB_USERNAME }}/your-docker-repo:${{ github.run_number }}
```
## 🐳 Step 6: Verify Your Docker Image

After the GitHub Actions workflow runs, verify that your Docker image was created successfully by visiting your Docker Hub account.

### 🧪 Testing the Docker Image Locally

To ensure the image functions as expected, pull it to your local machine and run it:

```bash
# Pull the Docker image from Docker Hub
docker pull your-docker-username/your-docker-repo:latest

# Run the Docker image, mapping port 80 in the container to port 8080 on your host machine
docker run -d -p 8080:80 your-docker-username/your-docker-repo:latest
