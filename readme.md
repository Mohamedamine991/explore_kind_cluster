
# Kubernetes Cluster Resource Export Scripts

## Overview

These scripts are designed to export the configuration and state of Kubernetes resources in a cluster. The first script targets namespaced resources, while the second script handles cluster-wide (non-namespaced) resources. The exported resources are saved in YAML format, with the YAML content processed through `kubectl neat` for readability.

## Prerequisites

- Ensure you have `kubectl` installed and configured to interact with your Kubernetes cluster.
- The `kubectl neat` plugin should be installed. You can install it using:
  ```sh
  kubectl krew install neat
  ```

## Script 1: Exporting Namespaced Resources

### Description

This script exports all namespaced resources from all namespaces in the cluster and saves them in the `namespaces/` directory.

### Script

\`\`\`sh
#!/bin/bash

# Fetch the list of all namespaced API resources
ns_api=$(kubectl api-resources -o name --namespaced=true)

# Fetch the list of all namespaces
namespaces=$(kubectl get namespaces -o name | sed 's/namespace\///g')

echo "Exporting namespaced resources"
mkdir namespaces

# Loop through each namespace
for n in $namespaces
do
    # Loop through each namespaced resource type
    for r in $ns_api
    do
        # Get the list of resource instances for the current type in the current namespace
        names=$(kubectl get "$r" -n "$n" -o name)
        if [ -z "$names" ]; then
            echo "Skipping because of empty value"
        else
            echo "Dumping $r in $n"
            # Loop through each resource instance
            for name in $names
            do
                # Create the directory structure and export the resource manifest
                mkdir -p "$(dirname "namespaces/$n/$name")"
                kubectl get "$name" -n "$n" -o yaml | kubectl neat > "namespaces/$n/$name.yaml"
                echo "Manifest saved to namespaces/$n/$name.yaml"
            done
        fi
    done
done
\`\`\`

### Explanation

1. **Fetch Namespaced Resources**: The script starts by fetching a list of all namespaced API resources.
2. **Fetch Namespaces**: It then fetches all the namespaces in the cluster.
3. **Directory Creation**: Creates a `namespaces/` directory to store the exported resources.
4. **Loop through Namespaces and Resources**:
   - For each namespace, it iterates over each namespaced resource type.
   - For each resource type, it fetches the resource instances and exports them to a YAML file.
   - Uses `kubectl neat` to clean up the YAML output for readability.

## Script 2: Exporting Cluster-wide Resources

### Description

This script exports all cluster-wide (non-namespaced) resources and saves them in the `cluster/` directory.

### Script

\`\`\`sh
#!/bin/bash

# Fetch the list of all non-namespaced API resources
non_ns_api=$(kubectl api-resources -o name --namespaced=false)

mkdir cluster

echo "Exporting cluster resources"
# Loop through each non-namespaced resource type
for r in $non_ns_api
do
    # Get the list of resource instances for the current type
    names=$(kubectl get "$r" -o name)
    if [ -z "$names" ]; then
        echo "Skipping because of empty value"
    else
        echo "Dumping $r"
        # Loop through each resource instance
        for name in $names
        do
            # Create the directory structure and export the resource manifest
            mkdir -p "$(dirname "cluster/$name")"
            kubectl get "$name" -o yaml | kubectl neat > "cluster/$name.yaml"
            echo "Manifest saved to cluster/$name.yaml"
        done
    fi
done
\`\`\`

### Explanation

1. **Fetch Non-Namespaced Resources**: The script starts by fetching a list of all non-namespaced API resources.
2. **Directory Creation**: Creates a `cluster/` directory to store the exported resources.
3. **Loop through Resources**:
   - For each resource type, it fetches the resource instances and exports them to a YAML file.
   - Uses `kubectl neat` to clean up the YAML output for readability.

## Usage

1. Save the scripts to files, for example, `export-namespaced.sh` and `export-cluster.sh`.
2. Make the scripts executable:
   ```sh
   chmod +x export-namespaced.sh export-cluster.sh
   ```
3. Run the scripts:
   ```sh
   ./export-namespaced.sh
   ./export-cluster.sh
   ```
4. The resources will be exported to the `namespaces/` and `cluster/` directories, respectively.

These scripts provide a comprehensive way to back up the state of a Kubernetes cluster, which can be useful for disaster recovery, auditing, or migration purposes.