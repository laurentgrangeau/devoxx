# Circuit breaking
```bash
kubectl apply -f <(istioctl kube-inject --debug -f samples/httpbin/httpbin.yaml)
cat samples/httpbin/routerules/httpbin-v1.yaml
istioctl create -f samples/httpbin/routerules/httpbin-v1.yaml
```

# Define maxConnections=1 and httpMaxPendingRequests=1
```bash
cat <<EOF | istioctl create -f -
apiVersion: config.istio.io/v1beta1
kind: DestinationPolicy
metadata:
  name: httpbin-circuit-breaker
spec:
  destination:
    name: httpbin
    labels:
      version: v1
  circuitBreaker:
    simpleCb:
      maxConnections: 1
      httpMaxPendingRequests: 1
      sleepWindow: 3m
      httpDetectionInterval: 1s
      httpMaxEjectionPercent: 100
      httpConsecutiveErrors: 1
      httpMaxRequestsPerConnection: 1
EOF
```

```bash
istioctl get destinationpolicy
```

# Deploy Fortio (stress test tool)
```bash
kubectl apply -f <(istioctl kube-inject --debug -f samples/httpbin/sample-client/fortio-deploy.yaml)
```

# Test Fortio
```bash
FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -curl  http://httpbin:8000/get
```

# See that it's working

# Break something (2 concurrent connections and 20 requests)
```bash
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```

# See that almost all requests made it

# Up to 3 concurrent connections
```bash
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```

# See that % respect circuit breaker settings

# See how much connections are flagged to circuit breaking
```bash
kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
```

# upstream_rq_pending_overflow are requests flag for circuit breaking

# Cleanup
```bash
istioctl delete routerule httpbin-default-v1
istioctl delete destinationpolicy httpbin-circuit-breaker
kubectl delete deploy httpbin fortio-deploy
kubectl delete svc httpbin
```