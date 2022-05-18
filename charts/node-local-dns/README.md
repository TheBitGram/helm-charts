# node-local-dns

This Helm chart will set up a NodeLocal DNSCache, by setting up the resources as outlined here: https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/

### Values

The following table lists the configurable parameters of the Node-local-dns chart and their default values.

| Parameter                | Description             | Default        |
| ------------------------ | ----------------------- | -------------- |
| `dnsUpstreamName` | The name for the KubeDNSUpstream service | `"kube-dns-upstream"` |
| `image.repository` |  | `"k8s.gcr.io/dns/k8s-dns-node-cache"` |
| `image.pullPolicy` |  | `"IfNotPresent"` |
| `image.tag` |  | `"1.21.1"` |
| `image.args.skipTeardown` |  | `true` |
| `image.args.syncInterval` |  | `"1ns"` |
| `image.args.interfaceName` |  | `"nodelocaldns"` |
| `image.args.healthPort` | `8080` |
| `image.args.setupIptables` | `false` |
| `image.args.setupEbtables` | `false` |
| `image.args.quiet` | `false` |
| `imagePullSecrets` |  | `[]` |
| `config.localDnsIp` | The loopback local IP address for the NodeLocal DNSCache, or `__PILLAR__LOCAL__DNS__` | `"169.254.20.11"` |
| `config.dnsServerIp` | The Cluster IP of the kube-dns service, or `__PILLAR__DNS__SERVER__` | `"172.20.0.10"` |
| `config.zones` | Sets up the CoreDNS Corefile | `{...}` |
| `useHostNetwork` |  | `true` |
| `updateStrategy.rollingUpdate.maxUnavailable` |  | `"10%"` |
| `priorityClassName` |  | `"system-node-critical"` |
| `podAnnotations` |  | `{}` |
| `podSecurityContext` |  | `{}` |
| `securityContext.privileged` |  | `true` |
| `readinessProbe` |  | `null` |
| `serviceAccount.create` |  | `true` |
| `serviceAccount.annotations` |  | `{}` |
| `serviceAccount.name` |  | `""` |
| `nodeSelector` |  | `{}` |
| `affinity` |  | `{}` |
| `tolerations` |  | `[{"key": "CriticalAddonsOnly", "operator": "Exists"}, {"effect": "NoExecute", "operator": "Exists"}, {"effect": "NoSchedule", "operator": "Exists"}]` |
| `resources.requests.cpu` |  | `"30m"` |
| `resources.requests.memory` |  | `"50Mi"` |
| `metrics.prometheusScrape` |  | `"true"` |
| `metrics.port` |  | `9253` |
