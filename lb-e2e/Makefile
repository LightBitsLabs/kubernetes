
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

.PHONY: kubernetes-test-linux-amd64.tar.gz
kubernetes-test-linux-amd64.tar.gz:
	rm -rf /tmp/kubernetes/test/bin
	mkdir -p /tmp/kubernetes/test/bin
	cp ./_output/dockerized/bin/linux/amd64/e2e.test \
		./_output/local/bin/linux/amd64/ginkgo \
		/tmp/kubernetes/test/bin/
	tar -czvf kubernetes-test-linux-amd64.tar.gz \
		-C /tmp/ \
		kubernetes
	rm -rf /tmp/kubernetes/test/bin

upload-release: FILE=kubernetes-test-linux-amd64.tar.gz
upload-release:
	curl -v -i -X POST \
		-H "Accept: application/vnd.github.v3+json" \
		https://api.github.com/repos/lightbitslabs/kubernetes/releases \
		-d '{"name": "$(CLUSTER_VERSION)-lightbits","body": "Patched e2e binaries with fixes", "draft": false, "prerelease": false}'; echo
	curl -X POST \
		-H "Content-Length: $(shell stat --printf=%s $(FILE))" \
		-H "Content-Type: $(shell file -b --mime-type $(FILE))" \
		-T "$(FILE)" \
		-H "Accept: application/vnd.github.v3+json" \
		https://uploads.github.com/repos/lightbitslabs/kubernetes/releases/$(GIT_BRANCH)/assets?name=$(FILE) ;echo

clean:
	rm -rf kubernetes-test-linux-amd64.tar.gz
	rm -rf ./_output/dockerized/bin/linux/amd64/e2e.test ./_output/local/bin/linux/amd64/ginkgo rm -rf /tmp/kubernetes/test/bin
