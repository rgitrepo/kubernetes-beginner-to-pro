# Debug Containers Tutorial

Welcome to this comprehensive tutorial on Debug Containers. This guide will take you step by step through the process of creating, copying, and managing debug and ephemeral containers, highlighting their uses, limitations, and best practices. 

### Table of Contents

1. [Introduction to Debug Containers](#introduction-to-debug-containers)
2. [Creating a Debug Container](#creating-a-debug-container)
3. [Copying a Running Pod as a Debug Container](#copying-a-running-pod-as-a-debug-container)
4. [Challenges with Debug Containers](#challenges-with-debug-containers)
5. [Managing Ephemeral Containers](#managing-ephemeral-containers)
6. [Practical Example of Creating an Ephemeral Container](#practical-example-of-creating-an-ephemeral-container)


---

## Introduction to Debug Containers

Debug containers are special containers used for troubleshooting issues within running pods. They allow developers and operators to inspect, modify, and troubleshoot a pod without directly interacting with the production environment. This practice is vital because directly accessing production environments can lead to unintended consequences, especially if changes are made live.

A debug container is essentially a container that runs alongside an existing pod, allowing you to investigate and troubleshoot issues within the pod in a safe manner.

[Back to top](#table-of-contents)

---

## Creating a Debug Container

To create a debug container, you'll follow a specific procedure that ensures the container is added to the pod without affecting the production environment. Here's how you can create a debug container:

### Steps:

1. **Prepare the Pod**: Ensure your pod is running. In this example, the pod is named `PPRAT`.

2. **Create the Debug Container**: Use the following command to create a debug container.
    ```bash
    kubectl debug -it --image=nginx --target=PPRAT
    ```
    In this command:
    - `-it` allows interactive mode.
    - `--image=nginx` specifies the image for the debug container.
    - `--target=PPRAT` targets the running pod named PPRAT.

3. **Interact with the Debug Container**: Once the debug container is created, you can run any commands inside it. For example, you can use:
    ```bash
    cat /etc/nginx/nginx.conf
    ```
    This command would show the configuration file for Nginx if it's used in the container.

4. **Exit the Debug Container**: After troubleshooting, you can exit the debug container with:
    ```bash
    exit
    ```

### Example:
If you accidentally forget to include a command like `bash` in your debug container, you might encounter issues. Always double-check your command syntax to ensure the debug container functions as expected.

[Back to top](#table-of-contents)

---

## Copying a Running Pod as a Debug Container

Sometimes, instead of adding a debug container, you may want to create a copy of a running pod for debugging purposes. This allows you to troubleshoot without risking changes to the live environment.

### Steps:

1. **Run the Copy Command**:
    ```bash
    kubectl debug --copy-to=copy-of-pod --target=PPRAT
    ```
    In this command:
    - `--copy-to=copy-of-pod` creates a copy of the running pod named PPRAT.
    - `--target=PPRAT` specifies the target pod to copy.

2. **Debug the Copied Pod**: Once the copy is made, you can apply changes and debug without affecting the original pod.

### Note:
Always use this method in production environments to ensure that you don't disrupt live services.

[Back to top](#table-of-contents)

---

## Challenges with Debug Containers

Debug containers have some limitations:

1. **Cannot Kill Debug Containers Individually**: You cannot kill debug containers directly; you must kill the main container if you want to remove the debug container. This is a significant limitation, especially in production environments where you need to maintain control over all running containers.

2. **Ephemeral Containers**: These are temporary containers that are added to a pod for debugging purposes. They cannot be deleted once created, which can be a challenge when managing resources.

### Example:
In the transcript, multiple ephemeral containers were created, each with its own container ID. These containers could not be deleted, which posed a challenge in resource management.

[Back to top](#table-of-contents)

---

## Managing Ephemeral Containers

Ephemeral containers are used for short-term debugging tasks within a pod. However, they come with a significant downside: they cannot be deleted after being created. This limitation makes them less flexible compared to regular debug containers.

### Steps:

1. **Create an Ephemeral Container**:
    ```bash
    kubectl debug -it --image=nginx --target=PPRAT
    ```
    This command will create an ephemeral container within the pod PPRAT.

2. **Interact with the Ephemeral Container**: You can perform various tasks within the ephemeral container, but remember that it cannot be deleted afterward.

### Example:
In the example from the transcript, ephemeral containers were created, but when the user attempted to delete them, they found it was impossible. This is a critical consideration when using ephemeral containers in production environments.

[Back to top](#table-of-contents)

---

## Practical Example of Creating an Ephemeral Container

Let's walk through the process of creating an ephemeral container with a real example:

### Example:

1. **Create the Ephemeral Container**:
    ```bash
    kubectl debug -it --image=nginx --target=PPRAT
    ```

2. **Run Commands Inside the Container**:
    - Check Nginx configuration:
      ```bash
      cat /etc/nginx/nginx.conf
      ```
    - Verify the container's functionality.

3. **Limitations**: As mentioned earlier, you cannot delete this container once it's created. It remains part of the pod until the pod is terminated.

[Back to top](#table-of-contents)

