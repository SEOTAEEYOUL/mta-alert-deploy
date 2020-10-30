# [Multi-Tenancy Alert]

### 1. Prometheus AlertRule 과 연동
AlertManager 구성파일에 설정하여 사용함(아래의  webhook url)
* monitoring-prometheus-alertmanager-cm.yaml

```
    receivers:
    - name: 'slack-notifications'
      slack_configs:
      - channel: '#mta'
        send_resolved: true
        icon_emoji: ":whale2:"
      webhook_configs:
      - url: 'http://mta-alert.mta-infra.svc.cluster.local:8080/alert'
```

### 2. mta-alert  
#### 2.1 deployment  
- 별도의 구성 파일 없이 env에 변수 형태로 설정하여 사용함
  - 현재 DB 관련 구성파일이 있으나 ConfigMap 구성 예정
- SLACK 정보 및 AWS SNS 정보는 현재 환경 변수로 전달 받게 되어 있음
- 향후 json config 등을 고려 할 수 있음
  - kubernetes configmap 사용시 ConfigMap 변경시 Reload 필요
   
| 환경 변수| 설명 | 내용 |
|:-------- |:---- |:---- |
| SLACK_API_TOKEN | SLACK API TOKEN | "xoxb-1219098448230-1257217978624-Q**********************b" |
| SLACK_WEBHOOK   | SLACK WEBHOOK URL | "https://hooks.slack.com/services/T*********S/B01*******Q/uSs*M*Q*u*u*y*f*O*7*2*G*" |
| SLACK_USER_NAME | bot Name | "Multi-Tenancy Alert" |
| SLACK_CHANNEL   | alert 를 표시할 default Slack 채널 </br> 기본 채널| "#default" |
| ENV             | 개발, 운영 구분 | "dev" or "stg", "prd" |
| AWS_ACCESS_KEY_ID | "AKIAY6----------XJ2L" |
| AWS_SECRET_ACCESS_KEY | "5UpYW1------------------------J/IX93y2yh" | 


#### 2.2 container
```
      - CentOS7/node10 사용
      - server.js - alertmanager 에서 온 json 형태의 alert message 를 읽어 통품감 및 EM 연계 파일 생성
        .통품감 : Critical
        .EM     : warning
```

#### 2.3 서비스 url
```
      - http://www.skmta.net/
        Multi-Tenancy Alert WordCloud 화면- 향후 Alert WordCloud 로 사용 예정
      - http://www.skmta.net/api-docs/
        Multi-Tenancy Alert Swagger 화면
      - http://mta-alert.mta-infra.svc.cluster.local:8080/alert-eks
        외부 연도 webhook
      - http://mta-alert.mta-infra.svc.cluster.local:8080/slack
        slack message 보내는 용
```

