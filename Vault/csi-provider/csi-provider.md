# CSI Secrets Store Driver

The CSI Secrets Store driver enables users to create `SecretProviderClass` objects, which define the secret provider to use and the specific secrets to retrieve. When pods that request CSI volumes are created, the driver communicates with the Vault CSI Provider (if the provider is set to `vault`). The Vault CSI Provider retrieves the secrets from Vault using the specified Secret Provider Class and the pod's service account, then mounts these secrets into the pod's CSI volume.

Secrets are fetched from Vault and populated into the CSI secrets store volume during the `ContainerCreation` phase, causing the pods to be blocked from starting until the secrets are successfully retrieved and written to the volume.

## Features

The Vault CSI Provider includes the following capabilities:

- Support for all Vault secret engines
- Authentication via the requesting pod's service account
- Secure TLS/mTLS communications with Vault
- Ability to render Vault secrets as files
- Dynamic lease caching and renewal handled by the Agent
- Synchronization of secrets to Kubernetes secrets for use as environment variables
- Installation support via [Vault Helm](https://developer.hashicorp.com/vault/docs/platform/k8s/helm)

## Authenticating with Vault

The Vault CSI Provider authenticates with Vault using the service account associated with the pod that mounts the CSI volume. It supports both [Kubernetes](https://developer.hashicorp.com/vault/docs/auth/kubernetes) and [JWT](https://developer.hashicorp.com/vault/docs/auth/jwt) authentication methods. The pod's service account must be bound to a Vault role and a policy that allows access to the necessary secrets.

It is highly recommended to utilize dedicated Kubernetes service accounts for pods to restrict access to only the secrets that are required.

## Example: Secret Provider Class

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
    # Configuration for Vault address and TLS can typically be set using Helm, 
    # but can be overridden here:
    # vaultAddress: 'https://vault:8200'
    # vaultCACertPath: '/vault/tls/ca.crt'
    objects: |
      - objectName: "dbUsername"
        secretPath: "database/creds/db-app"
        secretKey: "username"
      - objectName: "dbPassword"
        secretPath: "database/creds/db-app"
        secretKey: "password"
    # "objectName" serves as an alias to reference the secret within the SecretProviderClass
    # and corresponds to the filename containing the secret.
    # "secretPath" is the path in Vault where the secret is retrieved from.
    # "secretKey" is the key used to extract the desired value from the Vault secret response.

```
## Deployment YAML

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
