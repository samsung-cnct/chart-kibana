# kibana-chart
[![Build Status](https://jenkins.migrations.cnct.io/buildStatus/icon?job=pipeline-kibana/master)](https://jenkins.cnct.io/job/pipeline-kibana/job/master)

A Kibana Helm chart that provides data visualization for logs in your elasticsearch cluster.

## Install
To install the chart with the release name `kibana`:
 ```
 helm repo add cnct https://charts.cnct.io
 helm repo update
 helm install cnct/kibana --name kibana
 ```  
