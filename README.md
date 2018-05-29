<p align="center">
  <img height="300" src="https://sr.ht/2mc0.png">
</p>

<h1 align="center">tor-controller</h1>

[![Build Status](https://img.shields.io/travis-ci/kragniz/tor-controller.svg?style=flat-square)](https://travis-ci.org/kragniz/tor-controller)

Sprinkle some onions on your kubernetes clusters.

Quickstart
----------

Generate a test private key:

    openssl genrsa -out private_key 1024

Put your private key into a secret:

    kubectl create secret generic example-onion-key --from-file=private_key

Create an onion service:

```yaml
apiVersion: tor.k8s.io/v1alpha1
kind: OnionService
metadata:
  name: example-onion-service
spec:
  selector:
    app: httpd
  ports:
    - targetPort: 8080
      publicPort: 80
  privateKeySecret:
    name: example-onion-key
    key: private_key
```

List active OnionServices:

```
$ kubectl get onionservices -o=custom-columns=NAME:.metadata.name,HOSTNAME:.status.hostname
NAME                    HOSTNAME
example-onion-service   h7px2yyugjqkztrb.onion
```

tor-controller creates the following resources for each OnionService:

- a service, which is used to send traffic to application pods
- a configmap containing tor configuration pointing at the service
- tor daemon pod, which serves incoming traffic from the tor network

<p align="center">
  <img src="https://sr.ht/6WbX.png">
</p>

Using with nginx ingress
------------------------

tor-controller on its own simply directs TCP traffic to a backend service.
If you want to serve HTTP stuff, you'll probably want to pair it with nginx-ingress.

To do this, first install nginx-ingress normally. Then point an onion service
at the nginx-ingress-controller, for example:

```yaml
apiVersion: tor.k8s.io/v1alpha1
kind: OnionService
metadata:
  name: nginx-onion-service
spec:
  ports:
  - publicPort: 80
    targetPort: 80
    name: http
  selector:
    app: nginx-ingress-controller
    name: nginx-ingress-controller
  privateKeySecret:
    name: nginx-onion-key
    key: private_key
```
