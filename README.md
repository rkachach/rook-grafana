# Rook grafana configuration

In order to configure grafana for Rook we have to configure two components **Prometheus**  and **Grafana**. For Prometheus the user has to follow the [instructions](https://rook.io/docs/rook/latest/Storage-Configuration/Monitoring/ceph-monitoring/) step by step. In this setup Grafana consumes the metrics provided by Prometheus to show nice dashboards to the user.


### Grafana configuration

Before going on with Grafana configuratio please make sure Prometheus is up & running correctly in your Rook cluster. To get the URL where Prometheus is litening user can use (save its value as we will need it later):

    echo "http://$(kubectl -n rook-ceph -o jsonpath={.status.hostIP} get pod prometheus-rook-prometheus-0):30900"

#### Grafana SSL certificate

In this guide we will be configuring Grafana to use https, therefore the first step is to generate the certificate the will be used by Grafana (just keep empty all the fields that openssl asks to introduce by hitting enter).

    openssl req -newkey rsa:4096 -new -nodes -keyout grafana.key -x509 -days 365 -out grafana.crt
    kubectl -n rook-ceph create secret tls grafana-tls --key grafana.key --cert grafana.crt

At the end of this process openssl should generate two files: **grafana.crt** and **grafana.key**

Store the generated files as k8s secrets:

    kubectl -n rook-ceph create secret tls grafana-tls --key grafana.key --cert grafana.crt

#### Prepare Grafana configuration files

Before deploying grafana we have to set **prometheus-service-ip:port** in the configuration file grafana-**datasource-config.yaml** to Promtheus endpoint (got previously). At this point we should be ready to deploy Grafana by using the following command:

    kubectl -n rook-ceph apply -f grafana-ini.yaml -f grafana-datasource-config.yaml -f deployment.yaml

The first file **grafana-ini.yaml** will be mapped to /etc/grafana/grafana.ini meanwhile the file grafana-datasource-config.yaml is mapped to /etc/grafana/provisioning/datasources/prometheus.yaml.

#### Create a service for Grafana deployment

    kubectl -n rook-ceph apply -f  service.yaml

This command will create the service grafana as a NodePort listening on the port 32000. In case of minikube users grafana service url can be obtained by running:

    minikube service grafana -n rook-ceph --url


#### Reset Grafana user/password

By default Grafana uses admin:<random_password> as credentials. For development environments it's handly to just set the password to admin (i.e). Following command will set Grafana's password to 'admin':

    kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pods -l "app=grafana" -o jsonpath="{.items[0].metadata.name}") grafana-cli admin reset-admin-password admin
