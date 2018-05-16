# Setting request timeout
```bash
istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml
```

# Route all to v2
```bash
cat <<EOF | istioctl replace -f -
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: reviews-default
spec:
  destination:
    name: reviews
  route:
  - labels:
      version: v2
EOF
```

# Add 2s delay
```bash
cat <<EOF | istioctl replace -f -
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: ratings-default
spec:
  destination:
    name: ratings
  route:
  - labels:
      version: v1
  httpFault:
    delay:
      percent: 100
      fixedDelay: 2s
EOF
```

# See 2s delay for rating

# Add 1s timeout (default 15s)
```bash
cat <<EOF | istioctl replace -f -
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: reviews-default
spec:
  destination:
    name: reviews
  route:
  - labels:
      version: v2
  httpReqTimeout:
    simpleTimeout:
      timeout: 1s
EOF
```

# See 1s delay but no reviews available

# Cleanup
```bash
istioctl delete -f samples/bookinfo/kube/route-rule-all-v1.yaml
```