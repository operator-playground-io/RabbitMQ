---
title: How to Uninstall rabbitmq Helm chart 
description: This tutorial explains how to Uninstall rabbitmq helm chart
---

### Uninstall rabbitmq Helm Chart

Check your deployed rabbitmq helm chart:

```execute
 helm list -n rabbitmq
```

 Output will be similar:

```output

```

To uninstall the rabbitmq helm chart, use the helm delete command:

```execute
helm delete rabbitmq -n rabbitmq
```

Output:

```output
release "rabbitmq" uninstalled
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

```execute
helm status rabbitmq -n rabbitmq
```

Output:

```
Error: release: not found
```

### Cleanup other resources


To delete the PVCs created, execute the below commands:

```execute
kubectl -n rabbitmq get pvc | grep rabbitmq
```

```execute
kubectl -n rabbitmq delete pvc/PVC_VOLUME
```

Output:

```output

```
