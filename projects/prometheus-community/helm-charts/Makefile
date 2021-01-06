REPO?=helm-charts
CLONE_URL?=https://github.com/prometheus-community/$(REPO).git

GIT_COMMIT?=$(shell cat GIT_COMMIT)

ifeq ("$(CLONE_URL)","")
	$(error No clone url was provided.)
endif

ifeq ("$(REPO)","")
	$(error No repository name was provided.)
endif

ifeq ("$(GIT_COMMIT)","")
	$(error No git commit was provided.)
endif

CHART_SCRIPTS_DIR=$(shell git rev-parse --show-toplevel)/helm-charts/scripts

.PHONY: clone
clone: clean
	git clone $(CLONE_URL) $(REPO)
	cd $(REPO) && git checkout $(GIT_COMMIT)

.PHONY: install-toolchain
install-toolchain:
	${CHART_SCRIPTS_DIR}/install-toolchain.sh

.PHONY: publish
publish:
	${CHART_ROOT}/scripts/publish-charts.sh ${REPO}/charts/prometheus

.PHONY: clean
clean:
	rm -rf $(REPO)
