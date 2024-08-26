### Use of IngressClass

The `IngressClass` in Kubernetes defines which controller should implement an Ingress resource, specifying how traffic should be managed and routed. This is crucial in environments with multiple Ingress controllers or when specific traffic behavior is required.

#### YAML Example with `IngressClass`

Here's an example of an Ingress resource that includes an `IngressClassName`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 8080
```

In this YAML file:
- The `ingressClassName: nginx` specifies that the NGINX Ingress controller should handle the traffic routing for this Ingress resource.

### Why Not Having `IngressClass` Causes an Issue

If an Ingress resource does not specify an `IngressClass`, Kubernetes cannot determine which controller should manage the resource, leading to errors and traffic routing failures.

#### Output of Logs Without `IngressClass` Defined

When an `Ingress` resource is applied without specifying an `IngressClass` and no default IngressClass is set, the logs from the Ingress controller might show errors similar to the following:

```bash
kubectl logs -l app=nginx-ingress -n ingress-nginx
```

Example log output without `IngressClass`:

```
E0825 14:32:15.123456 1 leaderelection.go:331] error retrieving resource lock kube-system/nginx-ingress-controller-leader-nginx: configmaps "nginx-ingress-controller-leader-nginx" is forbidden: User "system:serviceaccount:ingress-nginx:nginx-ingress-serviceaccount" cannot get resource "configmaps" in API group "" in the namespace "kube-system"
E0825 14:32:15.223456 1 controller.go:1288] Ingress example-ingress does not contain a valid IngressClass. Please ensure that the `ingressClassName` field is correctly set in the Ingress resource or that a default IngressClass exists.
```

In this example:
- The log shows that the Ingress resource does not contain a valid `IngressClass`, leading to the controller not processing the Ingress resource.
- The error indicates the absence of an `IngressClass`, causing the Ingress controller to fail to route traffic as expected.

### Conclusion

The `IngressClass` is crucial for specifying which controller should handle the traffic rules defined in an Ingress resource. Without it, Kubernetes may not be able to correctly apply traffic rules, leading to errors and an inaccessible application. Adding the `IngressClassName` field in the Ingress YAML ensures the correct controller manages the resource, avoiding issues such as the 404 errors mentioned earlier.
