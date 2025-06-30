# DenyIP

DenyIP is a middleware plugin for [Traefik](https://github.com/traefik/traefik) which accepts IP addresses or IP address ranges and blocks requests originating from those IPs. Supports both IPv4 and IPv6 addresses.

## Configuration

### Static

In the example below `fowardedHeaders.insecure` is enabled in order to allow the IP address to be available from proxied requests. In a production environment, you may want to consider using [`forwardedHeaders.trustedIPs`](https://docs.traefik.io/routing/entrypoints/#forwarded-headers)

```yaml
experimental:
  pilot:
    token: "xxxxx"
  plugins:
    denyip:
      modulename = "github.com/developmentaid-org/denyIP"
      version = "v1.0.0"

entryPoints:
  http:
    address: ":80"
    forwardedHeaders:
      insecure: true
```

### Dynamic

To configure the `DenyIP` plugin you should create a [middleware](https://docs.traefik.io/middlewares/overview/) in your dynamic configuration as explained [here](https://docs.traefik.io/middlewares/overview/). The following example creates and uses the `denyip` middleware plugin to deny all requests originating from the configured `ipDenyList` array. `ipDenyList` will accept:
- IPv4 addresses (e.g., `127.0.0.1`)
- IPv4 CIDR ranges (e.g., `192.168.0.0/24`)
- IPv6 addresses (e.g., `2001:db8::1`)
- IPv6 CIDR ranges (e.g., `2001:db8::/32`)

> Note: Providing invalid ip addresses or ranges in `ipDenyList` will cause an error and the plugin will not load.

```yaml
http:
  # Add the router
  routers:
    my-router:
      entryPoints:
      - http
      middlewares:
      - denyip
      service: service-foo
      rule: Path(`/foo`)

  # Add the middleware
  middlewares:
    denyip:
      plugin:
        ipDenyList:
          # IPv4 examples
          - 24.0.0.0/12
          - 127.0.0.1
          # IPv6 examples
          - 2001:db8::/32
          - 2001:db8::1
          # ... rest of your deny list ...

  # Add the service
  services:
    service-foo:
      loadBalancer:
        servers:
        - url: http://localhost:5000/
```