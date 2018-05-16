# Open Grafana
```bash
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
```

# Visit dashboard
http://localhost:3000/dashboard/db/istio-dashboard

# Send traffic
```bash
curl http://$GATEWAY_URL/productpage
```