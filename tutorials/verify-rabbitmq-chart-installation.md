### Verify Helm Chart Installation

Once the helm chart installation done you need to verify all the pods and services are up and running

Execute below command to check status of pods and services: 

### Check the pod status

```execute
kubectl get pods --namespace rabbitmq
```

You will see similar to this output:

```output
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

```output
NAME             READY   STATUS    RESTARTS   AGE
pod/rabbitmq-0   1/1     Running   0          59s

NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                 AGE
service/rabbitmq            ClusterIP   10.96.90.114   <none>        5672/TCP,4369/TCP,25672/TCP,15672/TCP   59s
service/rabbitmq-headless   ClusterIP   None           <none>        4369/TCP,5672/TCP,25672/TCP,15672/TCP   59s

NAME                        READY   AGE
statefulset.apps/rabbitmq   1/1     59s
```
 
**Port Forwading**
To access RabbitMq to the host, the pod's port(15672) must be exposed locally with the following command.

```execute
kubectl port-forward pod/rabbitmq-0 15672:15672 -n rabbitmq
```
 
Output will be similar to this
 
```output
[student@event-k8s-ibm-operators-playground-vbpdqc ~]$ kubectl port-forward pod/rabbitmq-0 15672:15672 --address 0.0.0.0 -n rabbitmq
Forwarding from 0.0.0.0:15672 -> 15672
Handling connection for 15672
Handling connection for 15672
Handling connection for 15672
.
.
.
```

Copy Kubernetes cluster IP address and portnumber and place that in browser and you should see RabbitMq UI. 
