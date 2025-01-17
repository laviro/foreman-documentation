SHELL := /bin/bash
BUILD = foreman-el
BUILD_DIR = ../build
BUILD_DIRA = $(realpath $(BUILD_DIR))/
COMMON_DIR = ../common
ROOTDIR = $(realpath .)
NAME = $(notdir $(ROOTDIR))
DEST_DIR = $(BUILD_DIR)/$(subst doc-,,$(NAME))
DEST_HTML = $(DEST_DIR)/index-$(BUILD).html
DEST_PDF = $(BUILD_DIR)/$(subst doc-,,$(NAME))-$(BUILD).pdf
IMAGES_DIR = $(DEST_DIR)/images
IMAGES_TS = $(DEST_DIR)/.timestamp-images
CSS_SOURCES = $(shell find $(COMMON_DIR)/sass -type f)
SOURCES = master.adoc $(shell find . -type f -name \*.adoc)
DEPENDENCIES = $(shell find $(COMMON_DIR) -type f -name \*.adoc)
IMAGES = $(shell find ./images ./common/images -type f 2>/dev/null || true)
UNAME = $(shell uname)
DATE_MDY = $(shell LC_ALL=C date "+%b %d %Y")
DATE_MY = $(shell LC_ALL=C date "+%b %Y")

ifeq ($(UNAME), Linux)
BROWSER_OPEN = xdg-open
endif
ifeq ($(UNAME), Darwin)
BROWSER_OPEN = open
endif

ifeq ($(LINKS), LOCAL)
BASEURL_ATTR = "-a BaseURL=file://$(BUILD_DIRA)"
endif

all: html

html: prepare $(DEST_DIR)/$(BUILD).css $(IMAGES_TS) $(DEST_HTML)

pdf: prepare $(DEST_PDF)

prepare:
	@mkdir -p $(BUILD_DIR) $(DEST_DIR) $(IMAGES_DIR) $(IMAGES_DIR) $(DEST_DIR)/common/images

clean:
	@rm -rf "$(DEST_DIR)" "$(DEST_PDF)"

browser: html
	${BROWSER_OPEN} "file://$(realpath $(ROOTDIR)/$(DEST_HTML))"

serve: html
	python -m http.server --directory "$(realpath $(DEST_DIR))" 8813

linkchecker: html
	linkchecker -r1 -f $(COMMON_DIR)/linkchecker.ini --check-extern "file://$(realpath $(ROOTDIR)/$(DEST_HTML))"

open-pdf: pdf
	${BROWSER_OPEN} "$(realpath $(ROOTDIR)/$(DEST_PDF))"

$(BUILD_DIR)/default.css: $(CSS_SOURCES)
	bundle exec compass compile --force -c $(COMMON_DIR)/sass_config.rb --sass-dir $(COMMON_DIR)/sass --css-dir $(BUILD_DIR)

$(DEST_DIR)/$(BUILD).css: $(BUILD_DIR)/default.css
	cp $< $@

$(IMAGES_TS): $(IMAGES)
	cp $? $(IMAGES_DIR)/
	pushd "$(IMAGES_DIR)" >/dev/null && for IMG in *.*; do ln -s ../../images/$$IMG ../common/images/$$IMG; done && popd >/dev/null
	@touch $(IMAGES_TS)

$(DEST_HTML): $(SOURCES) $(DEPENDENCIES)
	bundle exec asciidoctor -a build=$(BUILD) $(BASEURL_ATTR) -a docinfo=shared -a date_mdy="$(DATE_MDY)" -a date_my="$(DATE_MY)" -a docdatetime="$(DATE_MY)" -a stylesheet=$(BUILD).css -a linkcss=true -b html5 -d book --trace -o $@ $< 2>&1 | tee $@.log
	@echo CHECKING FOR ASCIIDOCTOR ERRORS AND WARNINGS
	@! grep -Eq "^asciidoctor: (ERROR|WARNING)" $@.log
	@echo CHECKING FOR ORPHAN XREFS HTML LINKS
	@! grep -RE '<a href="#[a-zA-Z0-9_-]+">\[[a-zA-Z0-9_-]+\]</a>' $@

$(DEST_PDF): $(SOURCES) $(IMAGES) $(DEPENDENCIES)
	bundle exec asciidoctor-pdf -a build=$(BUILD) -d book --trace -o $@ $< 2>&1 | tee $@.log
	@echo CHECKING FOR ASCIIDOCTOR-PDF ERRORS AND WARNINGS
	@! grep -Eq "^asciidoctor: (ERROR|WARNING)" $@.log
