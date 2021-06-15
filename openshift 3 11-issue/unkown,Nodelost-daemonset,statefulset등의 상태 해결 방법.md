## unkown,Nodelost-daemonset,statefulset등의 상태 해결 방법

### 증상

1. 특정 노드에 속해 있는 Pod의 상태가 Unknown/ NodeLost로 표시 됩니다

2. DaemonSet의 경우 스케줄에 할당되어 있어 추가 생성 되지 않음 

3. Pod가 추가로 생성되는 경우 자원 경쟁에 의해 새로 생성되는 Pod는 Pending상태로 됩니다.



### 해결 방안

1>  atomic-openshift-node 서비스 재시작 

```
$  systemctl restart atomic-openshift-node
```



2>  위의 방법으로 정상적으로 구동되지 않는다면 아래의 절차 대로 노드 재부팅을 진행합니다.

```
$ oc adm manage-node <node1> <node2> --schedulable=false   // node unschedulable로 변경
$ oc delete pods <pod> --grace-period=0 --force  // pod들 삭제
$ systemctl stop atomic-openshift-node
$ restart VM or instance
$ oadm manage-node <node name> --schedulable=true
```

