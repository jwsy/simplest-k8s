# Simplest K8s Helm 4
This is the code for the simplest Helm deployment tutorial on my Medium blog at <https://medium.com/@jyeee/simplest-basic-helm-chart-tutorial-with-rancher-desktop-k8s-7b87c85d960e>

The simplest Helm chart consists of three components: 

1. The `Chart.yaml` file that is is copied from the command `helm create jade-shooter`
2. The `values.yaml` file that sets `image: ghcr.io/jwsy/jade-shooter-22:v2.0.3` and `replicaCount: 1`
3. The declarative yaml manifests in `templates/` are in this article https://itnext.io/simplest-minimal-k8s-app-tutorial-with-rancher-desktop-in-5-min-5481edb9a4a5
* `jade-shooter-deployment.yaml`: deploys a scalable `deployment` of a simple app which creates a scalable number of K8s `pod`s which respond to port 8080 and encapsulate a container based on the Nginx unprivileged container
* `jade-shooter-service.yaml`: creates a `service` that allows this webapp's port 8080 to communicate outside of its K8s namespace (AKA dedicated secure cluster) via port 38080
* `jade-shooter-ingress.yaml`: creates an `ingress` that exposes the `service` to requests outside of the K8s cluster at https://jade-shooter.rancher.localhost

## Current app version

The Helm chart currently pins the app to:

```yaml
appVersion: v2.0.3
```

The default deployment values are:

```yaml
image: ghcr.io/jwsy/jade-shooter-22:v2.0.3
replicaCount: 1
```

## Usage
1. Install Rancher Desktop
    
    Install Rancher Desktop https://rancherdesktop.io/, the easiest way to get a local K8s lab imo. Here's how I set mine up: https://medium.com/macoclock/rancher-desktop-setup-for-k8s-on-your-macos-laptop-6f1c576ceb48

2. Clone this repo and checkout the helm4 branch. 
    
    Take a look at what's in the simple helm chart which includes a boilerplate-laden `Chart.yaml`, a blank `values.yaml` file, and a `templates/` dir that has the contents of the simplest K8s tutorial
https://itnext.io/simplest-minimal-k8s-app-tutorial-with-rancher-desktop-in-5-min-5481edb9a4a5

    ```
    $ git clone https://github.com/jwsy/simplest-k8s.git -b helm4
    $ cd simplest-k8s/
    ```

    Helm will update the templates with values from the `values.yaml` provided or `--set` options passed. Using the `helm template` command, we can see 
    1. `templates/jade-shooter-deployment.yaml` uses template tags for `{{ .Values.replicaCount | default 1 }}` and `{{ .Values.image }}` 
    2. `values.yaml` sets `image: ghcr.io/jwsy/jade-shooter-22:v2.0.3` and `replicaCount: 1`
    3. `helm template .` uses the `Chart.yaml` and `values.yaml` in the current directory to substitute the values into the template and generate the K8s manifest that would be deployed 

        ```
        $ grep -E 'replicas|image' templates/jade-shooter-deployment.yaml
          replicas: {{ .Values.replicaCount | default 1 }}
                image: {{ .Values.image }}

        $ cat values.yaml
        image: ghcr.io/jwsy/jade-shooter-22:v2.0.3
        replicaCount: 1

        $ helm template . | grep -E 'replicas|image'
          replicas: 1
                image: ghcr.io/jwsy/jade-shooter-22:v2.0.3
        ```

3. Optional: set the number of replicas before deploying.

    Edit `values.yaml`:

    ```yaml
    replicaCount: 3
    ```

    Or pass it at install time:

    ```bash
    $ helm install js . --set replicaCount=3
    ```

4. Let's deploy the helm chart and watch the magic!
 
    ```bash
    $ helm install js .
    NAME: js
    LAST DEPLOYED: Fri Feb 24 14:51:18 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    ```

    
    You can see the full list of resources deployed with the command `kubectl get all,ing`. In the gif I'm running the command `watch -d -n1 kubectl -n default get all,ing`
    ```bash
    $ kubectl -n default get all,ing
    NAME                               READY   STATUS    RESTARTS   AGE
    pod/jade-shooter-8779489f9-zt9c5   1/1     Running   0          10s

    NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
    service/kubernetes             ClusterIP   10.43.0.1       <none>        443/TCP     23h
    service/jade-shooter-service   ClusterIP   10.43.223.212   <none>        38080/TCP   10s

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/jade-shooter   1/1     1            1           10s

    NAME                                     DESIRED   CURRENT   READY   AGE
    replicaset.apps/jade-shooter-8779489f9   1         1         1       10s

    NAME                                     CLASS     HOSTS                            ADDRESS       PORTS     AGE
    ingress.networking.k8s.io/jade-shooter   traefik   jade-shooter.rancher.localhost   172.20.10.8   80, 443   10s

    ```

5. Observe the workload
    
    Browse to https://jade-shooter.rancher.localhost to see the game running as version **v2.0.3**!

6. Change replicas after deploying
    
    Change the replica count with `helm upgrade`:

    ```
    $ helm upgrade js . --set replicaCount=3
    Release "js" has been upgraded. Happy Helming!
    NAME: js
    LAST DEPLOYED: Fri Feb 24 15:13:47 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    ```

    Verify the deployment scaled:

    ```bash
    $ kubectl get pods -l app=jade-shooter
    ```


## Clean up
To clean up, use `helm uninstall js`

## Install from OCI Registry (Helm 4)

Helm 4 supports OCI registries natively, so you can install this chart directly from GHCR without cloning the repo or adding a Helm repo first:

```bash
helm install jade-shooter oci://ghcr.io/jwsy/charts/jade-shooter --version 1.0.4
```

Set replicas with `--set`:

```bash
helm install jade-shooter oci://ghcr.io/jwsy/charts/jade-shooter --version 1.0.4 --set replicaCount=3
```

Upgrade to a new chart version or change values the same way:

```bash
helm upgrade jade-shooter oci://ghcr.io/jwsy/charts/jade-shooter --version 1.0.4 --set replicaCount=3
```

The chart is published at https://github.com/jwsy/simplest-k8s/pkgs/container/charts%2Fjade-shooter

## Notes
* The app is this customizable Kaboom space shooter created in this article: https://javascript.plainenglish.io/kaboom-js-repl-it-custom-top-down-shooter-in-5-min-ebad8157073a?postPublishedType=repub
* The container image is built and published from https://github.com/jwsy/jade-shooter-22

### Observing `helm ls`
Use the `helm ls` command to observe the app

```bash
$ helm ls
NAME	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART             	APP VERSION
js  	default  	2       	2023-02-24 15:13:47.620007 -0500 EST	deployed	jade-shooter-1.0.4	v2.0.3
```

### Using `diff` with `helm template`
```bash
$ diff <(helm template js . --set replicaCount=1) <(helm template js . --set replicaCount=3)
```

```diff
6c6
<   replicas: 1
---
>   replicas: 3
```
