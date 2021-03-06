CONTRAIL_VNC ?= https://github.com/kklimonda/contrail-vnc.git
BRANCH ?= master
SERIES ?= xenial
DISTRO ?= ubuntu
VERSION ?= 5.0
JOBS ?= 25
BUILD ?= debug
IMAGE ?= contrail-builder:xenial

bin:
	mkdir bin

bin/repo: bin
	curl https://storage.googleapis.com/git-repo-downloads/repo > bin/repo
	chmod +x bin/repo

src:
	mkdir src

src/.repo/: bin/repo src
	(cd src && ../bin/repo init -u ${CONTRAIL_VNC} -b ${BRANCH})

checkout: bin/repo src/.repo/
	(cd src && ../bin/repo sync)
	cp -R src/tools/packages/debian/contrail/debian/ src/debian/
	touch checkout

patch: checkout
	bash patch_contrail.sh
	touch patch

build: patch third_party
	docker run -u 1000:1000 -it -v `pwd`/src/:/src -v /lib/modules:/lib/modules -v /usr/src:/usr/src -w /src/ contrail-builder:${VERSION}-${SERIES} scons -j$(JOBS) --opt=$(BUILD) --root=/src/build/debian/tmp/

install:
	docker run -u 1000:1000 -it -v `pwd`/src/:/src -v /lib/modules:/lib/modules -v /usr/src:/usr/src -w /src/ contrail-builder:${VERSION}-${SERIES} make -f debian/rules install

third_party: checkout
	docker run -u 1000:1000 -e USER=${USER} -it -v `pwd`/src/:/src -w /src/third_party/ contrail-builder:${VERSION}-${SERIES} python fetch_packages.py
	touch third_party

docker:
	cat docker/Dockerfile.${SERIES} | docker build -t contrail-builder:${VERSION}-${SERIES} -

package:
	docker run -u 1000:1000 -e USER=${USER} -it -v `pwd`/scripts/:/scripts -v `pwd`/target:/target -w /target ${IMAGE} bash -x /scripts/package.sh ${DISTRO} ${SERIES} ${CONTRAIL_VNC} ${BRANCH} ${VERSION} ${JOBS}

.PHONY: build docker package
