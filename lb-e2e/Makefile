
export IMAGE_REGISTRY ?= lbdocker:5000
export GIT_BRANCH ?= $(shell git rev-parse --abbrev-ref HEAD)
export VERSION ?= $(GIT_BRANCH)
export REPOSITORY ?= $(IMAGE_REGISTRY)/e2e-binaries
export IMG ?= $(REPOSITORY):$(VERSION)

override GIT_VER := $(or \
	    $(shell git describe --tags --abbrev=8 --always --long --dirty),UNKNOWN)

build-deps:
	./build/run.sh ${MAKE} WHAT=test/e2e/e2e.test
	${MAKE} ginkgo

image-build:
	docker build \
		-f lb-e2e/Dockerfile.e2e \
		--build-arg GIT_VER=$(GIT_VER) \
		--build-arg GIT_BRANCH=$(GIT_BRANCH) \
		-t $(IMG) .

image-push:
	docker push $(IMG)


