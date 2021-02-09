<h1 align="center">RabbitMq</h1>


RabbitMQ is a messaging broker - an intermediary for messaging. It gives your applications a common platform to send and receive messages, and your messages a safe place to live until received.

RabbitMQ Cluster Kubernetes Operator is a Kubernetes operator that automates provisioning, management, and operations of RabbitMQ clusters running on Kubernetes.

Kubernetes Operators are software extensions to Kubernetes that provide custom resources for management of applications, services and their components.

In this and other Operator related guides, we use "Operator" (with a capital O) to refer to a Kubernetes Operator pattern implementation and "operator" (with a lowercase o) to refer to a technical operations engineer (administrator).

Kubernetes, with its API-centric approach, is an ideal operator to deploy RabbitMQ instances in a cluster. Kubernetes’ scaling capabilities and automated container deployment enhance RabbitMQ’s dependability in an extended cluster environment.

The Operator provides a consistent and easy way to deploy RabbitMQ clusters to Kubernetes and run them, including "day two" (continuous) operations. RabbitMQ clusters deployed using the operator can be used by applications running on Kubernetes or outside of Kubernetes.



**Prerequisites**

Access to a terminal window/ command line

A Kubernetes Cluster

The Kubernetes kubectl CLI tool


**Key Features**

Provisioning of single-node and multi-node RabbitMQ clusters

Automatic reconciliation of deployed clusters whenever their actual state does not match the expected state



**How RabbitMQ works on Kubernetes?**

The very purpose of Kubernetes is to orchestrate services and automate jobs in a highly distributed cluster. A messaging broker like RabbitMQ can enhance the quality of long-running tasks and stabilize vital background processes.

RabbitMQ uses the Advanced Message Queuing Protocol (AMQP) to standardize messaging between Producers, Brokers, and Consumers.

A Producer publishes a message to the Exchange.

The Exchange formats the message and forwards it to a predefined and selected Queue.

A Consumer then retrieves the formatted message and consumes it.

IMAGE:


The systematic distribution of messages improves communication between loosely coupled applications and services.


**RabbitMQ Cluster Reconciliation**

Deleted Secret objects will be recreated by the Kubernetes Operator but the newly generated secret value will not be deployed to the RabbitMQ cluster. For example, if the Secret with administrator credentials is deleted, a new Secret will be created with new username and password, but those will not be reflected in the RabbitMQ cluster. It works the same way for any Secret value.



**Monitoring**

The only thing left to make our cluster run unattended is adding some monitoring. We need to monitor both rabbit’s health and whether it’s properly clustered with the rest of nodes.

We have to remember the times when rabbitmqctl list_queues /rabbitmqctl list_channels was used as a method of monitoring, but this is not ideal, because it can’t distinguish between local and remote problems, and it creates significant network load. To that end, meet the new and shiny rabbitmqctl node_health_check – since 3.6.4 it’s the best way to check the health of any single RabbitMQ node.


**Checking whether a node is properly clustered requires several checks**


It should be clustered with the best node registered in the autocluster backend. This is the node that new nodes will attempt to join to, which is currently the first alive node in alphabetical order.

Even when the node is properly clustered with the discovery node, its data can still be diverged. Also this check is not transitive, so we need to check the partitions list both on the current node and on the discovery node.

All these clustering checks are implemented in separate commits and can be invoked using:

rabbitmqctl eval ‘autocluster:cluster_health_check_report().’

Using this rabbitmqctl command we can both detect any problem with our rabbit node and stop it immediately, so kubernetes will have a chance to do its restarting magic.

###Objective of tutorial

In this tutorial,we are going to cover following topics:

How to create RabbitMq Bitnami Helm Chart and verify its successful installation.

Verify status of pods and services.

Uninstall RabbitMq Helm Chart and release resources.




