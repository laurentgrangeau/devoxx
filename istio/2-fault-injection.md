# Inject HTTP delay
```bash
cat samples/bookinfo/kube/route-rule-ratings-test-delay.yaml
istioctl create -f samples/bookinfo/kube/route-rule-ratings-test-delay.yaml
istioctl get routerule ratings-test-delay -o yaml
```

# Login with Jason and see 7s delay

# Inject HTTP abort
```bash
cat samples/bookinfo/kube/route-rule-ratings-test-abort.yaml
istioctl delete -f samples/bookinfo/kube/route-rule-ratings-test-delay.yaml
istioctl create -f samples/bookinfo/kube/route-rule-ratings-test-abort.yaml
istioctl get routerules ratings-test-abort -o yaml
```

# Login with Jason and see "product ratings not available"

# Cleanup
```bash
istioctl delete -f samples/bookinfo/kube/route-rule-all-v1.yaml
istioctl delete -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
istioctl delete -f samples/bookinfo/kube/route-rule-ratings-test-delay.yaml
istioctl delete -f samples/bookinfo/kube/route-rule-ratings-test-abort.yaml
```