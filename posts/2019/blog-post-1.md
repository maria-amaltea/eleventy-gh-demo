---
layout: post-layout.njk
title: Kubernetes 1.24 - Introducing Non-Graceful Node Shutdown Alpha
date: 2019-05-30
tags: ['post']
---
<img src="{{ '/img/kubernetes.png' | url }}" />

<!-- Excerpt Start -->

Kubernetes v1.24 introduces alpha support for Non-Graceful Node Shutdown. This feature allows stateful workloads to failover to a different node after the original node is shutdown or in a non-recoverable state such as hardware failure or broken OS.

<!-- Excerpt End -->
 
You might have heard about the Graceful Node Shutdown capability of Kubernetes, and are wondering how the Non-Graceful Node Shutdown feature is different from that. Graceful Node Shutdown allows Kubernetes to detect when a node is shutting down cleanly, and handles that situation appropriately. A Node Shutdown can be "graceful" only if the node shutdown action can be detected by the kubelet ahead of the actual shutdown. However, there are cases where a node shutdown action may not be detected by the kubelet. This could happen either because the shutdown command does not trigger the systemd inhibitor locks mechanism that kubelet relies upon, or because of a configuration error (the ShutdownGracePeriod and ShutdownGracePeriodCriticalPods are not configured properly).

Graceful node shutdown relies on Linux-specific support. The kubelet does not watch for upcoming shutdowns on Windows nodes (this may change in a future Kubernetes release).

When a node is shutdown but without the kubelet detecting it, pods on that node also shut down ungracefully. For stateless apps, that's often not a problem (a ReplicaSet adds a new pod once the cluster detects that the affected node or pod has failed). For stateful apps, the story is more complicated. If you use a StatefulSet and have a pod from that StatefulSet on a node that fails uncleanly, that affected pod will be marked as terminating; the StatefulSet cannot create a replacement pod because the pod still exists in the cluster. As a result, the application running on the StatefulSet may be degraded or even offline. If the original, shut down node comes up again, the kubelet on that original node reports in, deletes the existing pods, and the control plane makes a replacement pod for that StatefulSet on a different running node. If the original node has failed and does not come up, those stateful pods would be stuck in a terminating status on that failed node indefinitely.


## Want your own blog?
The code for this blog is available on GitHub:
[https://github.com/JonUK/eleventy-blog](https://github.com/JonUK/eleventy-blog)
