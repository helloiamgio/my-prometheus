# Query Prometheus utili per k8s


**Kubernetes Node not ready (instance {{ $labels.instance }})
**
`kube_node_status_condition{condition="Ready",status="true"} == 0`


**Kubernetes Node memory pressure (instance {{ $labels.instance }})
**
`kube_node_status_condition{condition="MemoryPressure",status="true"} == 1`


**Kubernetes Node disk pressure (instance {{ $labels.instance }})
**
`kube_node_status_condition{condition="DiskPressure",status="true"} == 1`


**Kubernetes Node network unavailable (instance {{ $labels.instance }})
**
`kube_node_status_condition{condition="NetworkUnavailable",status="true"} == 1`


**Kubernetes Node out of pod capacity (instance {{ $labels.instance }})
**
`sum by (node) ((kube_pod_status_phase{phase="Running"} == 1) + on(uid) group_left(node) (0 * kube_pod_info{pod_template_hash=""})) / sum by (node) (kube_node_status_allocatable{resource="pods"}) * 100 > 90`


**Kubernetes Container oom killer (instance {{ $labels.instance }})
**
`(kube_pod_container_status_restarts_total - kube_pod_container_status_restarts_total offset 10m >= 1) and ignoring (reason) min_over_time(kube_pod_container_status_last_terminated_reason{reason="OOMKilled"}[10m]) == 1`


**Kubernetes Job failed (instance {{ $labels.instance }})
**
`kube_job_status_failed > 0`


**Kubernetes CronJob suspended (instance {{ $labels.instance }})
**
`kube_cronjob_spec_suspend != 0`


**Kubernetes PersistentVolumeClaim pending (instance {{ $labels.instance }})
**
`kube_persistentvolumeclaim_status_phase{phase="Pending"} == 1`


**Kubernetes Volume out of disk space (instance {{ $labels.instance }})
**
`kubelet_volume_stats_available_bytes / kubelet_volume_stats_capacity_bytes * 100 < 10`


**Kubernetes Volume full in four days (instance {{ $labels.instance }})
**
`predict_linear(kubelet_volume_stats_available_bytes[6h:5m], 4 * 24 * 3600) < 0`


**Kubernetes PersistentVolume error (instance {{ $labels.instance }})
**
`kube_persistentvolume_status_phase{phase=~"Failed|Pending", job="kube-state-metrics"} > 0`


**Kubernetes StatefulSet down (instance {{ $labels.instance }})
**
`kube_statefulset_replicas != kube_statefulset_status_replicas_ready > 0`


**Kubernetes HPA scale inability (instance {{ $labels.instance }})
**
`(kube_horizontalpodautoscaler_spec_max_replicas - kube_horizontalpodautoscaler_status_desired_replicas) * on (horizontalpodautoscaler,namespace) (kube_horizontalpodautoscaler_status_condition{condition="ScalingLimited", status="true"} == 1) == 0`


**Kubernetes HPA metrics unavailability (instance {{ $labels.instance }})
**
`kube_horizontalpodautoscaler_status_condition{status="false", condition="ScalingActive"} == 1`


**Kubernetes HPA scale maximum (instance {{ $labels.instance }})
**
`(kube_horizontalpodautoscaler_status_desired_replicas >= kube_horizontalpodautoscaler_spec_max_replicas) and (kube_horizontalpodautoscaler_spec_max_replicas > 1) and (kube_horizontalpodautoscaler_spec_min_replicas != kube_horizontalpodautoscaler_spec_max_replicas)`


**Kubernetes HPA underutilized (instance {{ $labels.instance }})
**
`max(quantile_over_time(0.5, kube_horizontalpodautoscaler_status_desired_replicas[1d]) == kube_horizontalpodautoscaler_spec_min_replicas) by (horizontalpodautoscaler) > 3`


**Kubernetes Pod not healthy (instance {{ $labels.instance }})
**
`sum by (namespace, pod) (kube_pod_status_phase{phase=~"Pending|Unknown|Failed"}) > 0`


**Kubernetes pod crash looping (instance {{ $labels.instance }})
**
`increase(kube_pod_container_status_restarts_total[1m]) > 3`


**Kubernetes ReplicaSet replicas mismatch (instance {{ $labels.instance }})
**
`kube_replicaset_spec_replicas != kube_replicaset_status_ready_replicas`


**Kubernetes Deployment replicas mismatch (instance {{ $labels.instance }})
**
`kube_deployment_spec_replicas != kube_deployment_status_replicas_available`


**Kubernetes StatefulSet replicas mismatch (instance {{ $labels.instance }})
**
`kube_statefulset_status_replicas_ready != kube_statefulset_status_replicas`


