
# CSI Secrets Store Driver

The CSI Secrets Store driver allows users to create `SecretProviderClass` objects that define which secret provider to use and what secrets to retrieve. When pods requesting CSI volumes are created, the CSI Secrets Store driver sends the request to the Vault CSI Provider if the provider is set to `vault`. The Vault CSI Provider utilizes the specified Secret Provider Class and the pod's service account to retrieve secrets from Vault and mount them into the pod's CSI volume.

Secrets are retrieved from Vault and populated into the CSI secrets store volume during the `ContainerCreation` phase. This means that pods will be blocked from starting until the secrets have been read from Vault and written to the volume.

## Features

The Vault CSI Provider supports the following features:

- All Vault secret engines
- Authentication using the requesting pod's service account
- TLS/mTLS communications with Vault
- Rendering Vault secrets to files
- Dynamic lease caching and renewal performed by Agent
- Syncing secrets to Kubernetes secrets for use as environment variables
- Installation via [Vault Helm](https://developer.hashicorp.com/vault/docs/platform/k8s/helm)

## Authenticating with Vault

The Vault CSI Provider authenticates with Vault using the service account of the pod that mounts the CSI volume. Both [Kubernetes](https://developer.hashicorp.com/vault/docs/auth/kubernetes) and [JWT](https://developer.hashicorp.com/vault/docs/auth/jwt) authentication methods are supported. The pod's service account must be bound to a Vault role and a policy that grants access to the desired secrets.

It is highly recommended to run pods with dedicated Kubernetes service accounts to ensure that applications cannot access more secrets than they require.

## Secret Provider Class Example

### SecretProviderClass YAML

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: vault-db-creds
spec:
  provider: vault
  parameters:
    roleName: 'app'
    # Vault address and TLS configuration can typically be set via the Helm chart, 
    # but can be overridden per SecretProviderClass:
    # vaultAddress: 'https://vault:8200'
    # vaultCACertPath: '/vault/tls/ca.crt'
    objects: |
      - objectName: "dbUsername"
        secretPath: "database/creds/db-app"
        secretKey: "username"
      - objectName: "dbPassword"
        secretPath: "database/creds/db-app"
        secretKey: "password"
    # "objectName" serves as an alias within the SecretProviderClass to reference
    # a specific secret. This also corresponds to the filename containing the secret.
    # "secretPath" indicates the path in Vault from which the secret should be retrieved.
    # "secretKey" specifies the key within the Vault secret response to extract a value.
```
```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  labels:
    app: demo
spec:
  selector:
    matchLabels:
      app: demo
  replicas: 1
  template:
    spec:
      serviceAccountName: app
      containers:
        - name: app
          image: my-app:1.0.0
          volumeMounts:
            - name: 'vault-db-creds'
              mountPath: '/mnt/secrets-store'
              readOnly: true
      volumes:
        - name: vault-db-creds
          csi:
            driver: 'secrets-store.csi.k8s.io'
            readOnly: true
            volumeAttributes:
              secretProviderClass: 'vault-db-creds'
