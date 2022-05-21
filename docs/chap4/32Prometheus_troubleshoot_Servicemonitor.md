# **第六节 Troubleshooting ServiceMonitor changes**

When creating/deleting/modifying `ServiceMonitor` objects it is sometimes not as obvious what piece is not working properly. 

This section gives a step by step guide how to troubleshoot such actions on a `ServiceMonitor` object.

## **1 Overview of ServiceMonitor tagging and related elements**

A common problem related to `ServiceMonitor` identification by Prometheus is related to an incorrect **tagging**, that does not match the `Prometheus` custom resource definition scope, or lack of permission for the Prometheus `ServiceAccount` to *get, list, watch* `Services` and `Endpoints` from the target application being monitored. 

As a general guideline consider the diagram below, giving an example of a `Deployment` and `Service` called `my-app`, being monitored by Prometheus based on a `ServiceMonitor` named `my-service-monitor`:

![Alt Image Text](../images/32_1.png "Headline image")

### **Note:** 

* The `ServiceMonitor` references a `Service` (not a `Deployment`, or a `Pod`), by **labels** and by the port name in the `Service`. 
* This **port name** is optional in Kubernetes, but must be specified for the `ServiceMonitor` to work. 
* It is not the same as the port name on the `Pod` or `container`, although it can be.

## **2 Has my ServiceMonitor been picked up by Prometheus?**


**`ServiceMonitor` objects are selected by the `serviceMonitorSelector` of a Prometheus object.**

**The name of a `ServiceMonitor` is encoded in the Prometheus configuration, so you can simply grep whether it is present there.** 

**The configuration generated by the Prometheus Operator is stored in a Kubernetes `Secret`,** named after the Prometheus object name prefixed with `prometheus-` and is located in the same namespace as the Prometheus object. 

For example for a Prometheus object called k8s one can find out if the ServiceMonitor named `elasticsearch` has been picked up with:

```
kubectl -n monitoring get secret prometheus-k8s -ojson | jq -r '.data["prometheus.yaml.gz"]' | base64 -d | gunzip | grep "elasticsearch"

- job_name: default/elasticsearch/0
    regex: elasticsearch-exporter
```