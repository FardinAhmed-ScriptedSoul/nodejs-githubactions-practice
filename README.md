# Node.js + GitHub Actions + Docker

A small Express app used to learn how GitHub Actions can automatically build and publish a Docker image.

## What happens in this project?

The workflow in `.github/workflows/deploy.yml` runs when code is pushed to `main`. It can also be started manually from the repository's **Actions** tab.

```text
Push to main (or run manually)
              |
              v
     GitHub Actions runner
       1. Checks out code
       2. Sets up Docker Buildx
       3. Logs in to GHCR with GITHUB_TOKEN
       4. Builds the Docker image
       5. Publishes image tags to GHCR
```

The published image is available at:

```text
ghcr.io/<owner>/<repository>:<tag>
```

The workflow creates a branch tag, a commit-SHA tag, and a `latest` tag when the branch is the repository's default branch. The workflow **publishes an image**; it does not start that image on a production server. Running the app in production would require a separate deployment step or hosting platform.

## The real-world analogy

Think of a software release like preparing and shipping a meal:

- **GitHub repository**: the recipe and ingredients.
- **Push event**: the order to prepare a new batch.
- **Workflow**: the kitchen's repeatable checklist.
- **Runner**: the kitchen where the work happens.
- **Actions and shell commands**: the tools and individual preparation steps.
- **Docker image**: the sealed, labeled meal package.
- **GHCR**: the warehouse where the package is stored.
- **Deployment**: delivering and serving the package to customers.

Automating the kitchen and putting a package in the warehouse does not, by itself, serve the meal. Likewise, a successful image-publishing workflow is not proof that the app is running in production.

## Run the app locally with Docker Compose

Docker Compose builds the image from the local `Dockerfile`, starts the app in the background, and maps host port `8080` to the app's container port `3000`.

```bash
docker compose up -d --build
curl http://localhost:8080
```

Expected response:

```json
{"msg":"Hello World"}
```

Useful commands:

```bash
docker compose logs -f app
docker compose ps
docker compose down
```

The app listens on port `3000` inside the container. `EXPOSE 3000` documents that container port; the Compose mapping (`8080:3000`) makes it reachable at `localhost:8080` on the host.

## GitHub Actions concepts

- **Workflow**: YAML automation definition stored under `.github/workflows/`.
- **Event/trigger**: something that starts a workflow, such as a push or manual dispatch.
- **Job**: a group of steps run on a runner.
- **Runner**: the machine that executes a job; this workflow uses a fresh GitHub-hosted Ubuntu runner.
- **Step**: one task in a job. A step can run a command or reuse an action.
- **Action**: reusable automation, referenced with `uses`, such as checkout or Docker Buildx setup.
- **Secret/token**: credential supplied securely to a workflow. This workflow uses the short-lived `GITHUB_TOKEN` rather than a hard-coded password.
- **Artifact/package**: an output stored for later use. Here, the Docker image is published as a GHCR package.

The workflow grants only `contents: read` and `packages: write`, which it needs to check out source code and publish the package.

## Interview questions and answers

**1. What is GitHub Actions?**

GitHub's CI/CD automation platform. Workflows describe event-triggered jobs and steps that run on GitHub-hosted or self-hosted runners.

**2. What is the difference between a workflow, job, and step?**

A workflow is the complete automation file; a job is a unit of work assigned to a runner; steps are the ordered tasks within that job.

**3. What is a runner?**

A machine that executes a job. A GitHub-hosted runner is provisioned for the job; a self-hosted runner is managed by the repository or organization.

**4. What is the difference between CI and CD?**

Continuous Integration (CI) frequently validates changes by building and testing them. Continuous Delivery/Deployment (CD) automates preparing or releasing those changes. This project's workflow builds and publishes an image; it does not deploy a running service.

**5. Why use `GITHUB_TOKEN` and workflow permissions?**

`GITHUB_TOKEN` is a workflow-provided credential scoped to the repository. Explicit permissions limit its access; `packages: write` allows this workflow to publish to GHCR without storing a personal access token in the repository.

**6. What is the difference between a Docker image and a container?**

An image is the packaged template; a container is a running instance created from that image.

**7. Why publish multiple image tags?**

A branch tag is convenient for following a branch, `latest` is a familiar default tag, and a commit-SHA tag identifies an exact build for traceability and rollback.

**8. Does a successful workflow mean the app is deployed?**

Not necessarily. Check what the workflow does: this one publishes an image to GHCR. A deployment stage must separately pull and run that image on a host or platform.

## Current limitation

The `npm test` script in `package.json` is still a placeholder that exits with an error. Add real tests and a test step before treating this workflow as a complete CI quality gate.
