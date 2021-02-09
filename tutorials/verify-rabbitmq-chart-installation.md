---
title: Verify rabbitmq Chart Installation
description: This tutorial explains how to verify that rabbitmq chart installed successfully
--


**To Create Namespace**

```execute
kubectl create ns rabbitmq
```

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-rabbitmq
   namespace: rabbitmq
   annotations:
     volume.beta.kubernetes.io/storage-class: "default"
 spec:
   accessModes:
     - ReadWriteOnce
   resources:
     requests:
       storage: 10Gi
 EOF
 
```

persistentvolumeclaim/data-rabbitmq created


```
helm install rabbitmq --set auth.username=admin,auth.password=passw0rd123,auth.erlangCookie=secretcookie,persistence.existingClaim=data-rabbitmq bitnami/rabbitmq -n rabbitmq
```

Your Output will look similar to this 

```
NAME: rabbitmq
LAST DEPLOYED: Wed Feb  3 14:49:03 2021
NAMESPACE: rabbitmq
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Credentials:

    echo "Username      : admin"
    echo "Password      : $(kubectl get secret --namespace rabbitmq rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)"
    echo "ErLang Cookie : $(kubectl get secret --namespace rabbitmq rabbitmq -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)"

RabbitMQ can be accessed within the cluster on port  at rabbitmq.rabbitmq.svc.

To access for outside the cluster, perform the following steps:

To Access the RabbitMQ AMQP port:

    echo "URL : amqp://127.0.0.1:5672/"
    kubectl port-forward --namespace rabbitmq svc/rabbitmq 5672:5672

To Access the RabbitMQ Management interface:

    echo "URL : http://127.0.0.1:15672/"
    kubectl port-forward --namespace rabbitmq svc/rabbitmq 15672:15672
    
```

**Once the helm chart installation done you need to verify all the pods and services are up and running**

Execute below command to check status of pods and services: 

### Check the pod status


```execute
kubectl get pods --namespace rabbitmq
```

You will see similar to this output:

```
NAME         READY   STATUS    RESTARTS   AGE
rabbitmq-0   1/1     Running   0          36s
```

Once the `rabbitmq` PODs are up and running , and `READY` states are `1/1`, your rabbitmq is ready to use.

### Check all the Kubernetes resources status

You can run the following command to know status of all the deployed resources inside the namespace `rabbitmq`


```execute
kubectl get all --namespace rabbitmq
```

All the deployment and service status should be Running.

```
NAME             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-0   1/1     Running   0          59s
```

```
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                 AGE
service/rabbitmq            ClusterIP   10.96.90.114   <none>        5672/TCP,4369/TCP,25672/TCP,15672/TCP   59s
service/rabbitmq-headless   ClusterIP   None           <none>        4369/TCP,5672/TCP,25672/TCP,15672/TCP   59s
```

```
NAME                        READY   AGE
statefulset.apps/rabbitmq   1/1     59s
```

**Describe Pods**

```execute
Name:         rabbitmq-0
Namespace:    rabbitmq
Priority:     0
Node:         event-k8s-ibm-operators-playground-vbpdqc.softlayer.com/169.62.52.103
Start Time:   Wed, 03 Feb 2021 14:49:07 -0600
Labels:       app.kubernetes.io/instance=rabbitmq
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=rabbitmq
              controller-revision-hash=rabbitmq-6756f58886
              helm.sh/chart=rabbitmq-8.9.2
              statefulset.kubernetes.io/pod-name=rabbitmq-0
Annotations:  checksum/config: 42b91b8ceee60ad495d08e67d43874e7fbb4d9aa931ca6dd85f4a41cb264d3d9
              checksum/secret: 8c09a95b04629ea3466fb64a0768a780ba03b376b64afb4f9cc7fb4d271caebe
Status:       Running
IP:           10.244.0.46
IPs:
  IP:           10.244.0.46
