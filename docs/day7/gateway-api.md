# Gateway API in Kubernetes

This tutorial delves into the Gateway API in Kubernetes, explaining its purpose, structure, and how to set it up. We’ll explore various aspects of the Gateway API, providing examples with YAML manifests and output commands to illustrate the concepts clearly.

## Table of Contents
1. [Introduction to Gateway API](#introduction-to-gateway-api)
2. [Gateway API Resource Model](#gateway-api-resource-model)
   - [GatewayClass](#gatewayclass)
   - [Gateway](#gateway)
   - [Routes](#routes)
3. [Modularity and Box Within a Box Concept](#modularity-and-box-within-a-box-concept)
   - [Team-Based Management](#team-based-management)
4. [Steps to Set Up a Gateway API in Kubernetes](#steps-to-set-up-a-gateway-api-in-kubernetes)
   - [Creating and Configuring GatewayClasses](#creating-and-configuring-gatewayclasses)
   - [Setting Up Gateways](#setting-up-gateways)
   - [Setting Up Routes](#setting-up-routes)
5. [Managing and Troubleshooting Gateways](#managing-and-troubleshooting-gateways)
6. [Conclusion](#conclusion)
7. [References](#references)

---

### Introduction to Gateway API

The Gateway API is a set of resources that model service networking in Kubernetes. It provides a more expressive and extensible way to define how traffic enters and leaves the Kubernetes cluster. Unlike traditional Ingress resources, the Gateway API introduces a resource model that separates infrastructure concerns from application-level routing.

[Back to TOC](#table-of-contents)

### Gateway API Resource Model

The Gateway API resource model is built around three key components:

#### GatewayClass

The `GatewayClass` defines a set of gateways with a common configuration and behavior. It represents the top-level configuration, specifying how gateways are instantiated and managed. This resource allows infrastructure providers to offer different classes of service, each tailored to specific networking needs.

#### Gateway

A `Gateway` represents an entry point for network traffic into the Kubernetes cluster. It is configured according to the specifications defined by the `GatewayClass`. The `Gateway` essentially requests a point where traffic can be translated to Services within the cluster. It can have various listeners that define how traffic is accepted and routed.

**Important Note:** For a `Route` to attach to a `Gateway`, at least one listener on the `Gateway` must explicitly allow this attachment. This means the listener must satisfy any conditions set by the `Route` for attachment, such as matching hostnames, namespaces, or kinds.

#### Routes

`Routes` describe how traffic coming via the Gateway maps to the Services. They define the rules for routing traffic to Kubernetes services based on various criteria like paths, headers, or other attributes. Routes can include `HTTPRoute`, `TCPRoute`, `TLSRoute`, and others, depending on the protocol and requirements.

[Back to TOC](#table-of-contents)

### Modularity and Box Within a Box Concept

The Gateway API in Kubernetes can be understood through the concept of "box within a box." This analogy helps to illustrate how the different components like `GatewayClass`, `Gateway`, and `HTTPRoute` are designed to be modular and can be managed independently.

1. **GatewayClass as the Outer Box**: The `GatewayClass` represents the top-level configuration, defining how Gateways are instantiated and managed. It serves as the blueprint for creating Gateways, specifying parameters and behaviors that apply to all Gateways using this class.

2. **Gateway as the Inner Box**: Within the `GatewayClass`, you have individual `Gateways`. Each `Gateway` is a concrete instance that represents an entry point for network traffic. It inherits the configuration from the `GatewayClass` but can have its own specific settings, such as listeners and routing policies.

3. **HTTPRoute as the Innermost Box**: Finally, within a `Gateway`, you have `HTTPRoute` (or other types of routes like `TCPRoute`). These routes define how traffic that enters the Gateway is directed to different services within the cluster. The `HTTPRoute` specifies granular routing rules based on paths, headers, or other criteria.

#### Team-Based Management

Because of this modular structure, different teams can manage each component independently:

- **Infrastructure Provider Team**: Responsible for managing `GatewayClass`, ensuring that the infrastructure layer is consistent and aligned with the organization’s networking policies.
- **Platform or Cluster Operations Team**: Manages the `Gateways`, ensuring they are correctly configured to route traffic appropriately, while also monitoring and scaling them as needed.
- **Application Development Team**: Manages `HTTPRoute` configurations, focusing on how traffic is routed to specific applications or services. This allows for granular control over traffic management without needing to understand the underlying infrastructure details.

[Back to TOC](#table-of-contents)

### Steps to Set Up a Gateway API in Kubernetes

This section will guide you through the practical steps involved in setting up a Gateway API in Kubernetes. We will cover the creation and configuration of `GatewayClass`, setting up `Gateways`, and defining `Routes`.

#### Creating and Configuring GatewayClasses

Let’s begin by creating a `GatewayClass`. The `GatewayClass` is the foundation of the infrastructure layer, defining the behavior of Gateways created under it.

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GatewayClass
metadata:
  name: my-gateway-class
spec:
  controllerName: example.com/gateway-controller
  description: "This GatewayClass defines the base infrastructure for my Gateway."
```

This `GatewayClass` specifies that any `Gateway` created using this class will be managed by the `example.com/gateway-controller`.

[Back to TOC](#table-of-contents)

#### Setting Up Gateways

Once the `GatewayClass` is defined, we can create a `Gateway`. The `Gateway` sits at the infrastructure layer and acts as the entry point for incoming traffic.

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: Gateway
metadata:
  name: my-gateway
  namespace: gateway-api-example-ns1
spec:
  gatewayClassName: my-gateway-class
  listeners:
  - name: web-listener
    protocol: HTTP
    port: 80
    allowedRoutes:
      kinds:
      - kind: HTTPRoute
      namespaces:
        from: Selector
        selector:
          matchLabels:
            expose-apps: "true"
```

In this configuration, the `Gateway` listens on port 80 for HTTP traffic. It allows `HTTPRoute` resources from namespaces with the label `expose-apps: "true"` to attach to this listener. This setup ensures that only Routes from specific namespaces can attach to this Gateway, adding an additional layer of control.

[Back to TOC](#table-of-contents)

#### Setting Up Routes

Routes are defined at the application layer. Here’s an example of an `HTTPRoute` that directs traffic based on specific rules.

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: HTTPRoute
metadata:
  name: my-http-route
  namespace: gateway-api-example-ns2
spec:
  parentRefs:
  - name: my-gateway
    namespace: gateway-api-example-ns1
  hostnames:
  - "example.com"
  rules:
  - matches:
    - path:
        type: Prefix
        value: "/"
    forwardTo:
    - serviceName: my-service
      port: 80
```

This `HTTPRoute` configuration directs traffic from `example.com` to the `my-service` service on port 80, ensuring that requests to the root path (`/`) are handled correctly. Additionally, since the `my-http-route` is in a different namespace (`gateway-api-example-ns2`) from the `Gateway`, it relies on the selector specified in the `Gateway` listener to be allowed to attach.

**Key Points:**
- The `parentRefs` field specifies the `Gateway` this `HTTPRoute` wants to attach to.
- The listener in the `Gateway` must allow attachments from the namespace of this `HTTPRoute` for the Route to be successfully attached.

[Back to TOC](#table-of-contents)

### Managing and Troubleshooting Gateways

Managing Gateways involves common Kubernetes operations like creating, listing, describing, and deleting Gateways. Below are the steps along with the expected outputs:

#### Creating a Gateway

To create a Gateway, use the following command:

```sh
kubectl apply -f gateway.yaml
```

**Output:**

```sh
gateway.networking.k8s.io/my-gateway created
```

This output confirms that the Gateway has been successfully created.

[Back to TOC](#table-of-contents)

#### Listing All Gateways

To see all the Gateways in your cluster:

```sh
kubectl get gateways
```

**Output:**

```sh
NAME          CLASS               ADDRESS   READY   AGE
my-gateway    my-gateway-class    10.0.0.1  True    5m
```

This output lists the Gateway with its class, assigned address, readiness state, and age.

[Back to TOC](#table-of-contents)

#### Describing a Gateway

For detailed information about a specific Gateway:

```sh
kubectl describe gateway my-gateway
```

**Output:**

```sh
Name:         my-gateway
Namespace:    gateway-api-example-ns1
Address:      10.0.0.1
Listeners:
  - Name: web-listener


    Port: 80
    Protocol: HTTP
Routes:
  - Kind: HTTPRoute
    Selector: expose-apps=true
```

This command provides a detailed description of the Gateway, including its listeners and associated routes.

[Back to TOC](#table-of-contents)

#### Deleting a Gateway

To remove a Gateway:

```sh
kubectl delete gateway my-gateway
```

**Output:**

```sh
gateway.networking.k8s.io "my-gateway" deleted
```

This output confirms that the Gateway has been successfully deleted.

[Back to TOC](#table-of-contents)

#### Troubleshooting Tips

When managing Gateways, you may encounter issues. Here are some common troubleshooting tips:

- **Check Gateway Status:** Ensure the Gateway is in a `Ready` state. If not, inspect the `GatewayClass` and `Listeners`.
- **Verify Routes:** Ensure your `HTTPRoute` or other route types are correctly linked to the Gateway and have valid configurations. Also, ensure that the Gateway's listener explicitly allows the attachment of the Routes.
- **Logs:** Check the logs of your Gateway controller for any errors or warnings.

[Back to TOC](#table-of-contents)

### Conclusion

The Gateway API provides a powerful, layered approach to managing ingress traffic in Kubernetes. By separating infrastructure concerns from application-level routing, it allows for more flexible and scalable networking configurations. This tutorial walked you through setting up and managing Gateways, Routes, and understanding the layered structure that makes the Gateway API so versatile.

[Back to TOC](#table-of-contents)

### References

[Gateway API Documentation](https://gateway-api.sigs.k8s.io/concepts/api-overview/)

[Back to TOC](#table-of-contents)
