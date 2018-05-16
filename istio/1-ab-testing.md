# Enable default route
```bash
cat samples/bookinfo/kube/route-rule-all-v1.yaml
istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml
istioctl get routerules
```

# See that there is only one version routed

# User-based testing (A/B testing)
```bash
cat samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
istioctl create -f
samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
istioctl get routerules
```

# Connect with Jason on productpage

# Cleanup
```bash
istioctl delete -f samples/bookinfo/kube/route-rule-all-v1.yaml
istioctl delete -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
```