# application-secrets - DEPRECATED

## Migrating to External-Secrets
**Warning - This process may cause downtime**
1. Make a PR to update the secrets module defined in main.tf
 - Change the `official_chart_name` from `application-secrets` to `external-secrets`
2. Update the secret values in AWS Secrets Manager from a string to be keypair with the key value
    1. <img src=https://user-images.githubusercontent.com/93568025/192052794-0c80027e-687a-48e8-bd11-233ac3bff6f1.png width=200/>
    2. <img src=https://user-images.githubusercontent.com/93568025/192052675-c315a253-2cc8-481f-b90c-c8c15ebc4d99.png width=200/>
    3. <img src=https://user-images.githubusercontent.com/93568025/192052686-f65fa1b9-8835-488d-aba5-ef5dbf293252.png width=200/>
    4. <img src=https://user-images.githubusercontent.com/93568025/192052692-a4385d1f-0f1c-4e63-ab9b-021bffca6262.png width=400/>
3. Merge the PR from step 1 to deploy the changes




This Helm chart will set up a [Kubernetes External Sercets](https://github.com/external-secrets/kubernetes-external-secrets#kubernetes-external-secrets) with AWS Secrets Manager. 

### Creating a Secret
Use the following CLI command to create a secret:
```
aws secretsmanager create-secret --name <SECRET_NAME> --secret-string "<secret_value>" --profile <profile>
```
And validate that it appears in the AWS console.

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
  official_chart_name = "application-secrets"
  # This is what you want to name the chart when deploying
  user_chart_name = "${local.fullnameOverride}-secrets"
  # The helm chart version you want to use
  helm_version = "0.1.0"
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
