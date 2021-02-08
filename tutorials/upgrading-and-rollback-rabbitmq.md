---
title: How to Upgrade and Rollback rabbitmq chart 
description: This tutorial explains how to Upgrade and Rollback rabbitmq helm chart
---



### Upgrading rabbitmq

It is also important to keep rabbitmq helm chart updated. 

To Upgrade rabbitmq helm chart,please execute following steps:

**Step 1:** To list all of your current releases in namespace "rabbitmq", run the following command :

```execute
helm list -n rabbitmq
```
You should get output similar to this:

Output:
```
 
```

You need to follow below steps.

**Step 2:** Update your Helm repositories with following command :

```execute
helm repo update 
```

You can expect the following output:

```output

```

**Step 3:** Check if there’s a newer version of the rabbitmq chart available using following command:

```execute
helm search repo rabbitmq --versions
```

You should see output similar to this:

```

```