---
title: Load balancing
weight: 2
---

**Table of contents**
{{< toc >}}

Load balancing is by default using Traefik.

Traefik has two modes for Kubernetes (both of which are enabled in k3s): CRD (Custom Resource Definition) and using k8s
default {{<newtablink "Ingress spec" "https://kubernetes.io/docs/concepts/services-networking/ingress/">}}.

## Middleware

Many things in Traefik is done using middleware, and it seems middleware can only be specified using the CRD mode.

And example of a middleware defined using CRD:
```yaml
# Strip prefix /foobar and /fiibar
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: test-stripprefix
spec:
  stripPrefix:
    prefixes:
      - /foobar
      - /fiibar
```

In order to use this middleware using standard Ingress kind, it should be specified with an annotation in the Ingress
yaml config file:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: foobar-ingress
  annotations:
    # This is the annotation that will enable the middleware for stripping prefix
    traefik.ingress.kubernetes.io/router.middlewares: default-test-stripprefix@kubernetescrd
spec:
  rules:
  - http:
      paths:
      - path: /foobar
        pathType: Prefix
        backend:
          service:
            name: foobar
            port:
              number: 80
```

### Relevant documentation

Link: https://doc.traefik.io/traefik/middlewares/overview/

There are examples for Kubernetes CRD under each middleware.

## TLS

TLS can be configured in the cluster using cert-manager, Let's Encrypt and Ingress rules.

Traefik has middleware for redirecting HTTP to HTTPS. See https://doc.traefik.io/traefik/middlewares/redirectscheme/
```yaml
# Redirect to https
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: test-redirectscheme
spec:
  redirectScheme:
    scheme: https
    permanent: true
```

### Cert-manager

Installation instruction here: https://cert-manager.io/docs/installation/kubernetes/

Follow the Installing with regular manifests if you don't want to use Helm. This is the easiest solution in my opinion.

After installing cert-manager, you need a ClusterIssuer. We'll be using Let's Encrypt.

Configuration guide here: https://cert-manager.io/docs/configuration/acme/

For k3s and traefik, the following should work for enabling LE:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: your@email.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-issuer
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: traefik
```

### Ingress configuration

Add following annotation to Ingress metadata: `cert-manager.io/cluster-issuer: letsencrypt-prod`

In order to force HTTPS, you need HTTPS redirect middleware:
```yaml
# Redirect to https
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-http
spec:
  redirectScheme:
    scheme: https
    permanent: true
```

Use this by deploying this middleware to the default namespace and use the annotation
`traefik.ingress.kubernetes.io/router.middlewares: default-redirect-http@kubernetescrd` in the Ingress metadata.