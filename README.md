# simplest-k8s with local mounts
Simplest k8s deployment as seen here https://jyeee.medium.com/simplest-basic-k8s-tutorial-to-mount-local-host-directory-into-a-pod-volume-example-with-rancher-18b4f1d75cd9

1. Install Rancher Desktop https://rancherdesktop.io/

2. Clone this repo and checkout the `mount-local` branch

    ```bash
    git clone https://github.com/jwsy/simplest-k8s.git
    git checkout mount-local
    ```

3. Create a namespace and apply the manifests in this repo to that namespace
 
    ```bash
    kubectl create namespace js
    kubectl -n js apply -f simplest-k8s
    ```
