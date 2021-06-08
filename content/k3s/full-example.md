---
title: Full example
weight: 100
---

## Full example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  minReadySeconds: 10
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    kubernetes.io/ingress.allow-http: "false"
    traefik.ingress.kubernetes.io/router.middlewares: default-redirect-http@kubernetescrd
spec:
  ingressClassName: traefik-lb
  tls:
    - secretName: nginx-tls
      hosts:
        - nginx.tychsen.me
  rules:
    - host: nginx.tychsen.me
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: nginx-stripprefix
spec:
  stripPrefix:
    prefixes:
      - /nginx
---
# Redirect to https
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-http
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx2
  annotations:
    traefik.ingress.kubernetes.io/router.middlewares: default-nginx-stripprefix@kubernetescrd
spec:
  ingressClassName: traefik-lb
  rules:
    - http:
        paths:
          - path: /nginx
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 80
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: nginx
```
