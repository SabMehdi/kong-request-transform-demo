# 🦍 Exposing an External REST Service via Kong Konnect (with SNI and request-transformer)

## 🤔 What is this?

This repo is a mini project that demonstrates how to proxy an **external HTTPS service** (like `httpbin.konghq.com`) via **Kong Konnect**, using:
- Kubernetes Ingress
- An `ExternalName` service
- The `request-transformer` plugin to fix **SNI issues**

## 🔥 The Problem

I needed to expose `httpbin.konghq.com/xml` through Kong, using a custom domain like: https://xml-proxy.example.com/xml


Easy, right? Except... it didn’t work. 😩

The request never made it to `httpbin.konghq.com`.

## 🕵️‍♂️ The Culprit: SNI

Kong was forwarding the request... but the TLS handshake was failing silently.

Why? Because `httpbin.konghq.com` is behind a shared IP and **relies on SNI** to know which SSL certificate to use. Without the proper `Host` header, the server doesn’t know who we’re trying to talk to — and it drops the connection.

## 🛠 The Fix: request-transformer

The solution was to **add the correct Host header** using the [`request-transformer`](https://docs.konghq.com/hub/kong-inc/request-transformer/) plugin.

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: set-upstream-host
config:
  replace:
    headers:
      - "host:httpbin.konghq.com"
plugin: request-transformer
```
## 📦 Deploy it Yourself
```
kubectl apply -f config.yaml
```

