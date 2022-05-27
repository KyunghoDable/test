# Install
```console
CLUSTER_NAME=
ALERTMANAGER_URL=
PROMETHEUS_URL=
GRAFANA_URL=
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
- 아래 방식으로 하는 경우 추가는 되지만 에러로 인해 보이지 않음 
- Grafana 자체적인 문제로 확인됨 (https://medium.com/@SergeyNuzhdin/going-open-source-in-monitoring-part-iii-10-most-useful-grafana-dashboards-to-monitor-kubernetes-7d22ac4645db)
- 따라서 추가는 Grafana 페이지에 들어가서 직접 해야함
```console
# Download Dashboard
curl -X GET -u $GITHUB_TOKEN:x-oauth-basic -H 'Accept: application/vnd.github.v4.raw' \
'https://api.github.com/repos/teamdable/ppap-devops-kubernetes/contents/system/monitor/dashboard/general/Pod-monitoring.json' > Pod-monitoring.json
curl https://grafana.com/api/dashboards/9614/revisions/1/download > nginx-ingress-dashboard.json
curl https://grafana.com/api/dashboards/6336/revisions/1/download > pod-mo.json

# Add Dashboard
# reco/ad api 등의 커스텀은 import 가 아직 되지 않음.. 확인 필요
kubectl -n monitoring delete configmap my-custom-dashboard
kubectl -n monitoring create configmap my-custom-dashboard --from-file Pod-monitoring.json --from-file nginx-ingress-dashboard.json --from-file pod-mo.json
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
