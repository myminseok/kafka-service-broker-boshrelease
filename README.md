# BOSH release for kafka-service-broker

This BOSH release and deployment manifest deploy a cluster of kafka/kafka manager/zookeeper, plus a Cloud Foundry service broker.

## Install

```shell
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka-service-broker

# name of cf deployment in BOSH, e.g. 'cf'
cf=cf
system_domain=$(bosh -d $cf manifest | bosh int - --path /instance_groups/name=api/jobs/name=cloud_controller_ng/properties/system_domain)
skip_verify=$(bosh -d $cf manifest | bosh int - --path /instance_groups/name=api/jobs/name=cloud_controller_ng/properties/ssl/skip_cert_verify)

cf_admin_password=$(credhub get -n $(credhub find -n cf/cf_admin_password -j | jq -r ".credentials[0].name") -j | jq -r ".value")

bosh deploy manifests/kafka-service-broker.yml \
  --vars-store tmp/creds.yml \
  -o manifests/operators/cf-integration.yml \
  -v cf-api-url=https://api.$system_domain \
  -v cf-skip-ssl-validation=$skip_verify \
  -v cf-admin-username=admin \
  -v "cf-admin-password=$cf_admin_password"

bosh -e d deploy -d kafka-service-broker ./manifests/kafka-service-broker.yml \
  --vars-store tmp/creds.yml \
  -o manifests/operators/cf-integration.yml \
  -o manifests/operators/az-override.yml \
  -o manifests/operators/stemcell-override.yml \
  -o manifests/operators/disk-override.yml \
  -o manifests/operators/single.yml \
  -o manifests/operators/networking.yml \
  -o manifests/operators/release-override.yml \
  -o manifests/operators/release-remove-url.yml \
  -v cf-api-url=https://api.$system_domain \
  -v cf-skip-ssl-validation=$skip_verify \
  -v cf-admin-username=admin \
  -v "cf-admin-password=$cf_admin_password" \
  -v kafka_network_name=$kafka_network_name  \
  -v stemcell_version=456.latest \
  -v zookeeper_persistent_disk=10240 \
  -v kafka_persistent_disk=10240 \
  -v kafka-manager_persistent_disk=10240 \
  -v "kafka_azs=[$az]"


bosh run-errand broker-registrar
```

The `cf-*` variables are used to register the broker as a service broker on Cloud Foundry.

### Uninstall

To uninstall the service broker and kafka/zookeeper clusters=... \

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka-service-broker
bosh run-errand broker-deregistrar
bosh delete-deployment
```
