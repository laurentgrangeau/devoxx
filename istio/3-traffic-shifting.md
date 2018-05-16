# Traffic shifting
```bash
istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml
```

# 50% red star
```bash
cat samples/bookinfo/kube/route-rule-reviews-50-v3.yaml
istioctl replace -f samples/bookinfo/kube/route-rule-reviews-50-v3.yaml
istioctl get routerule reviews-default -o yaml
```

# See that 50% red star

# 100% red star
```bash
cat samples/bookinfo/kube/route-rule-reviews-v3.yaml
istioctl replace -f samples/bookinfo/kube/route-rule-reviews-v3.yaml
```

# See that 100% red star

# Cleanup
```bash
istioctl delete -f samples/bookinfo/kube/route-rule-all-v1.yaml
```