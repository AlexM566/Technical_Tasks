# Server Monitoring

## 1. Performance metrics
### Metrics to track

- Thi is Depending on what is the Server scope, but there are some generic metrics as following:
- **CPU** : Utilization, Load Average
- **Memory**: Alocated, Used, Unused, Cache mem
- **DISK**: IOPS, Capacity like Free Space vs Used Space
- **Network**: Latency, RTT Latency
- **Service Status**: Just check key service status

### Automate Alerting
#### This is very depengin on the tool we are using
We are asuming to use Prometheus for this

```yaml
# prometheus-win-server-rules.yaml
groups:
- name: windows-server-alerts
  rules:
  - alert: High CPU
    expr: avg(rate(node_cpu_seconds_total[5m])) *100 >85
    for: 2m
    labels:
        severity: high
    annotations:
        summary: "High CPU detected over 85 for 2m on {{ $labels.instance }}"
```

## 2. Performance metrics on network devices
### Metrics to track
- **Interface Stats**
- **Device Resources**: CPU usage, mem usage
- **Flow Data**

# Automate alerting

```yaml
groups:
  - name: network-device-alerts
    rules:
      - alert: HighInterfaceUtilization
        expr: (rate(ifHCInOctets[5m]) + rate(ifHCOutOctets[5m])) * 8 > 0.9e9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High network usage on {{ $labels.instance }}"
```

## 3. How to analyze metrics 
Lets see how we can analyze this metrics

- Add thresholds with ranges
- Trend Correlation :  multiple values in the same graph
- Dashboard from enhancing the view: we can use Grafana for this 
- Anomaly detection; If possible we can integreate ML features presents in monitoring tools such Kibana (ELK stack)
