# Metrics, logs and trace
# Connect to Zipkin
```bash
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=zipkin -o jsonpath='{.items[0].metadata.name}') 9411:9411 &
```

# Generate trace for Zipkin
# Go to productpage
# Go to Zipkin