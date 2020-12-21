REPO?=prometheus
CLONE_URL?=https://github.com/prometheus/$(REPO).git
GIT_TAG?=$(shell cat GIT_TAG)

ifeq ("$(CLONE_URL)","")
	$(error No clone url was provided.)
endif

ifeq ("$(REPO)","")
	$(error No repository name was provided.)
endif

ifeq ("$(GIT_TAG)","")
	$(error No git tag was provided.)
endif

AWS_ACCOUNT_ID?=$(shell aws sts get-caller-identity --query Account --output text)
AWS_REGION?=us-west-2

IMAGE_REPO?=$(AWS_ACCOUNT_ID).dkr.ecr.$(AWS_REGION).amazonaws.com
IMAGE_NAME?=prometheus/prometheus
# This tag is overwritten in the prow job to point to the commit hash
IMAGE_TAG?=latest
IMAGE?=$(IMAGE_REPO)/$(IMAGE_NAME):$(IMAGE_TAG)

.PHONY: clone
clone: clean
	git clone $(CLONE_URL) $(REPO)
	cd $(REPO) && git checkout tags/$(GIT_TAG)

.PHONY: local-images
local-images: clone npm_licenses
	# Sleep to wait for buildkitd to start up
	sleep 10
	buildctl \
		build \
		--frontend dockerfile.v0 \
		--opt platform=linux/amd64 \
		--local dockerfile=./$(REPO) \
		--local context=./$(REPO) \
		--output type=oci,oci-mediatypes=true,name=$(IMAGE),dest=/tmp/prometheus.tar

.PHONY: images
images: clone npm_licenses
	# Sleep to wait for buildkitd to start up
	sleep 10
	buildctl \
		build \
		--frontend dockerfile.v0 \
		--opt platform=linux/amd64 \
		--local dockerfile=./$(REPO) \
		--local context=./$(REPO) \
		--output type=image,oci-mediatypes=true,name=$(IMAGE),push=true

.PHONY: npm_licenses
npm_licenses:
	make -C ./$(REPO) npm_licenses

.PHONY: docker
docker: clone
	docker build -t $(IMAGE) -f ./$(REPO)/Dockerfile ./$(REPO)

.PHONY: build-dependencies
build-dependencies:
	curl -sL https://dl.yarnpkg.com/rpm/yarn.repo -o /etc/yum.repos.d/yarn.repo
	curl -sL https://rpm.nodesource.com/setup_14.x | bash -
	yum install nodejs yarn bzip2 -y

.PHONY: build
build: build-dependencies local-images

.PHONY: release
release: build-dependencies images

.PHONY: clean
clean:
	rm -rf $(REPO)