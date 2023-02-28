# Simplest K8s Helm
This is the code for the simplest Helm deployment tutorial on my Medium blog.

The simplest Helm chart consists of three components: 

1. The `Chart.yaml` file that is is copied from the command `helm create jade-shooter`
2. The `values.yaml` file that contains a single line that sets `image: jwsy/jade-shooter:2.0.0` 
3. The declarative yaml manifests in `templates/` are in this article https://itnext.io/simplest-minimal-k8s-app-tutorial-with-rancher-desktop-in-5-min-5481edb9a4a5
* `jade-shooter-deployment.yaml`: deploys a scalable `deployment` of a simple app which creates a scalable number of K8s `pod`s which respond to port 8080 and encapsulate a container based on the Nginx unprivileged container
* `jade-shooter-service.yaml`: creates a `service` that allows this webapp's port 8080 to communicate outside of its K8s namespace (AKA dedicated secure cluster) via port 30080
* `jade-shooter-ingress.yaml`: creates an `ingress` that exposes the `service` to requests outside of the K8s cluster at https://jade-shooter.rancher.localhost

## Usage
1. Install Rancher Desktop
    
    Install Rancher Desktop https://rancherdesktop.io/, the easiest way to get a local K8s lab imo. Here's how I set mine up: https://medium.com/macoclock/rancher-desktop-setup-for-k8s-on-your-macos-laptop-6f1c576ceb48

2. Clone this repo and checkout the helm branch. 
    
    Take a look at what's in the simple helm chart which includes a boilerplate-laden `Chart.yaml`, a blank `values.yaml` file, and a `templates/` dir that has the contents of the simplest K8s tutorial
https://itnext.io/simplest-minimal-k8s-app-tutorial-with-rancher-desktop-in-5-min-5481edb9a4a5

    ```
    $ git clone https://github.com/jwsy/simplest-k8s.git -b helm
    $ cd simplest-k8s/
    ```

    Helm will update the templates with values from the `values.yaml` provided or `--set` options passed. Using the `helm template` command, we can see 
    1. `templates/jade-shooter-deployment.yaml` uses a template tag `{{ .Values.image }}` 
    2. `values.yaml` sets `image: jwsy/jade-shooter:2.0.0` 
    3. `helm template .` uses the `Chart.yaml` and `values.yaml` in the current directory to substitute the value into the template and generate the K8s manifest that would be deployed 

        ```
        $ grep -C2 image templates/*
        templates/jade-shooter-deployment.yaml-      containers:
        templates/jade-shooter-deployment.yaml-      - name: jade-shooter0
        templates/jade-shooter-deployment.yaml:        image: {{ .Values.image }}
        templates/jade-shooter-deployment.yaml-        resources:
        templates/jade-shooter-deployment.yaml-          limits:

        $ cat values.yaml
        image: jwsy/jade-shooter:2.0.0

        $ helm template . | grep -C2 image
            containers:
            - name: jade-shooter0
                image: jwsy/jade-shooter:2.0.0
                resources:
                limits:
        ```

3. Let's deploy the helm chart and watch the magic!
 
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
    service/jade-shooter-service   ClusterIP   10.43.223.212   <none>        30080/TCP   10s

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/jade-shooter   1/1     1            1           10s

    NAME                                     DESIRED   CURRENT   READY   AGE
    replicaset.apps/jade-shooter-8779489f9   1         1         1       10s

    NAME                                     CLASS     HOSTS                            ADDRESS       PORTS     AGE
    ingress.networking.k8s.io/jade-shooter   traefik   jade-shooter.rancher.localhost   172.20.10.8   80, 443   10s

    ```

4. Observe the workload
    
    Browse to https://jade-shooter.rancher.localhost to see the game running as version 2.0! 

5. Upgrade the app
    
    Change the image by setting the version in the `helm upgrade` command.

    ```
    $ helm upgrade js . --set image=jwsy/jade-shooter:2.0.1
    Release "js" has been upgraded. Happy Helming!
    NAME: js
    LAST DEPLOYED: Fri Feb 24 15:13:47 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    ```
6. Observe the upgraded workload.

    Browse to https://jade-shooter.rancher.localhost to see the game running as version **2.0.1**! 


## Clean up
To clean up, use `helm uninstall js`

## Notes
* The app is this customizable Kaboom space shooter created in this article: https://javascript.plainenglish.io/kaboom-js-repl-it-custom-top-down-shooter-in-5-min-ebad8157073a?postPublishedType=repub

### Observing `helm ls`
Use the `helm ls` command to observe the app

```bash
$ helm ls
NAME	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART             	APP VERSION
js  	default  	2       	2023-02-24 15:13:47.620007 -0500 EST	deployed	jade-shooter-0.1.0	v1.1
```

### Using `diff` with `helm template`
```bash
$ diff <(helm template js . --set image=jwsy/2.0.0) <(helm template js . --set image=jwsy/2.0.1)
```

```diff
31c31
<         image: jwsy/2.0.0
---
>         image: jwsy/2.0.1
```