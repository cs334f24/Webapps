.PHONY: all install test info

AVAILABLE_WEBAPPS := $(wildcard webapps-available/*)

all: install

install:
	cp ./webapps_mixins/* /etc/nginx/webapps-mixins
	cp ./webapps-available/* /etc/nginx/webapps-available

test:
	nginx -t

info:
	@echo "AVAILABLE WEBAPPS = $(shell ls /etc/nginx/webapps-available)"
