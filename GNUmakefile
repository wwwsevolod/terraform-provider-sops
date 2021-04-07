export CGO_ENABLED = 0
VERSION = $(shell git describe --tags --match='v*' --always)
RELEASE = $(patsubst v%,%,$(VERSION))# Remove leading v to comply with Terraform Registry conventions

CROSSBUILD_OS   = linux windows darwin
CROSSBUILD_ARCH = 386 amd64
SKIP_OSARCH     = darwin_386
OSARCH_COMBOS   = $(filter-out $(SKIP_OSARCH),$(foreach os,$(CROSSBUILD_OS),$(addprefix $(os)_,$(CROSSBUILD_ARCH))))

default: build

style:
	@echo ">> checking code style"
	! gofmt -d $(shell find . -path ./vendor -prune -o -name '*.go' -print) | grep '^'

vet:
	@echo ">> vetting code"
	go vet ./...

test:
	@echo ">> testing code"
	go test -v ./...

build:
	@echo ">> building binaries"
	go build -mod=vendor -o terraform-provider-sops

crossbuild: $(GOPATH)/bin/gox
	@echo ">> cross-building"
	gox -arch="$(CROSSBUILD_ARCH)" -os="$(CROSSBUILD_OS)" -osarch="$(addprefix !,$(subst _,/,$(SKIP_OSARCH)))" \
		-mod=vendor -output="binaries/$(VERSION)/{{.OS}}_{{.Arch}}/terraform-provider-sops_$(VERSION)"

$(GOPATH)/bin/gox:
	# Need to disable modules for this to not pollute go.mod
	@GO111MODULE=off go get -u github.com/mitchellh/gox

release: crossbuild
	@echo ">> uploading release $(VERSION)"
	mkdir -p releases
	set -e; for OSARCH in $(OSARCH_COMBOS); do \
		zip -j releases/terraform-provider-sops_$(RELEASE)_$$OSARCH.zip binaries/$(VERSION)/$$OSARCH/terraform-provider-sops_* > /dev/null; \
		hub release edit -m "" -a "releases/terraform-provider-sops_$(RELEASE)_$$OSARCH.zip#terraform-provider-sops_$(RELEASE)_$$OSARCH.zip" $(VERSION); \
	done
	@echo ">>> generating sha256sums:"
	cd releases; shasum -a 256 *.zip | tee terraform-provider-sops_$(RELEASE)_SHA256SUMS
	hub release edit -m "" -a "releases/terraform-provider-sops_$(RELEASE)_SHA256SUMS#terraform-provider-sops_$(RELEASE)_SHA256SUMS" $(VERSION)

.PHONY: all style vet test build crossbuild release
