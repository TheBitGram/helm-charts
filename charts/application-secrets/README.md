# application-secrets

This Helm chart will set up a [Kubernetes External Sercets](https://github.com/external-secrets/kubernetes-external-secrets#kubernetes-external-secrets) with AWS Secrets Manager. 

### Creating a Secret
Use the following CLI command to create a secret:
```
aws secretsmanager create-secret --name <secret_name> --secret-string "<secret_value>" --profile <profile>
```
And validate that it appears in the AWS console.

### Referencing the secret in a Deployment
In the `helm_values.yaml` file, add this configuration for `env` inside `containers`:
```
      env:
        base:
        - name: <ENV_VAR_NAME_1>
          valueFrom: 
            secretKeyRef:
              name: ${fullnameOverride}
              key: <secret_name_1> # must match name from external secrets
        - name: <ENV_VAR_NAME_2>
          valueFrom: 
            secretKeyRef:
              name: <external_secret_module_name>
              key: <secret_name_2> # must match name from external secrets
        perEnv: [ ]
```

Now you can reference the AWS secret as an environment variable inside the deployment.
