## Promethus Pod  상태 이상 해결 방법 

### 증상 

1. Promethus-k8s-0, Promethus-k8s-1 두 pod를 구성하는 일부 container에서 OOM ERROR 발생

2. Promethus에 메모리를 추가로 할당하기 전 Pod 재구동 시도

3. oc delete pod Promethus-k8s-0 -n openshift-monitoring , oc delete pod Promethus-k8s-1 -n openshift-monitoring 시도이후 아래와 같은 상태가 유지됨 



```
NAME            READY STATUS RESTARTS     AGE 
prometheus-k8s-0     0/4    Terminating    0          1y
prometheus-k8s-1     0/4   Terminating    0          1y

or 

NAME            READY STATUS RESTARTS     AGE 
prometheus-k8s-0     3/4    Terminating    0          1y
prometheus-k8s-1     3/4   Terminating    0          1y
```



### 해결 방안



1. oc 명령으로 강제 종료 

```
       oc delete pod prometheus-k8s-0 -n openshift-monitoring --grace-period=0 --force
       oc delete pod prometheus-k8s-1 -n openshift-monitoring --grace-period=0 --force
```

​      

 api를 이용하여 강제 종료 

처음 방법으로 종료가 되지 않으면 kubernet 이슈로 판단되며  api를 통하여 강제로 종료 합니다.

```
-    <token>은  oc whoami -t 명령으로 생성되는 토큰을 이용하시면 됩니다
        제약사항 
 			해당 네임스페이스에 대한 admin 계정이어야 합니다.
			 System:admin 은 사용할 수 없습니다.
 
-    아래는 api를 이용하여 pod를 종료하는 명령입니다.
         echo '{ "propagationPolicy": "Background" }' | curl -k -X DELETE -d @-  -H "Authorization: Bearer <token>" -H 'Accept: application/json' -H 'Content-Type: application/json'  https://<master_URL>:6443/api/v1/namespaces/openshift-monitoring/pods/prometheus-k8s-0
         echo '{ "propagationPolicy": "Background" }' | curl -k -X DELETE -d @-  -H "Authorization: Bearer <token>" -H 'Accept: application/json' -H 'Content-Type: application/json'  https://<master_URL>:6443/api/v1/namespaces/openshift-monitoring/pods/prometheus-k8s-1
```









