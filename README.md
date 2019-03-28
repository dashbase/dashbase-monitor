## Dashbase Monitor
This document describes how to set up Dashbase metrics monitoring cluster onto an existing Kubernetes.

####Concepts
* Kubernetes descriptor(aka `values.yaml`)

    It's a yaml-based file that describes how services are configured. Here's an [example](example-values.yaml).
    You will need to create your own and modify it prior to deploying.

#### Prerequisites:
* A Kubernetes cluster with full access.
* Helm

    ```
    helm init
    helm repo add chartmuseum https://charts.dashbase.io
    ```

* Preparatory work to expose your services via NGINX Ingress if running on a cloud cluster.
    1. Install Ingress

        ```helm install stable/nginx-ingress --name ingress```
    2. Configure the `ingress` section in your `values.yaml`. Enable the `ingress` and change the host to your own.

        ```yaml
        dashbase:
          ingress:
            enabled: true
            host: 127.0.0.1.xip.io # change it to your own domain name, e.g. monitor.dashbase.io. Services will be accessible at grafana.monitor.dashbase.io, etc.
        ```

    3. The ingress should expose an external ip. Register the `EXTERNAL-IP` with your DNS. Here's an example (Assuming that your host is `example-monitor.yourdomain.com`):

        ```bash
          kubectl --namespace default get services -o wide -w ingress-nginx-ingress-controller
          
          NAME                               TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
          ingress-nginx-ingress-controller   LoadBalancer   10.3.244.2   26.230.25.203   80:30402/TCP,443:31551/TCP   25m   app=nginx-ingress,component=controller,release=ingress
        ```
       And then bind `26.230.25.203` to `*.example-monitor.yourdomain.com`.
       


#### How to deploy
1. Prepare your Kubernetes descriptor (`values.yaml`). You can use the provided `examples-values.yaml` as a baseline.

2. Deploy with:

    ```bash
    helm upgrade <helm-release-name> chartmuseum/dashbase-monitor -f values.yaml -i
    ```
3. Get the status. It runs well if all pods are `1/1`.
   ```bash
   kubectl get po -w
   NAME                                                     READY   STATUS    RESTARTS   AGE
   alertmanager-0                                           1/1     Running   0          3m
   grafana-0                                                1/1     Running   0          3m
   influxdb-0                                               1/1     Running   0          3m
   ingress-nginx-ingress-controller-6844d9ddb-n8w9d         1/1     Running   0          3m
   ingress-nginx-ingress-default-backend-677b99f864-wlks4   1/1     Running   0          3m
   prometheus-0                                             1/1     Running   0          3m
   pushgateway-7bf4465f99-4qgv6                             1/1     Running   0          3m
   ```

Then you can visit pushgateway via `pushgateway.example-monitor.yourdomain.com`, grafana via `grafana.example-monitor.yourdomain.com`, etc.
#### further configuration


>>>>>>> master

Troubleshoot
1. [Dashbase Reference](https://dashbase.atlassian.net/wiki/spaces/DK/pages/192970771/Known+Issues+and+Solutions)
