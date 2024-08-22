# Kubernetes Application Deployment Tutorial

## Table of Contents (TOC)

1. [Introduction to Application Deployment](#introduction-to-application-deployment)
2. [Creating a Deployment](#creating-a-deployment)
   - [Using `kubectl create`](#using-kubectl-create)
   - [Specifying Image Versions](#specifying-image-versions)
   - [Common Errors and Interview Questions](#common-errors-and-interview-questions)
3. [Working with Docker Images](#working-with-docker-images)
   - [Pulling Images](#pulling-images)
   - [Using Tags in Docker](#using-tags-in-docker)
   - [Docker Hub Integration](#docker-hub-integration)
4. [Describing Deployments](#describing-deployments)
   - [Understanding Spec Files](#understanding-spec-files)
   - [Using `kubectl describe`](#using-kubectl-describe)
5. [Exposing Deployments via Services](#exposing-deployments-via-services)
   - [Service Types Overview](#service-types-overview)
   - [Exposing with `kubectl expose`](#exposing-with-kubectl-expose)
   - [LoadBalancer Service Example](#loadbalancer-service-example)
6. [Exploring Services](#exploring-services)
   - [Listing Services](#listing-services)
   - [Describing Services](#describing-services)
7. [Handling Ingress and API Gateways](#handling-ingress-and-api-gateways)
   - [Using Ingress in Production](#using-ingress-in-production)
   - [Understanding API Gateway](#understanding-api-gateway)
8. [Conclusion](#conclusion)

---

## Introduction to Application Deployment

In this tutorial, we will walk through the process of deploying a Go application on Kubernetes, using Docker for containerization and Kubernetes for orchestration. We will cover key topics such as creating deployments, working with Docker images, exposing services, and handling Ingress and API Gateway in production environments.

[Back to TOC](#table-of-contents)

---

## Creating a Deployment

### Using `kubectl create`

The first step in deploying an application in Kubernetes is to create a deployment. You can use the `kubectl create` command to do this. The command syntax is as follows:

```bash
kubectl create deploy <deployment_name> --image=<image_name>
```

This command creates a deployment with the specified name and image.

[Back to TOC](#table-of-contents)

### Specifying Image Versions

When pulling a Docker image, it is crucial to specify the correct version using tags. If you don't specify a tag, Kubernetes will attempt to pull the image with the `latest` tag by default. If the `latest` tag is not available, this can lead to errors.

Example:

```bash
docker pull <image_name>:1.2
```

In this example, the image with version `1.2` is pulled.

[Back to TOC](#table-of-contents)

### Common Errors and Interview Questions

An important interview question to consider: *What happens if you don't specify a tag when pulling a Docker image?*

Answer: Kubernetes will attempt to pull the `latest` tag. If the `latest` tag is not defined, no image will be pulled unless a specific version is provided.

[Back to TOC](#table-of-contents)

---

## Working with Docker Images

### Pulling Images

Before deploying your application, ensure that the Docker image is pulled from Docker Hub. Use the following command to pull an image:

```bash
docker pull <image_name>:<tag>
```

Make sure to verify the image by running:

```bash
docker images
```

[Back to TOC](#table-of-contents)

### Using Tags in Docker

Tags are essential for versioning Docker images. Always ensure that the correct tag is used when pulling or deploying images. For example:

```bash
docker pull myapp:1.2
```

This command pulls the image tagged with `1.2`.

[Back to TOC](#table-of-contents)

### Docker Hub Integration

You can host your Docker images on Docker Hub and pull them directly into your Kubernetes deployment. Ensure your image is pushed to Docker Hub using:

```bash
docker push <dockerhub_username>/<image_name>:<tag>
```

[Back to TOC](#table-of-contents)

---

## Describing Deployments

### Understanding Spec Files

The spec file of a deployment contains all the necessary details about the application deployment, such as the number of replicas, image, labels, etc.

[Back to TOC](#table-of-contents)

### Using `kubectl describe`

To view the details of a deployment, you can use the `kubectl describe` command:

```bash
kubectl describe deployment <deployment_name>
```

This command shows detailed information about the deployment, including labels, selectors, and other metadata.

Example output:

```bash
Name:               day6
Namespace:          default
Labels:             app=day6
...
```

[Back to TOC](#table-of-contents)

---

## Exposing Deployments via Services

### Service Types Overview

Kubernetes services are used to expose your deployments to the outside world or other components within the cluster. Common service types include:

- **ClusterIP**: Exposes the service on a cluster-internal IP.
- **NodePort**: Exposes the service on each Node's IP at a static port.
- **LoadBalancer**: Exposes the service externally using a cloud provider's load balancer.
- **ExternalName**: Maps a service to a DNS name.

[Back to TOC](#table-of-contents)

### Exposing with `kubectl expose`

To expose a deployment as a service, use the `kubectl expose` command:

```bash
kubectl expose deployment <deployment_name> --type=<service_type> --port=<port_number>
```

Example:

```bash
kubectl expose deployment day6 --type=LoadBalancer --port=8080
```

This exposes the deployment named `day6` on port `8080` as a LoadBalancer service.

[Back to TOC](#table-of-contents)

### LoadBalancer Service Example

Here’s an example of a LoadBalancer service configuration:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: day6-service
spec:
  type: LoadBalancer
  ports:
    - port: 8080
  selector:
    app: day6
```

This YAML defines a LoadBalancer service that exposes port 8080.

[Back to TOC](#table-of-contents)

---

## Exploring Services

### Listing Services

To list all services in your Kubernetes cluster, use:

```bash
kubectl get svc
```

This will show all the services, their types, and other details.

[Back to TOC](#table-of-contents)

### Describing Services

To get detailed information about a specific service, use:

```bash
kubectl describe svc <service_name>
```

This command provides an in-depth look at the service, including its external IP, ports, and other configuration details.

[Back to TOC](#table-of-contents)

---

## Handling Ingress and API Gateways

### Using Ingress in Production

Ingress in Kubernetes is used to manage external access to services, typically HTTP. It works on Layer 7 of the OSI model and can route traffic to different services based on the request path or host.

[Back to TOC](#table-of-contents)

### Understanding API Gateway

An API Gateway is used in production environments as an alternative to or in addition to Ingress. It provides a way to manage and route API requests, offering features like authentication, rate limiting, and more.

[Back to TOC](#table-of-contents)

---

## Conclusion

This tutorial has covered the fundamental aspects of deploying an application on Kubernetes, from creating deployments and working with Docker images to exposing services and handling Ingress/API Gateway. By understanding these concepts, you'll be better equipped to manage and deploy applications in a Kubernetes environment.

[Back to TOC](#table-of-contents)
