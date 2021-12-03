# introspection-proxy

This Helm chart will set up an NGINX `location` to proxy token introspection requests for each of the provided `hosts`, through the use of the `nginx.ingress.kubernetes.io/server-snippet` Ingress annotation.

The primary purpose of this is to enable the usage of the [Curity NGINX Phantom Token Module](https://github.com/curityio/nginx_phantom_token_module).

## Notes
Since the Ingress object requires a `path` to be defined for each rule, this chart will also set up a dummy Service and path in order to successfully update the nginx configuration.