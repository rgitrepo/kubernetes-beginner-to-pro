# Kubernetes Policies Tutorial

In this tutorial, you will learn about the different kinds of policies in Kubernetes, how they are applied, and the various tools and methods used to enforce these policies. The tutorial is divided into several sections, each focusing on a specific type of policy. At the end of each section, you can return to this Table of Contents (TOC) for easy navigation.

## Table of Contents (TOC)

1. [Introduction to Kubernetes Policies](#introduction)
2. [Apply Policies Using API Objects](#api-objects)
   - [NetworkPolicies](#networkpolicies)
   - [LimitRanges](#limitranges)
   - [ResourceQuotas](#resourcequotas)
3. [Apply Policies Using Admission Controllers](#admission-controllers)
   - [Built-in Admission Controllers](#built-in-admission-controllers)
4. [Apply Policies Using ValidatingAdmissionPolicy](#validating-admission-policy)
5. [Apply Policies Using Dynamic Admission Control](#dynamic-admission-control)
   - [Implementations](#implementations)
6. [Apply Policies Using Kubelet Configurations](#kubelet-configurations)
   - [Process ID Limits and Reservations](#process-id-limits)
   - [Node Resource Managers](#node-resource-managers)

---

## Introduction to Kubernetes Policies

Kubernetes policies are configurations that manage other configurations or runtime behaviors within a cluster. These policies help ensure security and best practices are followed consistently across all workloads and resources.

[Back to TOC](#table-of-contents)

---

## Apply Policies Using API Objects

### NetworkPolicies

NetworkPolicies can be used to restrict ingress and egress traffic for a workload. They define rules that specify how groups of pods are allowed to communicate with each other and with other network endpoints.

[Back to TOC](#table-of-contents)

### LimitRanges

LimitRanges manage resource allocation constraints across different object kinds. They ensure that all pods in a namespace adhere to specific resource limits, preventing any single pod from consuming too many resources.

[Back to TOC](#table-of-contents)

### ResourceQuotas

ResourceQuotas limit resource consumption for a namespace. By setting resource quotas, you can control the total amount of CPU, memory, and other resources that can be used within a namespace.

[Back to TOC](#table-of-contents)

---

## Apply Policies Using Admission Controllers

### Built-in Admission Controllers

An admission controller runs in the API server and can validate or mutate API requests. Kubernetes has several built-in admission controllers that act as policies, such as the AlwaysPullImages admission controller, which modifies a new Pod to set the image pull policy to Always.

[Learn more about Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)  
[Back to TOC](#table-of-contents)

---

## Apply Policies Using ValidatingAdmissionPolicy

Validating admission policies allow configurable validation checks to be executed in the API server using the Common Expression Language (CEL). For example, a ValidatingAdmissionPolicy can be used to disallow the use of the `latest` image tag.

[Learn more about Validating Admission Policies](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/)  
[Back to TOC](#table-of-contents)

---

## Apply Policies Using Dynamic Admission Control

Dynamic admission controllers (or admission webhooks) run outside the API server as separate applications that register to receive webhook requests to perform validation or mutation of API requests. These controllers can perform complex checks, including those requiring retrieval of other cluster resources and external data.

[Learn more about Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)  
[Back to TOC](#table-of-contents)

### Implementations

There are several dynamic admission controllers in the Kubernetes ecosystem that act as flexible policy engines:

- [Kubewarden](https://kubewarden.io/)
- [Kyverno](https://kyverno.io/)
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/)
- [Polaris](https://polaris.docs.fairwinds.com/)

[Back to TOC](#table-of-contents)

---

## Apply Policies Using Kubelet Configurations

### Process ID Limits and Reservations

Kubernetes allows configuring the Kubelet on each worker node. Process ID limits and reservations are used to limit and reserve allocatable PIDs on nodes.

[Back to TOC](#table-of-contents)

### Node Resource Managers

Node Resource Managers can manage compute, memory, and device resources for latency-critical and high-throughput workloads. They act as policies to ensure that resource allocation is optimized for these workloads.

[Back to TOC](#table-of-contents)

