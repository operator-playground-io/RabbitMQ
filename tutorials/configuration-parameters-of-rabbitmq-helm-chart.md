---
title: Configuration Parameters of rabbitmq Helm Chart
description: This tutorial explains about Configuration Parameters
---

### Parameters

Please check the below link for Parameters.

https://github.com/bitnami/charts/tree/master/bitnami/rabbitmq 


```execute
$ helm install rabbitmq \
  --set auth.username=admin,auth.password=secretpassword,auth.erlangCookie=secretcookie \
    bitnami/rabbitmq -n rabbitmq
```

The above command sets the RabbitMQ admin username and password to `admin` and `secretpassword` respectively. Additionally the secure erlang cookie is set to `secretcookie`.

### Configuration and installation details

**[Rolling VS Immutable tags]** (https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/)

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

Bitnami will release a new chart updating its containers if a new version of the main container, significant changes, or critical vulnerabilities exist.

 **Setting Pod's affinity**

This chart allows you to set your custom affinity using the `affinity` parameter. Find more information about Pod's affinity in the [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

As an alternative, you can use of the preset configurations for pod affinity, pod anti-affinity, and node affinity available at the [bitnami/common](https://github.com/bitnami/charts/tree/master/bitnami/common#affinities) chart. To do so, set the `podAffinityPreset`, `podAntiAffinityPreset`, or `nodeAffinityPreset` parameters.

 **Horizontal scaling**

To horizontally scale this chart once it has been deployed you have two options:

- Use `kubectl scale` command.
- Upgrading the chart with the following parameters:

```console
replicaCount=3
auth.password="$RABBITMQ_PASSWORD"
auth.erlangCookie="$RABBITMQ_ERLANG_COOKIE"

```

> Note: please note it's mandatory to indicate the password and erlangCookie that was set the first time the chart was installed to upgrade the chart. Otherwise, new pods won't be able to join the cluster.

When scaling down the solution unnecessary RabbitMQ nodes are automatically stopped, but they are not removed from the cluster. You need to manually remove them running the `rabbitmqctl forget_cluster_node` command. For instance, if you initially installed RabbitMQ with 3 replicas and then you scaled it down to 2 replicas, run the commands below (assuming that the release name is `rabbitmq` and you're using `hostname` as clustering type):

```execute
$ kubectl exec rabbitmq-0 --container rabbitmq -- rabbitmqctl forget_cluster_node rabbit@rabbitmq-2.rabbitmq-headless.default.svc.cluster.local
$ kubectl delete pvc data-rabbitmq-2
```

### Enabling TLS support

To enable TLS support you must generate the certificates using RabbitMQ [documentation](https://www.rabbitmq.com/ssl.html#automated-certificate-generation). Once you have your certificate, you have two alternatives:

A) Create a secret including the certificates:

```execute
$ kubectl create secret generic rabbitmq-certificates --from-file=./ca.crt --from-file=./tls.crt --from-file=./tls.key
```

Then, install the RabbitMQ chart setting the parameters below:

```execute
tls.enabled=true
tls.existingSecret=rabbitmq-certificates
```

B) Include the certificates in your values.yaml:

```yaml
auth:
  enabled: true
  caCertificate: |-
    -----BEGIN CERTIFICATE-----
    MIIDRTCCAi2gAwIBAgIJAJPh+paO6a3cMA0GCSqGSIb3DQEBCwUAMDExIDAeBgNV
    ...
    -----END CERTIFICATE-----
  serverCertificate: |-
    -----BEGIN CERTIFICATE-----
    MIIDqjCCApKgAwIBAgIBATANBgkqhkiG9w0BAQsFADAxMSAwHgYDVQQDDBdUTFNH
    ...
    -----END CERTIFICATE-----
  serverKey: |-
    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEA2iX3M4d3LHrRAoVUbeFZN3EaGzKhyBsz7GWwTgETiNj+AL7p
    ....
    -----END RSA PRIVATE KEY-----
```

- Setting [auth.tls.failIfNoPeerCert](https://www.rabbitmq.com/ssl.html#peer-verification-configuration) to `false` allows a TLS connection if client fails to provide a certificate.
- When setting [auth.tls.sslOptionsVerify](https://www.rabbitmq.com/ssl.html#peer-verification-configuration) to `verify_peer`, the node must perform peer verification. When set to `verify_none`, peer verification will be disabled and certificate exchange won't be performed.

**TLS integration with `cert-manager` (Let's Encrypt certificates)**

If using `cert-manager` to provision Let's Encrypt certificates, the `tls.crt` key in the generated TLS secret will contain the full certificate chain. Depending on the version of `cert-manager` in use, there can either be an empty `ca.crt` key, or none at all.

In order to instruct RabbitMQ to look for the CA certificate within the primary certificate, `auth.tls.existingSecretFullChain` can be set to `true`.

**Load Definitions**

It is possible to [load a RabbitMQ definitions file to configure RabbitMQ](http://www.rabbitmq.com/management.html#load-definitions). Because definitions may contain RabbitMQ credentials, [store the JSON as a Kubernetes secret](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod). Within the secret's data, choose a key name that corresponds with the desired load definitions filename (i.e. `load_definition.json`) and use the JSON object as the value. For example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: rabbitmq-load-definition
type: Opaque
stringData:
  load_definition.json: |-
    {
      "users": [
        {
          "name": "user",
          "password": "CHANGEME",
          "tags": "administrator"
        }
      ],
      "vhosts": [
        {
          "name": "/"
        }
      ]
    }
```

Then, specify the `load_definitions` property as an `extraConfiguration` pointing to the load definition file path within the container (i.e. `/app/load_definition.json`) and set `loadDefinition.enable` to `true`. Any load definitions specified will be available within in the container at `/app`.

> Loading a definition will take precedence over any configuration done through [Helm values](#parameters).

If needed, you can use `extraSecrets` to let the chart create the secret for you. This way, you don't need to manually create it before deploying a release. These secrets can also be templated to use supplied chart values. For example:

```yaml
extraSecrets:
  load-definition:
    load_definition.json: |
      {
        "users": [
          {
            "name": "{{ .Values.auth.username }}",
            "password": "{{ .Values.auth.password }}",
            "tags": "administrator"
          }
        ],
        "vhosts": [
          {
            "name": "/"
          }
        ]
      }
loadDefinition:
  enabled: true
  existingSecret: load-definition
extraConfiguration: |
  load_definitions = /app/load_definition.json
```

 **LDAP**

LDAP support can be enabled in the chart by specifying the `ldap.` parameters while creating a release. The following parameters should be configured to properly enable the LDAP support in the chart.

- `ldap.enabled`: Enable LDAP support. Defaults to `false`.
- `ldap.servers`: List of LDAP servers hostnames. No defaults.
- `ldap.port`: LDAP servers port. `389`.
- `ldap.user_dn_pattern`: Pattern used to translate the provided username into a value to be used for the LDAP bind. Defaults to `cn=${username},dc=example,dc=org`.
- `ldap.tls.enabled`: Enable TLS for LDAP connections. Defaults to `false`.

For example:

```console
ldap.enabled=true
ldap.serverss[0]="my-ldap-server"
ldap.port="389"
ldap.user_dn_pattern="cn=${username},dc=example,dc=org"
```

If `ldap.tls.enabled` is set to true, consider using `ldap.port=636` and checking the settings in the advancedConfiguration.

 **Memory high watermark**

It is possible to configure Memory high watermark on RabbitMQ to define [memory thresholds](https://www.rabbitmq.com/memory.html#threshold) using the `memoryHighWatermark.*` parameters. To do so, you have two alternatives:

A) Set an absolute limit of RAM to be used on each RabbitMQ node:

```console
memoryHighWatermark.enabled="true"
memoryHighWatermark.type="absolute"
memoryHighWatermark.value="512MB"
```

B) Set a relative limit of RAM to be used on each RabbitMQ node. To enable this feature, you must define the memory limits at POD level too:

```console
memoryHighWatermark.enabled="true"
memoryHighWatermark.type="relative"
memoryHighWatermark.value="0.4"
resources.limits.memory="2Gi"
```

**Adding extra environment variables**

In case you want to add extra environment variables (useful for advanced operations like custom init scripts), you can use the `extraEnvVars` property.

```yaml
extraEnvVars:
  - name: LOG_LEVEL
    value: error
```

Alternatively, you can use a ConfigMap or a Secret with the environment variables. To do so, use the `.extraEnvVarsCM` or the `extraEnvVarsSecret` properties.

**Plugins**

The Bitnami Docker RabbitMQ image ships a set of plugins by default. You can use the command below to obtain the whole list.

```bash
$ docker run --rm -it bitnami/rabbitmq -- ls /opt/bitnami/rabbitmq/plugins/
```

By default, this chart enables `rabbitmq_management` and `rabbitmq_peer_discovery_k8s` since they are required for RabbitMQ to work on K8s. To enable extra plugins, set the `extraPlugins` parameter with the list of plugins you want to enable.

In addition to this, you can also use the `communityPlugins` parameter to indicate a list of URLs separated by spaces where to download you custom plugins for RabbitMQ. For instance, use the parameters below to download a custom plugin during the container initialization and enable it:

```console
communityPlugins="http://some-public-url/my-custom-plugin-X.Y.Z.ez"
extraPlugins="my-custom-plugin"
```

 **Known issues**

- Changing the password through RabbitMQ's UI can make the pod fail due to the default liveness probes. If you do so, remember to make the chart aware of the new password. Updating the default secret with the password you set through RabbitMQ's UI will automatically recreate the pods. If you are using your own secret, you may have to manually recreate the pods.

**Persistence**

The [Bitnami RabbitMQ](https://github.com/bitnami/bitnami-docker-rabbitmq) image stores the RabbitMQ data and configurations at the `/opt/bitnami/rabbitmq/var/lib/rabbitmq/` path of the container.

The chart mounts a [Persistent Volume](http://kubernetes.io/docs/user-guide/persistent-volumes/) at this location. By default, the volume is created using dynamic volume provisioning. An existing PersistentVolumeClaim can also be defined.

 **Existing PersistentVolumeClaims**

1. Create the PersistentVolume
1. Create the PersistentVolumeClaim
1. Install the chart

```bash
$ helm install rabbitmq --set persistence.existingClaim=PVC_NAME bitnami/rabbitmq -n rabbitmq
```

 **Adjust permissions of the persistence volume mountpoint**

As the image runs as non-root by default, it is necessary to adjust the ownership of the persistent volume so that the container can write data into it.

By default, the chart is configured to use Kubernetes Security Context to automatically change the ownership of the volume. However, this feature does not work in all Kubernetes distributions.
As an alternative, this chart supports using an `initContainer` to change the ownership of the volume before mounting it in the final destination.

You can enable this `initContainer` by setting `volumePermissions.enabled` to `true`.

 **Configure the default user/vhost**

If you want to create default user/vhost and set the default permission. you can use `extraConfiguration`:

```yaml
auth:
  username: default-user
extraConfiguration: |-
  default_vhost = default-vhost
  default_permissions.configure = .*
  default_permissions.read = .*
  default_permissions.write = .*
```