#### 2.4 mta-alert log  
```   
$ ki logs -f mta-alert-d57d65cb4-579kj

> mta-alert@1.0.0 start /home/appadmin
> node --max-old-space-size=256 server.js

Running on http://0.0.0.0:8090, metrics exposed on /metrics endpoint
-- apiToken[xoxb-1219098448230-1257217978624-QsnbONsMnzanNACxQBWtkScb]
-- webhookUri[https://hooks.slack.com/services/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]
-- defaultChannel[default]
V8 Total Heap Size 262.38 MB
Running on http://0.0.0.0:8080, booting mta-alert
MongoDB 에 연결되었습니다. :mongodb://skmta:Skmta2020!@mta-mongodb-headless.mta-infra.svc.cluster.local:27017/mta
Successful connected to mongodb : mongodb://skmta:Skmta2020!@mta-mongodb-headless.mta-infra.svc.cluster.local:27017/mta
ACC|192.168.31.35 "-" - - [2020-10-20T00:18:43.028Z] 200_code 300_bytes 12169_usecs "GET /health HTTP/1.1" "undefined" "kube-probe/1.16+"
ACC|192.168.31.35 "-" - - [2020-10-20T00:18:53.010Z] 200_code 300_bytes 2329_usecs "GET /health HTTP/1.1" "undefined" "kube-probe/1.16+"
ACC|192.168.31.35 "-" - - [2020-10-20T00:19:03.008Z] 200_code 300_bytes 1175_usecs "GET /health HTTP/1.1" "undefined" "kube-probe/1.16+"
ACC|192.168.50.24 "-" - - [2020-10-20T00:19:05.261Z] 200_code 4984_bytes 3368_usecs "GET / HTTP/1.1" "undefined" "ELB-HealthChecker/2.0"
ACC|192.168.13.122 "-" - - [2020-10-20T00:19:05.271Z] 200_code 4984_bytes 1827_usecs "GET / HTTP/1.1" "undefined" "ELB-HealthChecker/2.0"
ACC|192.168.65.93 "-" - - [2020-10-20T00:19:05.314Z] 200_code 4984_bytes 1442_usecs "GET / HTTP/1.1" "undefined" "ELB-HealthChecker/2.0"
ACC|192.168.13.122 "-" - - [2020-10-20T00:21:50.286Z] 200_code 4984_bytes 2001.9999999999998_usecs "GET / HTTP/1.1" "undefined" "ELB-HealthChecker/2.0"
2020-10-20T00:22:52.473Z
{ receiver: 'slack-notifications',
  status: 'firing',
  alerts:
   [ { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:47:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=100+%2A+%28count+by%28job%29+%28up+%3D%3D+0%29+%2F+count+by%28job%29+%28up%29%29+%3E+10&g0.tab=1',
       fingerprint: '4c42f070fc8a4276' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:10:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: '6f8f778036cf4d49' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:06:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: '03cc0f2773680568' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:10:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: '4a3c116d579734b5' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:06:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: 'cb1f519728424aca' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:07:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: 'cff6c2bc16caf743' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:09:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: 'd8836952395cd9ce' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:09:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: '022cdaac498528a7' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:10:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: '767a08d9ef64ee84' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:01:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=increase%28kube_pod_container_status_restarts_total%7Bpod%21~%22vulnerability-advisor.%2A%22%7D%5B1h%5D%29+%3E+5&g0.tab=1',
       fingerprint: '502294be80f8c15e' },
     { status: 'resolved',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:11:41.379910692Z',
       endsAt: '2020-10-20T00:15:41.379910692Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=sum_over_time%28kube_pod_container_status_terminated_reason%7Breason%21%3D%22Completed%22%2Creason%21%3D%22OOMKilled%22%7D%5B5m%5D%29+%3E+0&g0.tab=1',
       fingerprint: 'a77f9a649a33274f' },
     { status: 'resolved',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:02:41.379910692Z',
       endsAt: '2020-10-20T00:06:41.379910692Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=sum_over_time%28kube_pod_container_status_terminated_reason%7Breason%21%3D%22Completed%22%2Creason%21%3D%22OOMKilled%22%7D%5B5m%5D%29+%3E+0&g0.tab=1',
       fingerprint: '41f212fe2537016c' },
     { status: 'resolved',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-20T00:10:41.379910692Z',
       endsAt: '2020-10-20T00:19:41.379910692Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=sum_over_time%28kube_pod_container_status_terminated_reason%7Breason%21%3D%22Completed%22%2Creason%21%3D%22OOMKilled%22%7D%5B5m%5D%29+%3E+0&g0.tab=1',
       fingerprint: '6c40585319677770' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:40:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: 'c099cc1017ea60a0' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:41:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: '8544372b87a5dcb6' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:39:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: '05f1323e3ab46f29' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:40:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: '159b244b63835776' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:39:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: 'c311a2dcc094aa87' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:39:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: 'df44bb389a15aeb0' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:40:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: 'b52c7fa6c58bce1b' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:40:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: 'dfd39efa0442ef34' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:41:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: '6774735047985765' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:41:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=rate%28kube_pod_container_status_restarts_total%5B1h%5D%29+%2A+3600+%3E+1&g0.tab=1',
       fingerprint: '04e3df325eacb0f3' },
     { status: 'firing',
       labels: [Object],
       annotations: [Object],
       startsAt: '2020-10-19T23:37:41.379910692Z',
       endsAt: '0001-01-01T00:00:00Z',
       generatorURL:
        'http://prometheus-server-5f8cb867d7-2dfbs:9090/graph?g0.expr=vector%281%29&g0.tab=1',
       fingerprint: '3d3224a282c856a7' } ],
  groupLabels: {},
  commonLabels: {},
  commonAnnotations: {},
  externalURL: 'http://localhost:9093',
  version: '4',
  groupKey: '{}:{}',
  truncatedAlerts: 0 }
{
"receiver": "slack-notifications",
  [
24
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:47:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"monitoring_target_down","job":"kubernetes-pods","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:47:41.379910692Z]
    "annotations": {"description":"12.5% or more of job[*kubernetes-pods*] targets are down.","summary":"Targets are down"}
  },
AlertMappings.selectOne(eks-cluster.mta:default:warning)
selectOne(eks-cluster.mta, default, warning)
AlertList.create 호출
AlertSummary.insert 호출
AlertSummary.plusCount 호출
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:10:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-coordinating-only-575f9657dc-77jk7","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:10:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-77jk7 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
AlertList.create 호출
AlertSummary.insert 호출
AlertSummary.plusCount 호출
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:06:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-coordinating-only-575f9657dc-fww7d","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:06:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-fww7d ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
AlertList.create 호출
AlertSummary.insert 호출
AlertSummary.plusCount 호출
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:10:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-data-0","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:10:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-data-0 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:06:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-data-1","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:06:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-data-1 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:07:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-ingest-f9b98d754-4lksw","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:07:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-4lksw ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:09:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-ingest-f9b98d754-tqgln","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:09:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-tqgln ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:09:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-master-0","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:09:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-master-0 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:10:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-master-1","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:10:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-master-1 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-20T00:01:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"pods_restarting","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"kibana","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-kibana-6c4b844bdb-7n5v8","severity":"warning"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:01:41.379910692Z]
    "annotations": {"description":"재시작 많은 Pod : mta-infra/elasticsearch-kibana-6c4b844bdb-7n5v8 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:warning)
selectOne(eks-cluster.mta, mta-infra, warning)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "resolved",
    "startsAt": "2020-10-20T00:11:41.379910692Z",
    "endsAt": "2020-10-20T00:15:41.379910692Z",
    "labels": {"alertname":"pods_terminated","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-data-0","reason":"Error","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:11:41.379910692Z]
    "annotations": {"description":"비정상 종료 Pod : mta-infra/elasticsearch-elasticsearch-data-0 , 사유 : Error , 인스턴스 : 192.168.47.192:8080 ","summary":"Pod was terminated"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "resolved",
    "startsAt": "2020-10-20T00:02:41.379910692Z",
    "endsAt": "2020-10-20T00:06:41.379910692Z",
    "labels": {"alertname":"pods_terminated","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-data-1","reason":"Error","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:02:41.379910692Z]
    "annotations": {"description":"비정상 종료 Pod : mta-infra/elasticsearch-elasticsearch-data-1 , 사유 : Error , 인스턴스 : 192.168.47.192:8080 ","summary":"Pod was terminated"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "resolved",
    "startsAt": "2020-10-20T00:10:41.379910692Z",
    "endsAt": "2020-10-20T00:19:41.379910692Z",
    "labels": {"alertname":"pods_terminated","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-ingest-f9b98d754-tqgln","reason":"Error","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-20T00:10:41.379910692Z]
    "annotations": {"description":"비정상 종료 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-tqgln , 사유 : Error , 인스턴스 : 192.168.47.192:8080 ","summary":"Pod was terminated"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:40:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"aws-node","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"kube-system","pod":"aws-node-l7jjz","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:40:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : kube-system/aws-node-l7jjz ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:kube-system:info)
selectOne(eks-cluster.mta, kube-system, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:41:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-coordinating-only-575f9657dc-77jk7","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:41:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-77jk7 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:39:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-coordinating-only-575f9657dc-fww7d","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:39:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-fww7d ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:40:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-data-0","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:40:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-data-0 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:39:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-data-1","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:39:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-data-1 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:39:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-ingest-f9b98d754-4lksw","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:39:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-4lksw ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:40:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-ingest-f9b98d754-tqgln","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:40:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-tqgln ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:40:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-master-0","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:40:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-master-0 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:41:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"elasticsearch","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-elasticsearch-master-1","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:41:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-master-1 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:41:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"action_required":"true","alertname":"pod_restart","app_kubernetes_io_instance":"prometheus","app_kubernetes_io_managed_by":"Helm","app_kubernetes_io_name":"kube-state-metrics","container":"kibana","helm_sh_chart":"kube-state-metrics-2.8.13","instance":"192.168.47.192:8080","job":"kubernetes-service-endpoints","kubernetes_name":"prometheus-kube-state-metrics","kubernetes_namespace":"mta-infra","kubernetes_node":"ip-192-168-55-122.ap-northeast-2.compute.internal","namespace":"mta-infra","pod":"elasticsearch-kibana-6c4b844bdb-7n5v8","severity":"info"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:41:41.379910692Z]
    "annotations": {"description":"1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-kibana-6c4b844bdb-7n5v8 ","summary":"Pod restarting a lot"}
  },
AlertMappings.selectOne(eks-cluster.mta:mta-infra:info)
selectOne(eks-cluster.mta, mta-infra, info)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  {
    "status": "firing",
    "startsAt": "2020-10-19T23:37:41.379910692Z",
    "endsAt": "0001-01-01T00:00:00Z",
    "labels": {"alertname":"EKS-MonitoringHeartbeat","severity":"none"},
dateStr: 2020-10-20 09:22:52
startsAt [2020-10-19T23:37:41.379910692Z]
    "annotations": {"description":"This is a Hearbeat event meant to ensure that the entire Alerting pipeline is functional.","summary":"Alerting Heartbeat"}
  },
AlertMappings.selectOne(eks-cluster.mta:default:none)
selectOne(eks-cluster.mta, default, none)
Waiting for available connection slot
AlertList.create 호출
AlertSummary.insert 호출
Waiting for available connection slot
AlertSummary.plusCount 호출
Waiting for available connection slot
  ]
}
ACC|192.168.27.168 "-" - - [2020-10-20T00:22:52.601Z] 200_code 20_bytes 109145_usecs "POST /alert-eks HTTP/1.1" "undefined" "Alertmanager/0.21.0"
Connection 1206 acquired
insert:데이터베이스에 연결 스레드 아이디:1206
Connection 1209 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1209
Connection 1205 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1204 acquired
insert:데이터베이스에 연결 스레드 아이디:1204
Connection 1207 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1207
Connection 1208 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1210 acquired
insert:데이터베이스에 연결 스레드 아이디:1210
Connection 1202 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'default'
   and     level = 'warning'

Connection 1211 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1203 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1203
Connection 1204 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'default', `alert_name` = 'monitoring_target_down', `channel_name` = 'default', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'default',
   alert_name = 'monitoring_target_down',
 channel_name = 'default'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1207 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1206 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1205 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:10:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-coordinating-only-575f9657dc-77jk7`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-77jk7 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1209 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1204 acquired
