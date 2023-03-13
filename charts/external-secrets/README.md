# external-secrets

This Helm chart will set up an [External Sercet](https://external-secrets.io/v0.6.0-rc1/api/externalsecret/) in the context of an [External Secrets Operator](https://github.com/external-secrets/external-secrets) with AWS Secrets Manager.

## Requirements

Note: This is already configured in dev and stage clusters

<details>
    <summary><b>Configuring External Secrets Operator (click to expand)</b></summary>

1. Install the [External Secrets Operator Helm Chart](https://external-secrets.io/v0.6.0-rc1/guides/getting-started/#installing-with-helm) to the cluster.

- External-secrets runs within your Kubernetes cluster as a deployment resource. It utilizes CustomResourceDefinitions to configure access to secret providers through SecretStore resources and manages Kubernetes secret resources with ExternalSecret resources.

2. Create a generic secret with our AWS credentials

- This secret is needed for the Cluster Secret Store to access our AWS Secrets Manager
  ```
  apiVersion: v1
  kind: Secret
  metadata:
    name: awssm-secret
    namespace: app
  type: Opaque
  data:
    AWS_ACCESS_KEY_ID: <base64 encoded key>
    AWS_SECRET_ACCESS_KEY: <base64 encoded secret key>
  ```

3. Create a [Cluster Secret Store](https://external-secrets.io/v0.6.0-rc1/api/clustersecretstore/)

- The ClusterSecretStore is a cluster scoped SecretStore that can be referenced by all ExternalSecrets from all namespaces. Use it to offer a central gateway to your secret backend.

  ```
  apiVersion: external-secrets.io/v1beta1
  kind: ClusterSecretStore
  metadata:
    name: aws-secrets
  spec:
    provider:
      aws:
        service: SecretsManager
        region: us-east-1
        auth:
          # Getting the accessKeyID and secretAccessKey from an already created Kubernetes Secret
          secretRef:
            accessKeyIDSecretRef:
              name: awssm-secret
              namespace: app
              key: AWS_ACCESS_KEY_ID
            secretAccessKeySecretRef:
              name: awssm-secret
              namespace: app
              key: AWS_SECRET_ACCESS_KEY

  status:
    # Standard condition schema
    conditions:
    # SecretStore ready condition indicates the given store is in ready
    # state and able to referenced by ExternalSecrets
    # If the `status` of this condition is `False`, ExternalSecret controllers
    # should prevent attempts to fetch secrets
    - type: Ready
      status: "False"
      reason: "ConfigError"
      message: "SecretStore validation failed"
      lastTransitionTime: "2019-08-12T12:33:02Z"
  ```

````
```js
function logSomething(something) {
  console.log('Something', something);
}
````

</details>

<br />

## Migrating from Application-Secrets

**Warning - This process may cause downtime**

<details>
    <summary><b>Steps to migrate from deprecated application-secrets (click to expand)</b></summary>

1. Make a PR to update the secrets module defined in main.tf

- Change the `official_chart_name` from `application-secrets` to `external-secrets`

2. Update the secret values in AWS Secrets Manager from a string to be keypair with the key value
   1. <img src=https://user-images.githubusercontent.com/93568025/192052794-0c80027e-687a-48e8-bd11-233ac3bff6f1.png width=200/>
   2. <img src=https://user-images.githubusercontent.com/93568025/192052675-c315a253-2cc8-481f-b90c-c8c15ebc4d99.png width=200/>
   3. <img src=https://user-images.githubusercontent.com/93568025/192052686-f65fa1b9-8835-488d-aba5-ef5dbf293252.png width=200/>
   4. <img src=https://user-images.githubusercontent.com/93568025/192052692-a4385d1f-0f1c-4e63-ab9b-021bffca6262.png width=400/>
3. Merge the PR from step 1 to deploy the changes

```js
function logSomething(something) {
  console.log("Something", something);
}
```

</details>

<br />

### Creating a Secret

Use the following CLI command to create a secret:

```
aws secretsmanager create-secret --name <SECRET_NAME> --secret-string "{\"value\":\"<SECRET_VALUE>\"}" --profile <profile>
```

And validate that it appears in the AWS console as a key/value pair like the image below.

<img src=https://user-images.githubusercontent.com/93568025/192052692-a4385d1f-0f1c-4e63-ab9b-021bffca6262.png width=400/>

SECRET_EXAMPLE-NUMBER-ONE in Secret Manager will map to a secret.exampleNumberOne env var

### Configuring a Secrets Deployment

Set `local.secrets` in `main.tf` to the list of secrets you want injected as environment variables into the deployment

```
locals {
  .
  .
  secrets = ["SECRET_NAME_1", "SECRET_NAME_2"]
  .
  .
}
```

Add the following section to `main.tf`

```
module "secrets" {
  source = "github.com/ManagedKube/kubernetes-ops//terraform-modules/aws/helm/helm_generic?ref=v1.0.9"

  # this is the helm repo add URL
  repository = "https://thebitgram.github.io/helm-charts/"
  # This is the helm repo add name
  official_chart_name = "external-secrets"
  # This is what you want to name the chart when deploying
  user_chart_name = "${local.fullnameOverride}-secrets"
  # The helm chart version you want to use
  helm_version = "0.4.0"
  # The namespace you want to install the chart into - it will create the namespace if it doesnt exist
  namespace = local.namespace
  # The helm chart values file
  helm_values = templatefile("${path.module}/secret_helm_values.yaml", {
    fullnameOverride = "${local.fullnameOverride}-secrets"
    namespace        = local.namespace
    secrets          = local.secrets
  })

  depends_on = [
    data.terraform_remote_state.eks
  ]
}
```

Make the App deployment depend on the secrets module

```
  depends_on = [
    module.secrets
  ]
```

Create the file `secret_helm_values.yaml` in the same directory as `main.tf` & `helm_values.yaml`

```
fullnameOverride: ${fullnameOverride}

namespace: ${namespace}

secrets: %{for secret in secrets}
  - ${secret}
  %{endfor}

```

### Referencing the secret in a Deployment

In the `helm_values.yaml` file, add this configuration for `envFrom` inside `containers`:

```
      envFrom:
      - secretRef:
          name: ${fullnameOverride}-secrets
```

Now you can reference the AWS secret as an environment variable inside the deployment.

### Failure to deserialize JSON values

We have observed issues with deseriailzing the secret value from they keypair. If this occurs, take the following steps:

1. Create a new secret in AWS of type "Other" and paste only the value. It should appear as in the example below. Name this secret <secret-name>\_RAW
2. Update the `secret_helm_values.yaml` file to add the line `useRaw: true`.
3. Update the `main.tf` file to use version "0.4.0"
