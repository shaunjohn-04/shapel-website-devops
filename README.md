# hosting_test

Static website automated deployment pipeline using Jenkins, Docker, SonarQube, and Kubernetes.

## Workflow
1. Jenkins pulls source code from GitHub (main branch).
2. Runs static code analysis using SonarQube Scanner.
3. Builds an Nginx Docker image containing the web assets in src/.
4. Pushes the Docker image to Docker Hub with the build number as the image tag.
5. Updates k8s/deployment.yml image tag and pushes the change back to the repository.