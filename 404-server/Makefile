# Copyright 2016 The Kubernetes Authors All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Build the default backend binary or image for amd64, arm, arm64 and ppc64le
#
# Usage:
# 	[PREFIX=gcr.io/google_containers/defaultbackend] [ARCH=amd64] [TAG=1.1] make (server|container|push)


all: push

TAG=1.2
PREFIX?=gcr.io/google_containers/defaultbackend
ARCH?=amd64
GOLANG_VERSION=1.6
TEMP_DIR:=$(shell mktemp -d)

# Set default base image dynamically for each arch
ifeq ($(ARCH),amd64)
	BASEIMAGE?=busybox
endif
ifeq ($(ARCH),arm)
	BASEIMAGE?=armel/busybox
endif
ifeq ($(ARCH),arm64)
	BASEIMAGE?=aarch64/busybox
endif
ifeq ($(ARCH),ppc64le)
	BASEIMAGE?=ppc64le/busybox
endif

server: server.go
	CGO_ENABLED=0 GOOS=linux GOARCH=$(ARCH) GOARM=6 go build -a -installsuffix cgo -ldflags '-w' -o server ./server.go

container:
	# Copy the whole directory to a temporary dir and set the base image
	cp -r ./* $(TEMP_DIR)
	cd $(TEMP_DIR) && sed -i "s|BASEIMAGE|$(BASEIMAGE)|g" Dockerfile

	# Compile the binary inside a container for reliable builds
	docker run --rm -it -v $(TEMP_DIR):/build golang:$(GOLANG_VERSION) /bin/bash -c "make -C /build server ARCH=$(ARCH)"

	docker build --pull -t $(PREFIX)-$(ARCH):$(TAG) $(TEMP_DIR)

	rm -rf $(TEMP_DIR)

push: container
	gcloud docker push $(PREFIX)-$(ARCH):$(TAG)

push-legacy: container
ifeq ($(ARCH),amd64)
	# Backward compatibility. TODO: deprecate this image tag
	docker tag -f $(PREFIX)-$(ARCH):$(TAG) $(PREFIX):$(TAG)
	gcloud docker push $(PREFIX):$(TAG)
endif

clean:
	rm -f server
