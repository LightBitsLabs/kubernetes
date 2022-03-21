# Kubernetes E2E Tests

- [Kubernetes E2E Tests](#kubernetes-e2e-tests)
  - [Overview](#overview)
  - [Assumptions](#assumptions)
    - [Add a fix to the e2e test suite](#add-a-fix-to-the-e2e-test-suite)
    - [Generate `ginkgo` binary](#generate-ginkgo-binary)
      - [From Source (preferred)](#from-source-preferred)
      - [From Build Binaries](#from-build-binaries)
    - [Build `e2e.test` Binary From Source](#build-e2etest-binary-from-source)
    - [Generate `e2e-binaries` Image](#generate-e2e-binaries-image)


We maintain our own fork of kubernetes source code to fix some issues with the e2e test suite, at:

https://github.com/LightBitsLabs/kubernetes

## Overview

Sometimes we see issues in tests that fail our CI and we need to respond quickly.

There are several options to handle this:

1. Open a ticket and wait for fix. (may take too long)
2. Create a fix and contribute to upstream. (best option)
3. skip the test for this version (bad idea)

This document will describe how we will handle the second option:

Open a ticket with the community. If it is a real issue and they fix it, this is the best.
If they acknowledge the issue but will fix it later or will not port it to older versions we need to port it our selves.

## Assumptions

1. The kubernetes repository is using semver tags to generate releases.
2. Each k8s version (e.g. `CLUSTER_VERSION`) will have a tag.
3. We will apply fixes only on formal tags.
4. Each k8s release will have a single branch named: `${tag_name}-branch` which will contain all patches we need for this version.
    for example for tag: `v1.22.15` we will have `v1.22.15-lightbits` branch.
### Add a fix to the e2e test suite

We will first clone the repository.

```bash
export CLUSTER_VERSION=v1.22.15
git clone https://github.com/LightBitsLabs/kubernetes
```

Checkout the specific tag we want into a new branch named `${CLUSTER_VERSION}-branch`

```bash
git checkout -b ${CLUSTER_VERSION}-branch ${CLUSTER_VERSION}

Apply the patches... and push it to the origin.

git push origin ${CLUSTER_VERSION}-branch
```

### Generate `ginkgo` binary

There are two ways to generate the ginkgo executable

#### From Source (preferred)

build `ginkgo` binary:

```bash
make ginkgo
```

Outcome will be placed at: `./_output/local/bin/linux/amd64/ginkgo`

#### From Build Binaries

Extract the correct ginkgo binary from the release:

```bash
test_archive_url="https://dl.k8s.io/$CLUSTER_VERSION/kubernetes-test-linux-amd64.tar.gz"
curl -L ${test_archive_url} | tar --strip-components=3 -zxf - kubernetes/test/bin/ginkgo
```

### Build `e2e.test` Binary From Source

build `e2e.test` binary we can issue the following command:

```bash
./build/run.sh make WHAT=test/e2e/e2e.test
```

Outcome will be placed at: `./_output/dockerized/bin/linux/amd64/e2e.test`

### Generate kubernetes-test-linux-amd64.tar.gz tarball

Now that we have the 2 binaries under `./_output/local/bin/linux/amd64/ginkgo` and `./_output/dockerized/bin/linux/amd64/e2e.test`
we need to pack them into a tarball to upload to the release

```bash
mkdir -p /tmp/e2e/kubernetes/test/bin
cp ./_output/dockerized/bin/linux/amd64/e2e.test ./_output/local/bin/linux/amd64/ginkgo /tmp/e2e/kubernetes/test/bin
pushd /tmp/e2e/
tar -czvf kubernetes-test-linux-amd64.tar.gz kubernetes/
popd
```

Now we would need to upload the outcome `kubernetes-test-linux-amd64.tar.gz` to the release through the UI by pressing edit and upload

### Generate `e2e-binaries` using provided Makefile

to automate the ginkgo and e2e binaries generation issue the following command

```bash
make -f lb-e2e/Makefile build-deps kubernetes-test-linux-amd64.tar.gz
```

### Create a tag and push it to github

```bash
git tag -a ${CLUSTER_VERSION}-lightbits -m "lightbits release for e2e ${CLUSTER_VERSION} tests"
git push origin ${CLUSTER_VERSION}-lightbits
```

### Through the UI create a Release

Go to [github](https://github.com/LightBitsLabs/kubernetes/releases/new) on that repo and create a release and upload the genearated file `kubernetes-test-linux-amd64.tar.gz` using this release.

