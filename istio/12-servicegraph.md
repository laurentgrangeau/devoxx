# Send some traffic
```bash
curl http://$GATEWAY_URL/productpage
```

# Open servicegraph
```bash
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}') 8088:8088 &
```

# Visit dashboard
http://localhost:8088/force/forcegraph.html

# Filter servicegraph
http://localhost:8088/force/forcegraph.html?time_horizon=15s&filter_empty=true