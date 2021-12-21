+++
title = "Predictkube"
availability = "v1.0+"
maintainer = "Dysnix"
description = "Scale applications based on PredictKube forecasting service."
layout = "scaler"
go_file = "predictkube_scaler"
+++

### Trigger Specification

This specification describes the `predictkube` trigger that scales based on a predicting load based on a given metric.

```yaml
triggers:
- type: predictkube
  metadata:
    # Required fields:
    predictHorizon: "2h"
    historyTimeWindow: "7d"
    prometheusAddress: http://<prometheus-host>:9090
    metricName: http_requests_total # Note: name to identify the metric, generated value would be `prometheus-http_requests_total`
    query: sum(rate(http_requests_total{deployment="my-deployment"}[2m])) # Note: query must return a vector/scalar single element response
    queryStep: "2m" # Note: query step duration for range prometheus queries
    threshold: '100'
```

**Parameter list:**

- `predictHorizon` - Prediction time interval.
- `historyTimeWindow` - Time range for which to request metrics from Prometheus.
- `prometheusAddress` - Address of Prometheus server.
- `metricName` - Name to identify the Metric in the external.metrics.k8s.io API. If using more than one trigger it is required that all `metricName`(s) be unique.
- `query` - Predict query that will yield the value for the scaler to compare the `threshold` against.
- `queryStep` - The maximum time between two slices within the boundaries for QML range query.
- `threshold` - Value to start scaling for.

### Authentication Parameters

Predictkube Scaler supports one type of authentication - authentication with auth gateway.

**Auth gateway based authentication:**

- `apiKey` - Api key previously issued for this tennant.

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
  apiKey: ZXlKaGJHY2lPaUpJVXpJMU5pSXNJblI1Y0NJNklrcFhWQ0o5LmV5SmhkV1FpT2lKMFpYTjBJRU55WldGMFpVTnNhV1Z1ZENJc0ltVjRjQ0k2TVRZME5qa3hOekkzTnl3aWMzVmlJam9pT0RNNE5qWTVPREF0TTJVek5TMHhNV1ZqTFRsbU1qUXRZV05rWlRRNE1EQXhNVEl5SW4wLjVRRXVPNl95c2RrMmFiR3ZrM1hwN1EyNU00SDRwSUZYZXFQMkU3bjlyS0k=
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