Controlled By:  StatefulSet/rabbitmq
Containers:
  rabbitmq:
    Container ID:   docker://fd368aa42476b1d525c962ce5e9e82029f3a6b03d195936e006e58cffd8cc6fc
    Image:          docker.io/bitnami/rabbitmq:3.8.11-debian-10-r0
    Image ID:       docker-pullable://bitnami/rabbitmq@sha256:ae4e7ab2049ed38b5165820a6235453b09faf633956c7ff5cfeffe330efaacab
    Ports:          5672/TCP, 25672/TCP, 15672/TCP, 4369/TCP
    Host Ports:     0/TCP, 0/TCP, 0/TCP, 0/TCP
    State:          Running
      Started:      Wed, 03 Feb 2021 14:49:08 -0600
    Ready:          True
    Restart Count:  0
    Liveness:       exec [/bin/bash -ec rabbitmq-diagnostics -q ping] delay=120s timeout=20s period=30s #success=1 #failure=6
    Readiness:      exec [/bin/bash -ec rabbitmq-diagnostics -q check_running && rabbitmq-diagnostics -q check_local_alarms] delay=10s timeout=20s period=30s #success=1 #failure=3
    Environment:
      BITNAMI_DEBUG:            false
      MY_POD_IP:                 (v1:status.podIP)
      MY_POD_NAME:              rabbitmq-0 (v1:metadata.name)
      MY_POD_NAMESPACE:         rabbitmq (v1:metadata.namespace)
      K8S_SERVICE_NAME:         rabbitmq-headless
      K8S_ADDRESS_TYPE:         hostname
      RABBITMQ_FORCE_BOOT:      no
      RABBITMQ_NODE_NAME:       rabbit@$(MY_POD_NAME).$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE).svc.cluster.local
      K8S_HOSTNAME_SUFFIX:      .$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE).svc.cluster.local
      RABBITMQ_MNESIA_DIR:      /bitnami/rabbitmq/mnesia/$(RABBITMQ_NODE_NAME)
      RABBITMQ_LDAP_ENABLE:     no
      RABBITMQ_LOGS:            -
      RABBITMQ_ULIMIT_NOFILES:  65536
      RABBITMQ_USE_LONGNAME:    true
      RABBITMQ_ERL_COOKIE:      <set to the key 'rabbitmq-erlang-cookie' in secret 'rabbitmq'>  Optional: false
      RABBITMQ_USERNAME:        admin
      RABBITMQ_PASSWORD:        <set to the key 'rabbitmq-password' in secret 'rabbitmq'>  Optional: false
      RABBITMQ_PLUGINS:         rabbitmq_management, rabbitmq_peer_discovery_k8s, rabbitmq_auth_backend_ldap
    Mounts:
      /bitnami/rabbitmq/conf from configuration (rw)
      /bitnami/rabbitmq/mnesia from data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from rabbitmq-token-jhqqs (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  configuration:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      rabbitmq-config
    Optional:  false
  data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  data-rabbitmq
    ReadOnly:   false
  rabbitmq-token-jhqqs:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  rabbitmq-token-jhqqs
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                                                              Message
  ----    ------     ----  ----                                                              -------
  Normal  Scheduled  115s  default-scheduler                                                 Successfully assigned rabbitmq/rabbitmq-0 to event-k8s-ibm-operators-playground-vbpdqc.softlayer.com
  Normal  Pulled     114s  kubelet, event-k8s-ibm-operators-playground-vbpdqc.softlayer.com  Container image "docker.io/bitnami/rabbitmq:3.8.11-debian-10-r0" already present on machine
  Normal  Created    114s  kubelet, event-k8s-ibm-operators-playground-vbpdqc.softlayer.com  Created container rabbitmq
  Normal  Started    114s  kubelet, event-k8s-ibm-operators-playground-vbpdqc.softlayer.com  Started container rabbitmq
  
  ```
  
  
 **Port Forwading**
 
 ```
 kubectl port-forward pod/rabbitmq-0 15672:15672 -n rabbitmq
 ```
 
 Output will be similar to this
 
 ```
 Forwarding from 127.0.0.1:15672 -> 15672
Forwarding from [::1]:15672 -> 15672
^C[student@event-k8s-ibm-operators-playground-vbpdqc ~]$ kubectl port-forward pod/rabbitmq-0 0.0.0.0/15672:15672 -n rabbitmq
error: error parsing local port '0.0.0.0/15672': strconv.ParseUint: parsing "0.0.0.0/15672": invalid syntax
[student@event-k8s-ibm-operators-playground-vbpdqc ~]$ kubectl port-forward pod/rabbitmq-0 15672:15672 --address 0.0.0.0 -n rabbitmq
Forwarding from 0.0.0.0:15672 -> 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672

```

Copy Kubernetes cluster IP address and portnumber and place that in browser and you should see RabbitMq UI. 