insert:데이터베이스에 연결 스레드 아이디:1204
Connection 1207 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1207
Connection 1206 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1205 acquired
insert:데이터베이스에 연결 스레드 아이디:1205
Connection 1208 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:06:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-coordinating-only-575f9657dc-fww7d`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-fww7d \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1210 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1204 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1206 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:06:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-data-1`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-data-1 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1208 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1202 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[]
조회된 값의 수 : 0
axios.post :
{
  "username": "[발생] *monitoring_target_down* - 9:22:52 AM\n",
  "channel": "default",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|monitoring_target_down>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|monitoring_target_down>",
      "fields": [
        {
          "title": "*알림명:* `monitoring_target_down`\n",
          "value": "*시간:* _2020-10-19T23:47:41.379910692Z_\n*심각도:* `warning`\n*환경:* *eks-cluster.mta*\n*요약:* Targets are down\n*내용:* 12.5% or more of job[*kubernetes-pods*] targets are down.\n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1211 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:10:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-data-0`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-data-0 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1209 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1209
Connection 1204 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1204
Connection 1206 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1208 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:07:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-ingest-f9b98d754-4lksw`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-4lksw \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1202 acquired
insert:데이터베이스에 연결 스레드 아이디:1202
Connection 1211 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1211
Connection 1210 acquired
insert:데이터베이스에 연결 스레드 아이디:1210
Connection 1206 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:09:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-ingest-f9b98d754-tqgln`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-tqgln \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1208 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1206 acquired
insert:데이터베이스에 연결 스레드 아이디:1206
Connection 1208 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:09:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-master-0`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-master-0 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1203 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'default'
AND   alert_name = 'monitoring_target_down'
AND channel_name = 'default'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1208 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1208
Connection 1203 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1203 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:10:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-master-1`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-master-1 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1207 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1205 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1209 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1204 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1202 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1211 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1210 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1206 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1203 acquired
insert:데이터베이스에 연결 스레드 아이디:1203
Connection 1207 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1207
Connection 1205 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'warning'

Connection 1209 acquired
insert:데이터베이스에 연결 스레드 아이디:1209
Connection 1204 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1204
Connection 1202 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1211 acquired
insert:데이터베이스에 연결 스레드 아이디:1211
Connection 1210 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1210
Connection 1206 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1205 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pods_restarting* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#00cccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:01:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-kibana-6c4b844bdb-7n5v8`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-kibana-6c4b844bdb-7n5v8 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1202 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[해소] *pods_terminated* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":good:",
  "attachments": [
    {
      "color": "#00cc00",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_terminated>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_terminated>",
      "fields": [
        {
          "title": "*알림명:* `pods_terminated`\n",
          "value": "*시간:* _2020-10-20T00:11:41.379910692Z_ ~ _2020-10-20T00:15:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-data-0`\n*환경:* *eks-cluster.mta*\n*요약:* Pod was terminated\n*내용:* 비정상 종료 Pod : mta-infra/elasticsearch-elasticsearch-data-0 , 사유 : Error , 인스턴스 : 192.168.47.192:8080 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1211 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_terminated', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_terminated',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1206 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[해소] *pods_terminated* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":good:",
  "attachments": [
    {
      "color": "#00cc00",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_terminated>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_terminated>",
      "fields": [
        {
          "title": "*알림명:* `pods_terminated`\n",
          "value": "*시간:* _2020-10-20T00:02:41.379910692Z_ ~ _2020-10-20T00:06:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-data-1`\n*환경:* *eks-cluster.mta*\n*요약:* Pod was terminated\n*내용:* 비정상 종료 Pod : mta-infra/elasticsearch-elasticsearch-data-1 , 사유 : Error , 인스턴스 : 192.168.47.192:8080 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1202 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1202
Connection 1211 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1206 acquired
insert:데이터베이스에 연결 스레드 아이디:1206
Connection 1205 acquired
insert:데이터베이스에 연결 스레드 아이디:1205
Connection 1208 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1203 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1211 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[해소] *pods_terminated* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":good:",
  "attachments": [
    {
      "color": "#00cc00",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_terminated>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_terminated>",
      "fields": [
        {
          "title": "*알림명:* `pods_terminated`\n",
          "value": "*시간:* _2020-10-20T00:10:41.379910692Z_ ~ _2020-10-20T00:19:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-ingest-f9b98d754-tqgln`\n*환경:* *eks-cluster.mta*\n*요약:* Pod was terminated\n*내용:* 비정상 종료 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-tqgln , 사유 : Error , 인스턴스 : 192.168.47.192:8080 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1208 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1208
Connection 1203 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'kube-system'
   and     level = 'info'

Connection 1211 acquired
insert:데이터베이스에 연결 스레드 아이디:1211
Connection 1203 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'kube-system',
    category: 'info',
    channel_name: 'kube-system',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, kube-system
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "kube-system",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:40:41.379910692Z_\n*심각도:* `info`\n*namespace:* `kube-system`\n*pod:* `aws-node-l7jjz`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : kube-system/aws-node-l7jjz \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1211 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'kube-system', `alert_name` = 'pod_restart', `channel_name` = 'kube-system', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'kube-system',
   alert_name = 'pod_restart',
 channel_name = 'kube-system'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1210 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_terminated'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1203 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1203
Connection 1211 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1210 acquired
insert:데이터베이스에 연결 스레드 아이디:1210
Connection 1211 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:41:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-coordinating-only-575f9657dc-77jk7`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-77jk7 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1210 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1211 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1211
Connection 1210 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1210 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:39:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-coordinating-only-575f9657dc-fww7d`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-coordinating-only-575f9657dc-fww7d \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1210 acquired
insert:데이터베이스에 연결 스레드 아이디:1210
Connection 1207 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1209 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_restarting', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_restarting',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1207 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1207
Connection 1209 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1209 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:40:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-data-0`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-data-0 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1209 acquired
insert:데이터베이스에 연결 스레드 아이디:1209
Connection 1202 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_terminated'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1206 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_terminated', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_terminated',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1205 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pods_terminated', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pods_terminated',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1202 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1202
Connection 1206 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1205 acquired
insert:데이터베이스에 연결 스레드 아이디:1205
Connection 1206 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:39:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-data-1`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-data-1 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1206 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1206
Connection 1204 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_restarting'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1204 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1204 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:39:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-ingest-f9b98d754-4lksw`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-4lksw \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1204 acquired
insert:데이터베이스에 연결 스레드 아이디:1204
Connection 1203 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'kube-system'
AND   alert_name = 'pod_restart'
AND channel_name = 'kube-system'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1210 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1208 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pods_terminated'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1211 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1203 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1203
Connection 1210 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1208 acquired
insert:데이터베이스에 연결 스레드 아이디:1208
Connection 1211 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1211
Connection 1210 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:40:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-ingest-f9b98d754-tqgln`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-ingest-f9b98d754-tqgln \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1207 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1209 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1210 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1207 acquired
insert:데이터베이스에 연결 스레드 아이디:1207
Connection 1209 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1209
Connection 1210 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:40:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-master-0`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-master-0 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1210 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1210 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:41:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-master-1`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-elasticsearch-master-1 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1210 acquired
insert:데이터베이스에 연결 스레드 아이디:1210
Connection 1202 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1205 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1202 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1202
Connection 1205 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'mta-infra'
   and     level = 'info'

Connection 1205 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'info',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'info' } ]
조회된 값의 수 : 1
axios.post : SLACK, mta-infra
{
  "username": "[발생] *pod_restart* - 9:22:52 AM\n",
  "channel": "mta-infra",
  "icon_emoji": ":whale:",
  "attachments": [
    {
      "color": "#0000cc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pod_restart>",
      "fields": [
        {
          "title": "*알림명:* `pod_restart`\n",
          "value": "*시간:* _2020-10-19T23:41:41.379910692Z_\n*심각도:* `info`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-kibana-6c4b844bdb-7n5v8`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 1 시간 내에 한번 이상 재시작한 Pod : mta-infra/elasticsearch-kibana-6c4b844bdb-7n5v8 \n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1205 acquired
insert:데이터베이스에 연결 스레드 아이디:1205
Connection 1206 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1204 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1206 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1206
Connection 1204 acquired
select cluster, namespace, category, channel_name, channel_type, level
  from alert_mappings
 where   cluster = 'eks-cluster.mta'
   and namespace = 'default'
   and     level = 'none'

Connection 1204 released
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[]
조회된 값의 수 : 0
axios.post :
{
  "username": "[발생] *EKS-MonitoringHeartbeat* - 9:22:52 AM\n",
  "channel": "default",
  "icon_emoji": ":earth_asia:",
  "attachments": [
    {
      "color": "#cccccc",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|EKS-MonitoringHeartbeat>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|EKS-MonitoringHeartbeat>",
      "fields": [
        {
          "title": "*알림명:* `EKS-MonitoringHeartbeat`\n",
          "value": "*시간:* _2020-10-19T23:37:41.379910692Z_\n*심각도:* `none`\n*환경:* *eks-cluster.mta*\n*요약:* Alerting Heartbeat\n*내용:* This is a Hearbeat event meant to ensure that the entire Alerting pipeline is functional.\n",
          "short": false
        }
      ]
    }
  ]
}
Connection 1204 acquired
insert:데이터베이스에 연결 스레드 아이디:1204
Connection 1204 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'default', `alert_name` = 'EKS-MonitoringHeartbeat', `channel_name` = 'default', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'default',
   alert_name = 'EKS-MonitoringHeartbeat',
 channel_name = 'default'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1204 acquired
plusCount:데이터베이스에 연결 스레드 아이디:1204
Connection 1203 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1208 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1204 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'default'
AND   alert_name = 'EKS-MonitoringHeartbeat'
AND channel_name = 'default'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1211 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1207 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1209 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1210 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
Connection 1202 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Connection 1205 released
실행 대상 SQL : INSERT INTO alert_summary SET `date` = '2020102009', `cluster` = 'eks-cluster.mta', `namespace` = 'mta-infra', `alert_name` = 'pod_restart', `channel_name` = 'mta-infra', `count` = 0
ON DUPLICATE KEY UPDATE
         date = '2020102009',
      cluster = 'eks-cluster.mta',
    namespace = 'mta-infra',
   alert_name = 'pod_restart',
 channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
ACC|192.168.31.35 "-" - - [2020-10-20T00:22:53.008Z] 200_code 300_bytes 831_usecs "GET /health HTTP/1.1" "undefined" "kube-probe/1.16+"
Connection 1206 released
실행 대상 SQL : UPDATE alert_summary
  SET count = count + 1
WHERE       date = '2020102009'
AND      cluster = 'eks-cluster.mta'
AND    namespace = 'mta-infra'
AND   alert_name = 'pod_restart'
AND channel_name = 'mta-infra'

OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  serverStatus: 2,
  warningCount: 0,
  message: '(Rows matched: 1  Changed: 1  Warnings: 0',
  protocol41: true,
  changedRows: 1 }
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
Status: 200, Body: ok
.
.
.
AlertMappings.selectOne(eks-cluster.mta:default:none) 수행 결과 :
[ RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: '+8201091540159',
    channel_type: 'SMS',
    level: 'warning' },
  RowDataPacket {
    cluster: 'eks-cluster.mta',
    namespace: 'mta-infra',
    category: 'warning',
    channel_name: 'mta-infra',
    channel_type: 'SLACK',
    level: 'warning' } ]
조회된 값의 수 : 2
sendSMS: +8201091540159
axios.post : SMS, +8201091540159
{
  "username": "[발생] *pods_restarting* - 12:02:52 PM\n",
  "channel": "+8201091540159",
  "icon_emoji": ":spider_web:",
  "attachments": [
    {
      "color": "#ffff00",
      "fallback": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "pretext": "eks-cluster.mta: <https://mta-admin.skmta.net|pods_restarting>",
      "fields": [
        {
          "title": "*알림명:* `pods_restarting`\n",
          "value": "*시간:* _2020-10-20T00:10:41.379910692Z_\n*심각도:* `warning`\n*namespace:* `mta-infra`\n*pod:* `elasticsearch-elasticsearch-data-0`\n*환경:* *eks-cluster.mta*\n*요약:* Pod restarting a lot\n*내용:* 재시작 많은 Pod : mta-infra/elasticsearch-elasticsearch-data-0 \n",
          "short": false
        }
      ]
    }
  ]
}



```
