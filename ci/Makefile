include ../config.mk

.PHONY: build
build:
	$(MAKE) -C .. pushed-private-ci-image pushed-private-ci-utils-image pushed-private-hail-buildkit-image

.PHONY: deploy
deploy: build
	! [ -z $(NAMESPACE) ]  # call this like: make deploy NAMESPACE=default
	python3 jinja2_render.py '{"code":{"sha":"$(shell git rev-parse --short=12 HEAD)"},"global":{"cloud":"$(CLOUD)"},"deploy":$(DEPLOY),"scope":"$(SCOPE)","default_ns":{"name":"$(NAMESPACE)"},"ci_image":{"image":"$(shell cat ../pushed-private-ci-image)"},"ci_utils_image":{"image":"'$$(cat ../pushed-private-ci-utils-image)'"},"ci_database":{"user_secret_name":"sql-ci-user-config"},"hail_buildkit_image":{"image":"'$$(cat ../pushed-private-hail-buildkit-image)'"}}' deployment.yaml deployment.yaml.out
	kubectl -n $(NAMESPACE) apply -f deployment.yaml.out


validate-envoy-config:
	HAIL_PRODUCTION_DOMAIN=$(DOMAIN) python3 ci/envoy.py \
		$(PROXY) \
		../letsencrypt/subdomains.txt \
		../gateway/cds.yaml.out \
		../gateway/rds.yaml.out
	cd .. && docker run --rm -it -v gateway:/gateway envoyproxy/envoy:v1.22.3 envoy --mode validate --config-path /gateway/cds.yaml.out
