+++
title = "Predictkube"
availability = "v1.0+"
maintainer = "Dysnix"
description = "AI-based predictive scaling based on Prometheus metrics."
layout = "scaler"
go_file = "predictkube_scaler"
+++

### Trigger Specification

This specification describes the `predictkube` trigger that scales based on a predicting load based on a `prometheus` metrics.

```yaml
triggers:
- type: predictkube
  metadata:
    # Required fields:
    predictHorizon: "2h"
    historyTimeWindow: "7d"
    prometheusAddress: http://<prometheus-host>:9090
    metricName: http_requests_total
    query: sum(rate(http_requests_total{deployment="my-deployment"}[2m]))
    queryStep: "2m"
    threshold: '100'
```

**Parameter list:**

- `predictHorizon` - Prediction time interval. It is usually equal to the cool-down period of your application.
- `historyTimeWindow` - Time range for which to request metrics from Prometheus. We recomend to use minimum 7-14 days time window as historical data.
- `prometheusAddress` - Address of Prometheus server.
- `metricName` - Name to identify the Metric in the external.metrics.k8s.io API. If using more than one trigger it is required that all `metricName`(s) be unique.
- `query` - Predict query that will yield the value for the scaler to compare the `threshold` against. Query must return a vector/scalar single element response
- `queryStep` - The maximum time between two slices within the boundaries for QML range query, used in query.
- `threshold` - Value to start scaling for.

### Authentication Parameters

Predictkube Scaler supports one type of authentication - authentication by API key.

**Auth gateway based authentication:**

- `apiKey` - Api key previously issued for this tenant.

### Example

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: predictkube-secrets
  namespace: some-namespace
type: Opaque
data:
  apiKey: # Required: base64 encoded value of PredictKube apiKey
---
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  name: keda-trigger-auth-predictkube-secret
  namespace: some-namespace
spec:
  secretTargetRef:
    # Required: API key for your predictkube account
  - parameter: apiKey
    name: predictkube-secrets
    key: apiKey
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: predictkube-scaledobject
  namespace: some-namespace
spec:
  scaleTargetRef:
    name: my-deployment
    kind: StatefulSet
  pollingInterval: 30
  cooldownPeriod: 7200
  minReplicaCount: 3
  maxReplicaCount: 50
  triggers:
  - type: predictkube
    metadata:
      predictHorizon: "2h"
      historyTimeWindow: "7d"
      prometheusAddress: http://<prometheus-host>:9090
      metricName: http_requests_total # Note: name to identify the metric, generated value would be `prometheus-http_requests_total`
      query: sum(rate(http_requests_total{deployment="my-deployment"}[2m])) # Note: query must return a vector/scalar single element response
      queryStep: "2m" # Note: query step duration for range prometheus queries
      threshold: '100'
    authenticationRef:
      name: keda-trigger-auth-predictkube-secret
```
