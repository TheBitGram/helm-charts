# idsvr-config-params

This Helm chart will set up an Kuberenetes Secret containing parameters for a Curity Identity Server configuration file. This Secret can then be used to set environment variables on the Identity Server. 

The primary purpose of this is to allow for a [parameterized configuration](https://curity.io/docs/idsvr/latest/configuration-guide/parameterized-configuration.html) while still keeping the parameters secure.
