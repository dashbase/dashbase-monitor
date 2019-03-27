## Dashbase Monitor
This document describes how to set up metrics-based monitor cluster onto Kubernetes.

####Concepts
* Kubernetes descriptor(aka values.yml)

    It's a yaml-based file which describe how services configure and work. Here's an [example](example-values.yaml).
    You need to modify it before deploy.
    
    Here're some settings you might want to change:
    1. Ingress host
        ```yaml
        dashbase:
          ingress:
            enabled: true
            host: 127.0.0.1.xip.io # change it to your own host
        ```
    2. InfluxDB Username/Password
        ```yaml
        dashbase:
          services:
            influxdb:
              environment:
                INFLUXDB_DATA_CACHE_MAX_MEMORY_SIZE: 2g
                INFLUXDB_ADMIN_USER: dashbase_monitor      # change it to your username
                INFLUXDB_ADMIN_PASSWORD: dashbase_monitor  # change it to your password
        ```

#### Prerequisites:
* A Kubernetes cluster with full access.
* Helm

    ```
    brew install kubernetes-helm
    helm init
    helm repo add chartmuseum https://charts.dashbase.io
    ```

* Preparatory work to expose your services
    1. Install Ingress

        ```helm install stable/nginx-ingress --name ingress```
    2. Configure `ingress` section in kubernetes descriptor. Enabled the ingress and change the host to your own.
    
    3. Get the Ingress exposed IP and bind DNS. Here's an example(Assuming that your host is `dashbase-monitor.yourhost.com`).
       ```bash
        kubectl --namespace default get services -o wide -w ingress-nginx-ingress-controller
        
        NAME                               TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
        ingress-nginx-ingress-controller   LoadBalancer   10.3.244.2   26.230.25.203   80:30402/TCP,443:31551/TCP   25m   app=nginx-ingress,component=controller,release=ingress
       ```
       And then bind `26.230.25.203` to `*.dashbase-monitor.yourhost.com`.
       


#### How to deploy
1. Prepare the Kubernetes descriptor.

2. Deploy by running

    ```bash
    helm upgrade <your-helm-release> chartmuseum/dashbase-monitor -f example-values.yaml -i
    ```
3. Get the status. It runs well if all pods are `1/1`.
   ```bash
   kc get po -w
   NAME                                                     READY   STATUS    RESTARTS   AGE
   alertmanager-0                                           1/1     Running   0          38m
   grafana-0                                                1/1     Running   0          38m
   influxdb-0                                               1/1     Running   0          38m
   ingress-nginx-ingress-controller-6844d9ddb-n8w9d         1/1     Running   0          32m
   ingress-nginx-ingress-default-backend-677b99f864-wlks4   1/1     Running   0          32m
   prometheus-0                                             1/1     Running   0          38m
   pushgateway-7bf4465f99-4qgv6                             1/1     Running   0          38m
   ```
#### further configuration



Troubleshoot
1. [Dashbase Reference](https://dashbase.atlassian.net/wiki/spaces/DK/pages/192970771/Known+Issues+and+Solutions)