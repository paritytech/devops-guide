# Building and Publishing Docker Images

Docker images make the life of node operators (and your DevOps team) easier. They allow you to get a node up and running without configuring the vm each time, and are helpful when you need to run _n_ vms in a network. External node operators (validators and RPC providers) will also thank you for providing them.

## Example Dockerfile

The following examples can be found in the Substrate Node Template repo: <https://github.com/substrate-developer-hub/substrate-node-template/blob/main/Dockerfile>

This follows the best practices to build the image in a secure way that minimises the attack surface, it is a similar version to that used to create the official Polkadot images. You can also consult [Docker's own best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/).

```dockerfile
# This is an example build stage for the node template. Here we create the binary in a temporary image.

# This is a base image to build substrate nodes
FROM docker.io/paritytech/ci-linux:production as builder

WORKDIR /node-template
COPY . .
RUN cargo build --locked --release

# This is the 2nd stage: a very small image where we copy the binary."
FROM docker.io/library/ubuntu:20.04
LABEL description="Multistage Docker image for Substrate Node Template" \
  image.type="builder" \
  image.authors="you@email.com" \
  image.vendor="Substrate Developer Hub" \
  image.description="Multistage Docker image for Substrate Node Template" \
  image.source="https://github.com/substrate-developer-hub/substrate-node-template" \
  image.documentation="https://github.com/substrate-developer-hub/substrate-node-template"

# Copy the node binary.
COPY --from=builder /node-template/target/release/node-template /usr/local/bin

RUN useradd -m -u 1000 -U -s /bin/sh -d /node-dev node-dev && \
  mkdir -p /chain-data /node-dev/.local/share && \
  chown -R node-dev:node-dev /chain-data && \
  ln -s /chain-data /node-dev/.local/share/node-template && \
  # unclutter and minimize the attack surface
  rm -rf /usr/bin /usr/sbin && \
  # check if executable works in this container
  /usr/local/bin/node-template --version

USER node-dev

EXPOSE 30333 9933 9944 9615
VOLUME ["/chain-data"]

ENTRYPOINT ["/usr/local/bin/node-template"]
```

## Automated Build Pipeline

The following examples can be found in the Substrate Node Template repo: <https://github.com/substrate-developer-hub/substrate-node-template/blob/main/.github/workflows/build-publish-image.yml>

This is an example GitHub action that will build and publish a Docker image to DockerHub. You will need to [add secrets to your GitHub Repository or Organization](https://docs.github.com/en/actions/security-guides/encrypted-secrets) to make this work and to publish images securely.

Most teams will trigger this either on a manual workflow, or when a new release is published. You will need to save the credentials for your DockerHub account in your GitHub secrets.

If you instead want to use another image repository (e.g. GitHub image registry), you can amend the Build and push Docker images step.

```yaml
# You need to add the following secrets to your GitHub Repository or Organization to make this work
# - DOCKER_USERNAME: The username of the DockerHub account. E.g. parity
# - DOCKER_TOKEN: Access token for DockerHub, see https://docs.docker.com/docker-hub/access-tokens/. E.g. VVVVVVVV-WWWW-XXXXXX-YYYY-ZZZZZZZZZ
# The following are setup as an environment variable below
# - DOCKER_REPO: The unique name of the DockerHub repository. E.g. parity/polkadot

name: Build & Publish Docker Image

# Controls when the action will run.
on:
  # Triggers the workflow on push events but only for the main branch
  # push:
  # branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Set an environment variable (that can be overriden) for the Docker Repo
env:
  DOCKER_REPO: parity/polkadot

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-22.04

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - name: Check out the repo
        uses: actions/checkout@v2.5.0

      # Login to Docker hub using the credentials stored in the repository secrets
      - name: Log in to Docker Hub
        uses: docker/login-action@v2.1.0
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      # Get the commit short hash, to use as the rev
      - name: Calculate rev hash
        id: rev
        run: echo "value=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      # Build and push 2 images, One with the version tag and the other with latest tag
      - name: Build and push Docker images
        uses: docker/build-push-action@v3.2.0
        with:
          context: .
          push: true
          tags: ${{ env.DOCKER_REPO }}:v${{ steps.rev.outputs.value }}, ${{ secrets.DOCKER_REPO }}:latest
```
