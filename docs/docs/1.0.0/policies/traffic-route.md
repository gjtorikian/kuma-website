# Traffic Route

This policy allows us to configure routing rules for L4 traffic running in our [Mesh](../mesh). This policy provides support for weighted routing and can be used to implement versioning across our services as well as deployment strategies like blue/green and canary.

`TrafficRoute` must select the data plane proxies to route the connection between them.

### Default TrafficRoute

The control plane creates a default `TrafficRoute` every time the new `Mesh` is created. The default `TrafficRoute` enables the traffic between all the services in the mesh. 

:::: tabs :options="{ useUrlFragment: false }"
::: tab "Kubernetes"
```yaml
apiVersion: kuma.io/v1alpha1
kind: TrafficRoute
mesh: default
metadata:
  name: route-all-default
spec:
  sources:
    - match:
        kuma.io/service: '*'
  destinations:
    - match:
        kuma.io/service: '*'
  conf:
    split:
      - weight: 100
        destination:
          kuma.io/service: '*'
```
:::

::: tab "Universal"
```yaml
type: TrafficRoute
name: route-all-default
mesh: default
sources:
  - match:
      kuma.io/service: '*'
destinations:
  - match:
      kuma.io/service: '*'
conf:
  split:
    - weight: 100
      destination:
        kuma.io/service: '*'
```
:::
::::

### Usage

By default when a service makes a request to another service, Kuma will round robin the request across every data plane proxy belogning to the destination service. It is possible to change this behavior by using this policy, for example:

:::: tabs :options="{ useUrlFragment: false }"
::: tab "Kubernetes"
```yaml
apiVersion: kuma.io/v1alpha1
kind: TrafficRoute
mesh: default
metadata:
  name: route-example
spec:
  sources:
    - match:
        kuma.io/service: backend_default_svc_80
  destinations:
    - match:
        kuma.io/service: redis_default_svc_6379
  conf:
    split:
      - weight: 90
        destination:
          kuma.io/service: redis_default_svc_6379
          version: '1.0'
      - weight: 10
        destination:
          kuma.io/service: redis_default_svc_6379
          version: '2.0'
```

We will apply the configuration with `kubectl apply -f [..]`.
:::

::: tab "Universal"
```yaml
type: TrafficRoute
name: route-example
mesh: default
sources:
  - match:
      kuma.io/service: backend
destinations:
  - match:
      kuma.io/service: redis
conf:
  split:
    - weight: 90
      destination:
        kuma.io/service: redis
        version: '1.0'
    - weight: 10
      destination:
        kuma.io/service: redis
        version: '2.0'
```

We will apply the configuration with `kumactl apply -f [..]` or via the [HTTP API](/docs/1.0.0/documentation/http-api).
:::
::::

In this example the `TrafficRoute` policy assigns a positive weight of `90` to the version `1.0` of the `redis` service and a positive weight of `10` to the version `2.0` of the `redis` service. 

:::tip
Note that routing can be applied not just on the automatically provisioned `service` Kuma tag, but on any other tag that we may want to add to our data plane proxies (like `version` in the example above).
:::

Kuma utilizes positive weights in the `TrafficRoute` policy and not percentages, therefore Kuma does not check if the total adds up to 100. If we want to stop sending traffic to a destination service we change the `weight` for that service to 0.
