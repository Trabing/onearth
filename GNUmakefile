# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
# http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ONEARTH_VERSION=1.1.1

PREFIX=/usr/local
SMP_FLAGS=-j $(shell cat /proc/cpuinfo | grep processor | wc -l)
LIB_DIR=$(shell \
	[ "$(shell arch)" == "x86_64" ] \
		&& echo "lib64" \
		|| echo "lib" \
)
RPMBUILD_FLAGS=-ba

MPL_ARTIFACT=matplotlib-1.5.1.tar.gz
MPL_URL=https://pypi.python.org/packages/source/m/matplotlib/$(MPL_ARTIFACT)
CGICC_ARTIFACT=cgicc-3.2.16.tar.gz
CGICC_URL=http://ftp.gnu.org/gnu/cgicc/$(CGICC_ARTIFACT)

MAPSERVER_VERSION=7.0.1
MAPSERVER_ARTIFACT=mapserver-$(MAPSERVER_VERSION).tar.gz
MAPSERVER_HOME=http://download.osgeo.org/mapserver
MAPSERVER_URL=$(MAPSERVER_HOME)/$(MAPSERVER_ARTIFACT)

all: 
	@echo "Use targets onearth-rpm"

onearth: mpl-unpack cgicc-unpack mapserver-unpack onearth-compile

#-----------------------------------------------------------------------------
# Download
#-----------------------------------------------------------------------------

download: mpl-download cgicc-download mapserver-download
	
mpl-download: upstream/$(MPL_ARTIFACT).downloaded

upstream/$(MPL_ARTIFACT).downloaded: 
	mkdir -p upstream
	rm -f upstream/$(MPL_ARTIFACT)
	( cd upstream ; wget $(MPL_URL) )
	touch upstream/$(MPL_ARTIFACT).downloaded
	
cgicc-download: upstream/$(CGICC_ARTIFACT).downloaded

upstream/$(CGICC_ARTIFACT).downloaded: 
	mkdir -p upstream
	rm -f upstream/$(CGICC_ARTIFACT)
	( cd upstream ; wget $(CGICC_URL) )
	touch upstream/$(CGICC_ARTIFACT).downloaded
	
mapserver-download: upstream/$(MAPSERVER_ARTIFACT).downloaded

upstream/$(MAPSERVER_ARTIFACT).downloaded:
	mkdir -p upstream
	rm -rf upstream/$(MAPSERVER_ARTIFACT)
	( cd upstream ; wget $(MAPSERVER_URL) )
	touch upstream/$(MAPSERVER_ARTIFACT).downloaded

#-----------------------------------------------------------------------------
# Compile
#-----------------------------------------------------------------------------
		
mpl-unpack: build/mpl/VERSION

build/mpl/VERSION:
	mkdir -p build/mpl
	tar xf upstream/$(MPL_ARTIFACT) -C build/mpl \
		--strip-components=1 --exclude=.gitignore
		
cgicc-unpack: build/cgicc/VERSION

build/cgicc/VERSION:
	mkdir -p build/cgicc
	tar xf upstream/$(CGICC_ARTIFACT) -C build/cgicc \
		--strip-components=1 --exclude=.gitignore
		
mapserver-unpack: build/mapserver/VERSION

build/mapserver/VERSION:
	mkdir -p build/mapserver
	tar xf upstream/$(MAPSERVER_ARTIFACT) -C build/mapserver \
		--strip-components=1 --exclude=.gitignore

onearth-compile:
	$(MAKE) -C src/modules/mod_onearth
	$(MAKE) -C src/modules/mod_oems
	$(MAKE) -C src/modules/mod_oemstime

#-----------------------------------------------------------------------------
# Install
#-----------------------------------------------------------------------------
install: onearth-install 