**Kubernetes Deployment generation mismatch (instance {{ $labels.instance }})
**
`kube_deployment_status_observed_generation != kube_deployment_metadata_generation`


**Kubernetes StatefulSet generation mismatch (instance {{ $labels.instance }})
**
`kube_statefulset_status_observed_generation != kube_statefulset_metadata_generation`


**Kubernetes StatefulSet update not rolled out (instance {{ $labels.instance }})
**
`max without (revision) (kube_statefulset_status_current_revision unless kube_statefulset_status_update_revision) * (kube_statefulset_replicas != kube_statefulset_status_replicas_updated)`


**Kubernetes DaemonSet rollout stuck (instance {{ $labels.instance }})
**
`kube_daemonset_status_number_ready / kube_daemonset_status_desired_number_scheduled * 100 < 100 or kube_daemonset_status_desired_number_scheduled - kube_daemonset_status_current_number_scheduled > 0`


**Kubernetes DaemonSet misscheduled (instance {{ $labels.instance }})
**
`kube_daemonset_status_number_misscheduled > 0`


**Kubernetes CronJob too long (instance {{ $labels.instance }})
**
`time() - kube_cronjob_next_schedule_time > 3600`


**Kubernetes Job slow completion (instance {{ $labels.instance }})
**
`kube_job_spec_completions - kube_job_status_succeeded - kube_job_status_failed > 0`


**Kubernetes API server errors (instance {{ $labels.instance }})
**
`sum(rate(apiserver_request_total{job="apiserver",code=~"(?:5..)"}[1m])) by (instance, job) / sum(rate(apiserver_request_total{job="apiserver"}[1m])) by (instance, job) * 100 > 3`


**Kubernetes API client errors (instance {{ $labels.instance }})
**
`(sum(rate(rest_client_requests_total{code=~"(4|5).."}[1m])) by (instance, job) / sum(rate(rest_client_requests_total[1m])) by (instance, job)) * 100 > 1`


**Kubernetes client certificate expires next week (instance {{ $labels.instance }})
**
`apiserver_client_certificate_expiration_seconds_count{job="apiserver"} > 0 and histogram_quantile(0.01, sum by (job, le) (rate(apiserver_client_certificate_expiration_seconds_bucket{job="apiserver"}[5m]))) < 7*24*60*60`


**Kubernetes client certificate expires soon (instance {{ $labels.instance }})
**
`apiserver_client_certificate_expiration_seconds_count{job="apiserver"} > 0 and histogram_quantile(0.01, sum by (job, le) (rate(apiserver_client_certificate_expiration_seconds_bucket{job="apiserver"}[5m]))) < 24*60*60`


**Kubernetes API server latency (instance {{ $labels.instance }})
**
`histogram_quantile(0.99, sum(rate(apiserver_request_duration_seconds_bucket{verb!~"(?:CONNECT|WATCHLIST|WATCH|PROXY)"} [10m])) WITHOUT (subresource)) > 1`


**Container killed (instance {{ $labels.instance }})
**
`time() - container_last_seen > 60`


**Container absent (instance {{ $labels.instance }})
**
`absent(container_last_seen)`


**Container High CPU utilization (instance {{ $labels.instance }})
**
`(sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod, container) / sum(container_spec_cpu_quota{container!=""}/container_spec_cpu_period{container!=""}) by (pod, container) * 100) > 80`


**Container High Memory usage (instance {{ $labels.instance }})
**
`(sum(container_memory_working_set_bytes{name!=""}) BY (instance, name) / sum(container_spec_memory_limit_bytes > 0) BY (instance, name) * 100) > 80`


**Container Volume usage (instance {{ $labels.instance }})
**
`(1 - (sum(container_fs_inodes_free{name!=""}) BY (instance) / sum(container_fs_inodes_total) BY (instance))) * 100 > 80`


**Container high throttle rate (instance {{ $labels.instance }})
**
`sum(increase(container_cpu_cfs_throttled_periods_total{container!=""}[5m])) by (container, pod, namespace) / sum(increase(container_cpu_cfs_periods_total[5m])) by (container, pod, namespace) > ( 25 / 100 )`


**Container Low CPU utilization (instance {{ $labels.instance }})
**
`(sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod, container) / sum(container_spec_cpu_quota{container!=""}/container_spec_cpu_period{container!=""}) by (pod, container) * 100) < 20`


**Container Low Memory usage (instance {{ $labels.instance }})
**
`(sum(container_memory_working_set_bytes{name!=""}) BY (instance, name) / sum(container_spec_memory_limit_bytes > 0) BY (instance, name) * 100) < 20`

