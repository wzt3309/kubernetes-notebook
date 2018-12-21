NS = wzt3309
VERSION ?=
TAG_LATEST ?= 0

REPO = prometheus-exporter-demo

.PHONY: build push release package delive

build:
	docker build -t $(NS)/$(REPO):$(VERSION) .
ifeq ($(TAG_LATEST),1)
	docker tag $(NS)/$(REPO):$(VERSION) $(NS)/$(REPO):latest
endif

push:
	docker push $(NS)/$(REPO):$(VERSION)
ifeq ($(TAG_LATEST),1)
	docker push $(NS)/$(REPO):latest
endif

release: build
	make push -e VERSION=$(VERSION) TAG_LATEST=$(TAG_LATEST)

package: build
	docker save $(NS)/$(REPO):$(VERSION) > /tmp/$(NS)-$(REPO)-$(VERSION).tar
ifeq ($(TAG_LATEST),1)
	docker save $(NS)/$(REPO):latest > /tmp/$(NS)-$(REPO)-latest.tar
endif

delive: package
	scp /tmp/$(NS)-$(REPO)-$(VERSION).tar k8s-node1:/tmp
	scp /tmp/$(NS)-$(REPO)-$(VERSION).tar k8s-node2:/tmp
	ssh  k8s-node1 -t "docker load < /tmp/$(NS)-$(REPO)-$(VERSION).tar"
	ssh  k8s-node2 -t "docker load < /tmp/$(NS)-$(REPO)-$(VERSION).tar"
ifeq ($(TAG_LATEST),1)
	scp /tmp/$(NS)-$(REPO)-latest.tar k8s-node1:/tmp
	scp /tmp/$(NS)-$(REPO)-latest.tar k8s-node2:/tmp
	ssh  k8s-node1 -t "docker load < /tmp/$(NS)-$(REPO)-latest.tar"
	ssh  k8s-node2 -t "docker load < /tmp/$(NS)-$(REPO)-latest.tar"
endif

default: build

clean:
	rm -rf /tmp/$(NS)-$(REPO)-*.tar
	ssh k8s-node1 "rm -rf /tmp/$(NS)-$(REPO)-*.tar"
	ssh k8s-node2 "rm -rf /tmp/$(NS)-$(REPO)-*.tar"
ifeq ($(TAG_LATEST),1)
	docker rmi $(NS)/$(REPO):latest
	ssh k8s-node1 "docker rmi $(NS)/$(REPO):latest"
	ssh k8s-node2 "docker rmi $(NS)/$(REPO):latest"
endif
	docker rmi $(NS)/$(REPO):$(VERSION)
	@echo "Delete all untagged/dangling (<none>) images"
	-docker rmi `docker images -q -f dangling=true`
	ssh k8s-node1 "docker rmi $(NS)/$(REPO):$(VERSION)"
	ssh k8s-node2 "docker rmi $(NS)/$(REPO):$(VERSION)"
	