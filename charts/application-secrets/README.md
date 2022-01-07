# application-secrets

This Helm chart will set up a [Kubernetes External Sercets](https://github.com/external-secrets/kubernetes-external-secrets#kubernetes-external-secrets) with AWS Secrets Manager. 

### Creating a Secret
Use the following CLI command to create a secret:
```
aws secretsmanager create-secret --name <SECRET_NAME> --secret-string "<secret_value>" --profile <profile>
```
And validate that it appears in the AWS console.

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
