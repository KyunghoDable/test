# Value 파일 내 AlertManager 슬랙 채널 및 타이틀명 변경
alertmanager.config.receivers[1].slack_configs.title
alertmanager.config.receivers[1].slack_configs.channel

# Install
```console
CLUSTER_NAME=fluentbit-test-1
ALERTMANAGER_URL=asd.dable.io
PROMETHEUS_URL=zxc.dable.io
GRAFANA_URL=qwe.dable.io
GITHUB_TOKEN=""

# Create Monitoring Namespace
kubectl create ns monitoring

# Prometheus Operator Install
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install -n monitoring prometheus-operator prometheus-community/kube-prometheus-stack -f prometheus-operator-values.yaml \
    --set grafana.ingress.hosts={$GRAFANA_URL} \
    --set alertmanager.alertmanagerSpec.externalUrl=http://$ALERTMANAGER_URL \
    --set alertmanager.ingress.hosts={$ALERTMANAGER_URL} \
    --set prometheus.prometheusSpec.externalUrl=http://$PROMETHEUS_URL \
    --set prometheus.ingress.hosts={$PROMETHEUS_URL}
```


# Dashboard
- ad-api, reco-api 등의 커스텀 대시보드는 이 방법으로는 추가가 되지 않음 ( GUI 에서는 가능 )
- 이유는 찾아봐야할 것 같다
```console
# Download Dashboard
curl -X GET -u $GITHUB_TOKEN:x-oauth-basic -H 'Accept: application/vnd.github.v4.raw' \
'https://api.github.com/repos/teamdable/ppap-devops-kubernetes/contents/system/monitor/dashboard/ad-api.json' > ad-api.json
curl https://grafana.com/api/dashboards/9614/revisions/1/download | sed "s/\${DS_PROMETHEUS}/Prometheus/g" > nginx-ingress-dashboard.json
curl https://grafana.com/api/dashboards/7645/revisions/120/download | sed "s/\${DS_PROMETHEUS}/Prometheus/g" > istio-controlplane-dashboard.json
curl https://grafana.com/api/dashboards/8588/revisions/1/download | sed "s/\${DS_PROMETHEUS}/Prometheus/g" > metric-dashboard.json

# Add Dashboard
kubectl -n monitoring delete configmap my-custom-dashboard
kubectl -n monitoring create configmap my-custom-dashboard --from-file nginx-ingress-dashboard.json --from-file istio-controlplane-dashboard.json --from-file metric-dashboard.json --from-file ad-api.json
kubectl -n monitoring annotate configmaps my-custom-dashboard k8s-sidecar-target-directory="/tmp/dashboards/dable"
kubectl -n monitoring label configmaps my-custom-dashboard grafana_dashboard=1
```



# Uninstall
```console
# Prometheus Operator Delete
helm delete -n monitoring prometheus-operator
# Completely Delete(CRD Delete)
kubectl delete crd alertmanagerconfigs.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd podmonitors.monitoring.coreos.com
kubectl delete crd probes.monitoring.coreos.com
kubectl delete crd prometheuses.monitoring.coreos.com
kubectl delete crd prometheusrules.monitoring.coreos.com
kubectl delete crd servicemonitors.monitoring.coreos.com
kubectl delete crd thanosrulers.monitoring.coreos.com
```
