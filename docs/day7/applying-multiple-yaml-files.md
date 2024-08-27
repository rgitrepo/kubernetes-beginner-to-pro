### Tutorial: Applying Multiple YAML Files Using `kubectl apply -f` on a Folder (with Recursive Option)

When managing Kubernetes resources, you may have YAML files organized across multiple directories. Instead of applying each file individually or manually navigating through directories, you can apply all the files in a directory and its subdirectories recursively using the `-R` option with `kubectl apply -f`.

#### Step 1: Organize Your YAML Files

Organize your YAML files within directories and subdirectories. For example:

```
/my-kubernetes-configs
  ├── base
  │   ├── deployment.yaml
  │   ├── service.yaml
  ├── overlays
  │   ├── dev
  │   │   ├── configmap.yaml
  │   │   └── secret.yaml
  │   └── prod
  │       ├── configmap.yaml
  │       └── secret.yaml
```

#### Step 2: Use the `kubectl apply -f` Command with the `-R` Option

To apply all the YAML files in the directory and its subdirectories, navigate to the parent directory and use the following command:

```bash
kubectl apply -f . -R
```

Or specify the path to the directory if you are outside of it:

```bash
kubectl apply -f /path/to/my-kubernetes-configs -R
```

#### How It Works:

- The `-R` (or `--recursive`) option tells `kubectl` to search through the specified directory and all of its subdirectories for YAML files to apply.
- The `-f` flag specifies the root directory containing the resource definitions.

#### Example Output:

Running the command will produce output similar to this:

```bash
deployment.apps/my-deployment created
service/my-service created
configmap/my-dev-configmap created
secret/my-dev-secret created
configmap/my-prod-configmap created
secret/my-prod-secret created
```

#### Notes:

- `kubectl apply -f . -R` will apply all valid Kubernetes resource files found in the directory and its subdirectories.
- Ensure that all files have the correct `.yaml` or `.yml` extensions and are properly formatted.
- If a file is not a valid Kubernetes resource definition, `kubectl` will return an error for that specific file.

#### Conclusion:

Using `kubectl apply -f <directory> -R` is a powerful way to deploy complex Kubernetes configurations spread across multiple directories. This approach is particularly useful for managing environments with different overlays (e.g., development, staging, production) or when organizing resources hierarchically. By utilizing the recursive option, you can efficiently manage and deploy all your Kubernetes resources with a single command.
