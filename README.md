# Check the Vault Client Count in K8s

## Run a Vault Dev Server
```shell
vault server -dev -dev-root-token-id=root
```

## Prepare Vault
In a separate terminal run the following commands
```shell
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=root
vault auth enable kubernetes
```

## Prepare K8s
```shell
kubectl create ns app1-dev
kubectl create ns app1-test
```

## Get Vault Auditor Tool ready
```shell
mkdir logs
vault audit enable file file_path=./logs/vault_audit.log
```

## Write out the policy named app1 that enables the read capability for secrets at path secret/data/database/config
```shell
vault policy write app1 - <<EOH
path "secret/data/database/config" {
  capabilities = ["read"]
}
EOH
```

## Create a Kubernetes authentication role named app1:

```shell
vault write auth/kubernetes/role/app1-nonprod \
        bound_service_account_names=app1-vault \
        bound_service_account_namespaces=app1-dev,app1-test \
        policies=app1 \
        ttl=24h
```

## Create a service account
`service-account-app1-vault.yaml`

## Deploy an app
`deployment-orgchart.yml`


# Conclusion
I created two namespaces. One called `app1-dev` and another called `app1-test`. Both have the same service account called `app1-vault`. I created one role called `app1-nonprod`. This role looks like this:
```
vault write auth/kubernetes/role/app1-nonprod \
        bound_service_account_names=app1-vault \
        bound_service_account_namespaces=app1-dev,app1-test \
        policies=app1 \
        ttl=24h
```

Vault created two entities. Here is the metadata for the alias of the entities:

app1-TEST-f7ab1389-af78-410d-9622-763d7254864e
service_account_name `app1-vault`
service_account_namespace `app1-test`
service_account_secret_name `app1-vault-token-sv7gq`
service_account_uid `f7ab1389-af78-410d-9622-763d7254864e`

service_account_name `app1-vault`
service_account_namespace `app1-dev`
service_account_secret_name `app1-vault-token-t66jq`
service_account_uid `c1a78c0d-998a-4573-801c-7248d5af038a`