onearth-install:
	install -m 755 -d $(DESTDIR)/$(PREFIX)/$(LIB_DIR)/httpd/modules
	install -m 755 src/modules/mod_onearth/.libs/mod_twms.so \
		$(DESTDIR)/$(PREFIX)/$(LIB_DIR)/httpd/modules/mod_twms.so
	install -m 755 src/modules/mod_onearth/.libs/mod_onearth.so \
		$(DESTDIR)/$(PREFIX)/$(LIB_DIR)/httpd/modules/mod_onearth.so
	install -m 755 src/modules/mod_oems/.libs/mod_oems.so \
		$(DESTDIR)/$(PREFIX)/$(LIB_DIR)/httpd/modules/mod_oems.so
	install -m 755 src/modules/mod_oemstime/.libs/mod_oemstime.so \
		$(DESTDIR)/$(PREFIX)/$(LIB_DIR)/httpd/modules/mod_oemstime.so
		
	install -m 755 -d $(DESTDIR)/$(PREFIX)/bin
	install -m 755 src/modules/mod_onearth/oe_create_cache_config \
		$(DESTDIR)/$(PREFIX)/bin/oe_create_cache_config
	install -m 755 src/layer_config/bin/oe_configure_layer.py  \
		-D $(DESTDIR)/$(PREFIX)/bin/oe_configure_layer
	install -m 755 src/layer_config/bin/oe_generate_empty_tile.py  \
		-D $(DESTDIR)/$(PREFIX)/bin/oe_generate_empty_tile.py
	install -m 755 src/onearth_logs/onearth_logs.py  \
		-D $(DESTDIR)/$(PREFIX)/bin/onearth_metrics
	install -m 755 src/generate_legend/oe_generate_legend.py  \
		-D $(DESTDIR)/$(PREFIX)/bin/oe_generate_legend.py
	install -m 755 src/mrfgen/mrfgen.py  \
		-D $(DESTDIR)/$(PREFIX)/bin/mrfgen
	install -m 755 src/mrfgen/colormap2vrt.py  \
		-D $(DESTDIR)/$(PREFIX)/bin/colormap2vrt.py
	install -m 755 src/mrfgen/RGBApng2Palpng  \
		-D $(DESTDIR)/$(PREFIX)/bin/RGBApng2Palpng
	install -m 755 src/scripts/oe_validate_palette.py  \
		-D $(DESTDIR)/$(PREFIX)/bin/oe_validate_palette.py

	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/onearth
	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/onearth/empty_tiles
	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/onearth/apache
	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/onearth/apache/kml
	install -m 755 src/cgi/twms.cgi \
		-t $(DESTDIR)/$(PREFIX)/share/onearth/apache
	install -m 755 src/cgi/wmts.cgi \
		-t $(DESTDIR)/$(PREFIX)/share/onearth/apache
	cp src/cgi/kml/* \
		-t $(DESTDIR)/$(PREFIX)/share/onearth/apache/kml
	cp src/cgi/index.html \
		-t $(DESTDIR)/$(PREFIX)/share/onearth/apache
	cp src/mrfgen/empty_tiles/* \
		-t $(DESTDIR)/$(PREFIX)/share/onearth/empty_tiles

	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/onearth/mrfgen
	cp src/mrfgen/empty_tiles/* \
		$(DESTDIR)/$(PREFIX)/share/onearth/mrfgen

	install -m 755 -d $(DESTDIR)/etc/onearth/config
	cp -r src/layer_config/conf \
		$(DESTDIR)/etc/onearth/config
	cp -r src/layer_config/layers \
		$(DESTDIR)/etc/onearth/config
	cp -r src/layer_config/schema \
		$(DESTDIR)/etc/onearth/config
	cp -r src/layer_config/mapserver \
		$(DESTDIR)/etc/onearth/config
	install -m 755 -d $(DESTDIR)/etc/onearth/config/headers

	install -m 755 -d $(DESTDIR)/etc/onearth/metrics
	cp -r src/onearth_logs/logs.* \
		$(DESTDIR)/etc/onearth/metrics
	cp -r src/onearth_logs/tilematrixsetmap.* \
		$(DESTDIR)/etc/onearth/metrics

	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/onearth/demo
	cp -r src/demo/* $(DESTDIR)/$(PREFIX)/share/onearth/demo

	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/mpl
	cp -r build/mpl/* $(DESTDIR)/$(PREFIX)/share/mpl
	install -m 755 -d $(DESTDIR)/$(PREFIX)/share/cgicc
	cp -r build/cgicc/* $(DESTDIR)/$(PREFIX)/share/cgicc


#-----------------------------------------------------------------------------
# Local install
#-----------------------------------------------------------------------------
local-install: onearth-local-install

onearth-local-install: 
	mkdir -p build/install
	$(MAKE) onearth-install DESTDIR=$(PWD)/build/install

#-----------------------------------------------------------------------------
# Artifacts
#-----------------------------------------------------------------------------
artifacts: onearth-artifact

onearth-artifact: onearth-clean
	mkdir -p dist
	rm -rf dist/onearth-$(ONEARTH_VERSION).tar.bz2
	tar cjvf dist/onearth-$(ONEARTH_VERSION).tar.bz2 \
		--transform="s,^,onearth-$(ONEARTH_VERSION)/," \
		src/modules/mod_onearth src/modules/mod_oems src/modules/mod_oemstime src/scripts \
		src/layer_config src/mrfgen src/cgi src/demo src/onearth_logs src/generate_legend GNUmakefile

#-----------------------------------------------------------------------------
# RPM
#-----------------------------------------------------------------------------
rpm: onearth-rpm

onearth-rpm: onearth-artifact 
	mkdir -p build/rpmbuild/SOURCES
	mkdir -p build/rpmbuild/BUILD	
	mkdir -p build/rpmbuild/BUILDROOT
	rm -f dist/onearth*.rpm
	cp \
		upstream/$(MPL_ARTIFACT) \
		upstream/$(CGICC_ARTIFACT) \
		upstream/$(MAPSERVER_ARTIFACT) \
		dist/onearth-$(ONEARTH_VERSION).tar.bz2 \
		build/rpmbuild/SOURCES
	rpmbuild \
		--define _topdir\ "$(PWD)/build/rpmbuild" \
		-ba deploy/onearth/onearth.spec 
	mv build/rpmbuild/RPMS/*/onearth*.rpm dist

#-----------------------------------------------------------------------------
# Mock
#-----------------------------------------------------------------------------
mock: onearth-mock

onearth-mock:
	mock --clean
	mock --init
	mock --copyin dist/gibs-gdal-*$(GDAL_VERSION)-*.$(shell arch).rpm /
	mock --install yum
	mock --shell \
	       "yum install -y /gibs-gdal-*$(GDAL_VERSION)-*.$(shell arch).rpm"
	mock --rebuild --no-clean \
		dist/mod_twms-$(ONEARTH_VERSION)-*.src.rpm

#-----------------------------------------------------------------------------
# Clean
#-----------------------------------------------------------------------------
clean: onearth-clean
	rm -rf build

onearth-clean:
	$(MAKE) -C src/modules/mod_onearth clean
	$(MAKE) -C src/modules/mod_oems clean
	$(MAKE) -C src/modules/mod_oemstime clean

distclean: clean
	rm -rf dist
	rm -rf upstream


