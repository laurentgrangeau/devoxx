# Rate limit
```bash
cat samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
istioctl create -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
cat samples/bookinfo/kube/route-rule-reviews-v3.yaml
istioctl create -f samples/bookinfo/kube/route-rule-reviews-v3.yaml
```

# See that Jason go to v2 and everything go to v3

# Create ratelimit-handler.yaml
# memquota adapter with 5000rps
```yaml
apiVersion: config.istio.io/v1alpha2
kind: memquota
metadata:
  name: handler
  namespace: istio-system
spec:
  quotas:
  - name: requestcount.quota.istio-system
    # default rate limit is 5000qps
    maxAmount: 5000
    validDuration: 1s
    # The first matching override is applied.
    # A requestcount instance is checked against override dimensions.
    overrides:
    # The following override applies to traffic from 'rewiews' version v2,
    # destined for the ratings service. The destinationVersion dimension is ignored.
    - dimensions:
        destination: ratings
        source: reviews
        sourceVersion: v2
      maxAmount: 1
      validDuration: 1s
```

# Create rate limit
```bash
istioctl create -f ratelimit-handler.yaml
```

# Create ratelimit-rule.yaml
# Create a rule that apply quota
```yaml
apiVersion: config.istio.io/v1alpha2
kind: quota
metadata:
  name: requestcount
  namespace: istio-system
spec:
  dimensions:
    source: source.labels["app"] | source.service | "unknown"
    sourceVersion: source.labels["version"] | "unknown"
    destination: destination.labels["app"] | destination.service | "unknown"
    destinationVersion: destination.labels["version"] | "unknown"
---
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: quota
  namespace: istio-system
spec:
  actions:
  - handler: handler.memquota
    instances:
    - requestcount.quota
```

# Create rate limit
```bash
istioctl create -f ratelimit-rule.yaml
```

# Generate some loads
```bash
while true; do curl -s -o /dev/null http://$GATEWAY_URL/productpage; done
```

# Login in browser as Jason and Jason is limited at 1rps

# Cleanup
```bash
istioctl delete -f ratelimit-handler.yaml
istioctl delete -f ratelimit-rule.yaml
istioctl delete -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
istioctl delete -f samples/bookinfo/kube/route-rule-reviews-v3.yaml
```