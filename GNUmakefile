DEBNAME := prometheus-gobgp-exporter
APP_REMOTE := github.com/greenpau/gobgp_exporter
# renovate: datasource=github-releases depName=greenpau/gobgp_exporter
EXPORTER_VERSION := v1.2.2
APPDESCRIPTION := Prometheus Exporter for GoBGP
APPURL := https://github.com/greenpau/gobgp_exporter
ARCH := amd64
DEB_ARCH := amd64
GO_BUILD_SOURCE := .

# Setup
BUILD_NUMBER ?= 0
DEBVERSION := $(EXPORTER_VERSION:v%=%)-$(BUILD_NUMBER)
GOPATH := $(abspath gopath)
APPHOME := $(GOPATH)/src/$(APP_REMOTE)

# Let's map from go architectures to deb architectures, because they're not the same!
GO_amd64_ARCH := amd64

# Version info for binaries
.EXPORT_ALL_VARIABLES:

.PHONY: package
package: $(addsuffix .deb, $(addprefix $(DEBNAME)_$(DEBVERSION)_, $(foreach a, $(DEB_ARCH), $(a))))

.PHONY: checkout
checkout: $(APPHOME)

.PHONE: parp
parp: $(APPHOME)
$(GOPATH):
	mkdir $(GOPATH)

.PHONE: build
build: $(APPHOME)/bin/gobgp-exporter
$(APPHOME)/bin/gobgp-exporter: $(GOPATH)
	git clone --depth 1 --branch $(EXPORTER_VERSION) https://$(APP_REMOTE) $(APPHOME)
	cd $(APPHOME) && git checkout $(EXPORTER_VERSION)
	cd $(APPHOME) && $(MAKE)

$(DEBNAME)_$(DEBVERSION)_%.deb: $(APPHOME)/bin/gobgp-exporter
	bundle exec fpm -f \
	-s dir \
	-t deb \
	--license Apache \
	--deb-priority optional \
	--deb-systemd-enable \
	--deb-systemd-restart-after-upgrade \
	--deb-systemd-auto-start \
	--after-install=deb-scripts/after-install.sh \
	--after-upgrade=deb-scripts/after-install.sh \
	--after-remove=deb-scripts/after-remove.sh \
	--depends adduser,systemd \
	--maintainer github@growse.com \
	--vendor https://github.com/greenpau/ \
	-n $(DEBNAME) \
	--description "$(APPDESCRIPTION)" \
	--url $(APPURL) \
	--prefix / \
	-a $* \
	-v $(DEBVERSION) \
	--deb-systemd deb-scripts/prometheus-gobgp-exporter.service \
	--deb-systemd-auto-start \
	--deb-systemd-enable \
	--deb-systemd-restart-after-upgrade \
	--config-files /etc/default/prometheus-gobgp-exporter \
	deb-scripts/prometheus-gobgp-exporter.defaults=/etc/default/prometheus-gobgp-exporter \
	$<=/usr/sbin/gobgp_exporter

.PHONY: clean
clean:
	chmod -R +w gopath
	rm -f *.deb
	rm -rf $(GOPATH)
