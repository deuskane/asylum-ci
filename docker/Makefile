IMAGE_NAME     := asylum-ci
CONTAINER_NAME := asylum-ci-container
TAG            := latest
DOCKERFILE     := Dockerfile
BUILD_CONTEXT  := .
MOUNT_PATH     := $(HOME)/Work/Repo/asylum-project

.PHONY: help build run shell stop clean

help:
	@echo "Asylum CI Docker helpers"
	@echo ""
	@echo "Targets:"
	@echo "  build   Build the Docker image ($(IMAGE_NAME):$(TAG))."
	@echo "  run     Run the container interactively with the project mounted at /workspace."
	@echo "  shell   Open a shell inside the running container."
	@echo "  stop    Stop the container if it is running."
	@echo "  clean   Remove the local Docker image."
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
		-v "$(MOUNT_PATH):/workspace" \
		-w /workspace \
		$(IMAGE_NAME):$(TAG) /bin/bash

stop:
	docker stop $(CONTAINER_NAME) || true

clean:
	docker rmi $(IMAGE_NAME):$(TAG) || true
