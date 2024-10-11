SERVER_CHART_VERSION := 32.0.0
DAEMON_CHART_VERSION := 32.0.2
UI_CHART_VERSION := 32.0.1

helm:
	helm repo add rucio https://rucio.github.io/helm-charts
	helm repo update rucio

rucio-server: helm
	helm template usdf rucio/rucio-server --version=${SERVER_CHART_VERSION} --values=values-rucio-server.yaml > helm-rucio-server.yaml

rucio-daemons: helm
	helm template usdf rucio/rucio-daemons --version=${DAEMON_CHART_VERSION} --values=values-rucio-daemons.yaml > helm-rucio-daemons.yaml

rucio-ui: helm
	helm template usdf rucio/rucio-ui --version=${UI_CHART_VERSION} --values=values-rucio-ui.yaml > helm-rucio-ui.yaml

rucio: rucio-server rucio-daemons rucio-ui

package-reaper-certs:
	rm -rf etc/.secrets/grid-certificates
	mkdir -p etc/.secrets/grid-certificates
	cp /etc/grid-security/certificates/*.0 etc/.secrets/grid-certificates/
	cp /etc/grid-security/certificates/*.signing_policy etc/.secrets/grid-certificates/
	cd etc/.secrets && tar cfz grid-certificates.tar.gz grid-certificates/

store-reaper-certs: package-reaper-certs
	base64 --wrap=0 etc/.secrets/grid-certificates.tar.gz | vault kv patch secret/rubin/usdf-rucio/rucio grid-certificates.tar.gz=-

get-secrets:
	mkdir -p rucio/etc/.secrets
	vault kv get --field=usdf-server-hostkey secret/rubin/usdf-rucio/rucio  > etc/.secrets/hostkey.pem
	vault kv get --field=usdf-server-hostcert secret/rubin/usdf-rucio/rucio  > etc/.secrets/hostcert.pem
	vault kv get --field=usdf-server-cafile secret/rubin/usdf-rucio/rucio  > etc/.secrets/ca.pem
	vault kv get --field=usdf-db-conn-str secret/rubin/usdf-rucio/rucio  > etc/.secrets/db-conn-str.txt
	vault kv get --field=usdf-elasticsearch-password secret/rubin/usdf-rucio/rucio  > etc/.secrets/usdf-elasticsearch-password	
	cp etc/automatix.json etc/.secrets/automatix.json

clean-secrets:
	rm -rf etc/.secrets 

run-dump: 
	kubectl kustomize .

dump: get-secrets rucio run-dump clean-secrets

run-apply:  
	kubectl apply -k .

apply: get-secrets rucio run-apply clean-secrets

run-destroy:
	kubectl delete -k .

destroy: get-secrets rucio run-destroy clean-secrets


