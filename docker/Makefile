IMAGE_NAME        := asylum-ci
CONTAINER_NAME    := asylum-ci-container
TAG               := latest
DOCKERFILE        := Dockerfile
BUILD_CONTEXT     := .
DOCKER_MOUNT_PATH ?= $(PWD)

.PHONY: help build run shell stop clean purge

help:
	@echo "Asylum CI Docker helpers"
	@echo ""
	@echo "Targets:"
	@echo "  build   Build the Docker image ($(IMAGE_NAME):$(TAG))."
	@echo "  run     Run the container interactively with the project mounted at /workspace."
	@echo "  shell   Open a shell inside the running container."
	@echo "  stop    Stop the container if it is running."
	@echo "  clean   Remove the local Docker image."
	@echo "  purge   Remove Docker build cache and unused Docker data."
	@echo ""
	@echo "Example:"
	@echo "  make build"
	@echo "  make run"

build:
	docker build -t $(IMAGE_NAME):$(TAG) -f $(DOCKERFILE) $(BUILD_CONTEXT)

run:
	docker rm -f $(CONTAINER_NAME) >/dev/null 2>&1 || true
	docker run --rm -it \
		--name $(CONTAINER_NAME) \
		-v "$(PWD):/workspace" \
		-w /workspace \
		$(IMAGE_NAME):$(TAG)

shell:
	docker rm -f $(CONTAINER_NAME) >/dev/null 2>&1 || true
	docker run --rm -it \
		--name $(CONTAINER_NAME) \
		-v "$(DOCKER_MOUNT_PATH):/workspace" \
		-w /workspace \
		$(IMAGE_NAME):$(TAG) /bin/bash

stop:
	docker stop $(CONTAINER_NAME) || true

clean:
	docker rmi $(IMAGE_NAME):$(TAG) || true

purge:
	@echo "Purging Docker build cache and unused data..."
	docker builder prune -af
	docker system prune -af --volumes
