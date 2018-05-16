# Mirorring
# Start 2 httpbin

# Deployment httpbinv1
```bash
cat <<EOF | istioctl kube-inject -f - | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: httpbin-v1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      containers:
      - image: docker.io/kennethreitz/httpbin
        imagePullPolicy: IfNotPresent
        name: httpbin
        command: ["gunicorn", "--access-logfile", "-", "-b", "0.0.0.0:8080", "httpbin:app"]
        ports:
        - containerPort: 8080
EOF
```

# Deployment httpbinv2
```bash
cat <<EOF | istioctl kube-inject -f - | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: httpbin-v2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: httpbin
        version: v2
    spec:
      containers:
      - image: docker.io/kennethreitz/httpbin
        imagePullPolicy: IfNotPresent
        name: httpbin
        command: ["gunicorn", "--access-logfile", "-", "-b", "0.0.0.0:8080", "httpbin:app"]
        ports:
        - containerPort: 8080
EOF
```

# Create httpbin service
```bash
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
spec:
  ports:
  - name: http
    port: 8080
  selector:
    app: httpbin
EOF
```

# Create sleep service (curl to provide load)
```bash
cat <<EOF | istioctl kube-inject -f - | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: sleep
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sleep
    spec:
      containers:
      - name: sleep
        image: tutum/curl
        command: ["/bin/sleep","infinity"]
        imagePullPolicy: IfNotPresent
EOF
```

# Create default route -> v1
```bash
cat <<EOF | istioctl create -f -
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: httpbin-default-v1
spec:
  destination:
    name: httpbin
  precedence: 5
  route:
  - labels:
      version: v1
EOF
```

# All traffic should go to v1
```bash
export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8080/headers'
```

# Check logs for v1 and v2 pods
# Only traffic for v1 pod
```bash
kubectl logs -f httpbin-v1-2113278084-98whj -c httpbin
```

# Create route to mirror traffic to v2
```bash
cat <<EOF | istioctl create -f -
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: mirror-traffic-to-httbin-v2
spec:
  destination:
    name: httpbin
  precedence: 11
  route:
  - labels:
      version: v1
    weight: 100
  - labels: 
      version: v2
    weight: 0
  mirror:
    name: httpbin
    labels:
      version: v2
EOF
```

# Send traffic
```bash
kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8080/headers'
```

# Check logs for v1 and v2
# We should see traffic for v1 and v2
```bash
kubectl logs -f httpbin-v1-2113278084-98whj -c httpbin
```

# Cleanup
```bash
istioctl delete routerule mirror-traffic-to-httbin-v2
istioctl delete routerule httpbin-default-v1
kubectl delete deploy httpbin-v1 httpbin-v2 sleep
kubectl delete svc httpbin
```