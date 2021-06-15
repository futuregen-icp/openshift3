## console,web-console 파드 이상 새결 방법

### 증상 

1. Master node 생성되는 console pod 3개중 1개가 CrashLoopBackOff 발생

2. Master node 생성되는 webconsole pod 3개중 1개가 CrashLoopBackOff 발생



### 해결 방법

1. oc 명령으로 강제 종료를 통한 셀프 힐링이 가능할 것으로 보이나 SR 진행중 

```
         oc delete pod webconsole-7fc8468cb-gr9j9 -n openshift-monitoring --grace-period=0 --force
         oc delete pod console-65d9895fd5-xq9f9-n openshift-monitoring --grace-period=0 --force
```

​     2.   다른 방법  

 

클러스터 master에서 구동되는 console, webconsole 파드의 상태가 3개 중 1개 CrashLoopBackOff 발생하여 지속적인 restart가 되고 있는 증상에 대해

1>  아래의 버그로 리포팅 되어 있음. [Bug 1689690]

​     [버그]

​          Bug 1689690 - starter-us-east-2 & 2a experienced outage because kube ip not  present in master iptables: tcp 172.30.0.1:443: getsockopt: no route to host [1]

​      [오류메세지]

 		web-console pod에서 "Error: Get https://172.130.0.1:443/.well-known/oauth-authorization-server: dial tcp 172.130.0.1:443: getsockopt: no route to host"

​      [버그 내용]

 		iptables가 정상동작 하지 않거나 iptables update 문제로 인해 kubenetes api service가  정상적으로 등록되어 있지 않았을때 발생하는 케이스로 패치는 없으며 관련된 services들을 재시작하여 복구.

​      [복구 절차]

   	  Steps taken to get this cluster working again:
   	
   	     # mv /etc/sysconfig/iptables /etc/sysconfig/iptables.containsbak
   	     # systemctl stop atomic-openshift-node
   	     # systemctl stop docker



 	    # systemctl stop iptables
 	   	# systemctl disable iptables
 	   	# systemctl mask iptables



 	    # iptables -F
 	   	# iptables -tnat –flush



 ```
      # systemctl start docker
      # systemctl start atomic-openshift-node
 ```

​	     

 	# delete pods for ovs & sdn for affected master
 		*wait a few minutes*

   eventually, `iptables-save | grep 172.30.0.1/` begins returning results => grep으로  보여지지 않는다면 해당 master reboot 및 router restart가 필요

 

2>  위의 iptables에서 특별한 문제가 발견되지 않는다면 다른 사례로  dnsmasq service restart로 해결된 케이스.

​      \# systemctl restart dnsmasq.service

​     이후 SDN pods와 web console pod restart 수행

  

관련 링크 

[1] https://bugzilla.redhat.com/show_bug.cgi?id=1689690

[2] https://access.redhat.com/solutions/2045283

