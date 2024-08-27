
# Understanding `forwardTo` and `backendRefs` in Gateway API's HTTPRoute

## Table of Contents

1. [Introduction](#introduction)
2. [What is `forwardTo`?](#what-is-forwardto)
   - [Usage Example](#usage-example-forwardto)
3. [What is `backendRefs`?](#what-is-backendrefs)
   - [Usage Example](#usage-example-backendrefs)
4. [Differences Between `forwardTo` and `backendRefs`](#differences-between-forwardto-and-backendrefs)
5. [When to Use `forwardTo` vs `backendRefs`](#when-to-use-forwardto-vs-backendrefs)
6. [Conclusion](#conclusion)

---

## 1. Introduction
The Gateway API in Kubernetes provides a more powerful and flexible way to route traffic compared to traditional Ingress resources. Two key fields in the `HTTPRoute` resource are `forwardTo` and `backendRefs`. This tutorial will explore these fields, their differences, and when to use each.

[Back to TOC](#table-of-contents)

---

## 2. What is `forwardTo`?
The `forwardTo` field in the `HTTPRoute` resource is used to define the backend services to which traffic should be routed. It’s a more general approach to specifying backends and is often used when you don’t need the advanced features provided by `backendRefs`.

### Usage Example: `forwardTo`
Here’s an example of how you might configure `forwardTo` in an `HTTPRoute`:

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: HTTPRoute
metadata:
  name: example-route
spec:
  parentRefs:
  - name: example-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: "/"
    forwardTo:
    - serviceName: backend1
      port: 80
      weight: 80
    - serviceName: backend2
      port: 80
      weight: 20
```

In this example, traffic matching the path `/` is forwarded to `backend1` and `backend2`, with 80% of the traffic going to `backend1` and 20% to `backend2`.

[Back to TOC](#table-of-contents)

---

## 3. What is `backendRefs`?
The `backendRefs` field in the `HTTPRoute` resource offers a more powerful way to define backend references. It provides more control and flexibility, allowing you to specify not only Kubernetes `Service` objects but also other custom resources as backends.

### Usage Example: `backendRefs`
Here’s an example of how to configure `backendRefs` in an `HTTPRoute`:

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: HTTPRoute
metadata:
  name: example-route
spec:
  parentRefs:
  - name: example-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: "/"
    backendRefs:
    - name: backend1
      kind: Service
      port: 80
      weight: 80
    - name: backend2
      kind: Service
      port: 80
      weight: 20
```

In this example, similar to `forwardTo`, traffic is routed to `backend1` and `backend2`, but `backendRefs` allows for specifying the resource `kind`, giving more flexibility.

[Back to TOC](#table-of-contents)

---

## 4. Differences Between `forwardTo` and `backendRefs`
The key differences between `forwardTo` and `backendRefs` include:

- **Flexibility:** `backendRefs` allows for referencing various types of backend resources, not just services, making it more versatile.
- **Explicitness:** `backendRefs` is more explicit, requiring you to specify the resource `kind`, which adds clarity to your routing configurations.
- **Future-Proofing:** `backendRefs` is part of the newer API design and is better suited for more complex and scalable environments.

[Back to TOC](#table-of-contents)

---

## 5. When to Use `forwardTo` vs `backendRefs`
- **Use `forwardTo`** if:
  - You have a simple routing scenario with basic backend services.
  - You don’t need the additional flexibility or explicitness provided by `backendRefs`.
  - You want to maintain backward compatibility with older configurations.

- **Use `backendRefs`** if:
  - You need to reference custom resources as backends.
  - You require more explicit control over your routing configurations.
  - You are working in a more complex or scalable environment where future-proofing is important.

[Back to TOC](#table-of-contents)

---

## 6. Conclusion
Both `forwardTo` and `backendRefs` are powerful tools in the Gateway API's `HTTPRoute` resource. While `forwardTo` offers simplicity and ease of use, `backendRefs` provides greater flexibility and control. Understanding when and how to use each will help you design more effective and scalable routing configurations in Kubernetes.

[Back to TOC](#table-of-contents)

---

This concludes the tutorial on `forwardTo` and `backendRefs`. By using the Table of Contents and the provided examples, you should now have a clear understanding of these fields and how to implement them effectively in your `HTTPRoute` configurations.
