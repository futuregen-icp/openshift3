## 사용자 인증 보안 정책 

	
	아래의  roles은 `oc get clusterroles`로 확인 가능하며 
    주로 사용되는 권한을  
	
	- openshift user   : web-console 또는 commend line명령을 위해 로그인을 하기 위한 계정  
	- servive acount   : 사용자의 로그인등 관여 없이 내부적인 API를 호출하기 위한 계정 
	                     pod 복제 및 삭제 검색 등을 위한 API 호출
	- security context : pod가 리소스에 접근 할 수 있는법위 설정   


### 클러스터 보안정책

기본역활|설명  
--------|-----  
cluster-admimn|role를 가진 사용자는 클러스터관리 가능  
cluster-status|role를 가진 사용자는 클러스터 정보 엑세스 가능
self-provisioner|새로운 프로젝트 생성 가능  

**정책 확인**  
oc get clusterrolebinding --all-namespaces

**사용자에게 클러스터 관리자 권한 추가**  
oc adm policy add-cluster-role-to-user cluster-admin <username>
 
**사용자에게 클러스터 관리자 권한 제거**  
oc adm policy remove-cluster-role-from-user cluster-admin <username>

**사용자에게 클러스터 관리자 권한 추가**  
oc adm policy add-cluster-role-to-user cluster-role <username>

**사용자에게 클러스터 관리자 권한 추가**  
oc adm policy remove-cluster-role-from-user cluster-role <username>

### 사용자 보안 정책
 
기본역활 | 설명  
--------|-----  
admin|프로젝트의 모든 권한관리  
edit|특정 프로젝트에서 서비스및 배포 구성 및 일반 어플리케이션 제작 변경 삭제 가능  
vew|수정할 수 없지만 프로젝트에서 대부분의 객체를 볼 수있는 사용자입니다. 역할이나 바인딩을 보거나 수정할 수 없습니다.  
basic-user| 프로젝트에 대한 읽기 엑세스 권한   
  
**정책 확인** 
oc get rolebinding --all-namespaces

**사용자에게 관리자 권한 추가**  
oc adm policy add-role-to-user admin <username>

**사용자에게 관리자 권한 제거**  
oc adm policy remove-role-to-user admin <username>

## Service Acount
web-console 처럼 일반적인 사용자 자격증명이 불가능할 겅우 API를 독립적으로 호출하도록 하기위한 계정  
 - pod 복제를 위한 replica set 호출  
 - 내부 응용프로그램 조회를 위해 호출시  
 - 외부 모니터링등의 연동을 위한 호출시 
 - 일반 계정과 연동을 위해 서비스 계정을 위한 자격 증명   

프로젝트 생성시 기본적으로 생성되는 Service Acount

Service Acount|내용 
--------------|----
builder|Pod 빌드용으로 사용
deployer|Pod 배포용으로 사용
default|기본으로 사용되는 Service Account

## 보안 컨텍스트 (SCC)

	NAME                 PRIV      CAPS      SELINUX     RUNASUSER          FSGROUP     SUPGROUP    PRIORITY   READONLYROOTFS   VOLUMES
	anyuid               false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    10         false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
	hostaccess           false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath persistentVolumeClaim projected secret]
	hostmount-anyuid     false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath nfs persistentVolumeClaim projected secret]
	hostnetwork          false     []        MustRunAs   MustRunAsRange     MustRunAs   MustRunAs   <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
	kube-state-metrics   false     []        RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
	node-exporter        false     []        RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
	nonroot              false     []        MustRunAs   MustRunAsNonRoot   RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
	privileged           true      [*]       RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
	restricted           false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]

**openshift에서는 기본적으로 restricted scc를 이용하여 pod 생성** 

**다른 권한이 필요하다면 서비스 계정에 매핑하여 사용** 

	oc create serviceaccount <service-name>
	oc adm policy add-scc-to-user SCC -z serviceaccount

**컨테이너의 제약 사항을 확인** 

	oc export pod pod-name > output.yaml
	oc adm policy scc-suject-review -f output.yaml

    RESOURCE              ALLOWED BY
	Pod/ruby-ex-6-build   privileged

**root로 구동되어야 하는 pod가 있는 경우**

    oc create serviceaccount useroot
	oc get scc restricted --export -o yaml > runasroot.yaml
	edit runasroot.yaml
	```
	[...]
	allowPrivilegedContainer:  true
	[...]
	runAsUser:
  	  type: RunAsAny
	[...]
	```
    oc create -f  runasroot.yaml
    oc adm policy add-scc-to-user runasroot -z useroot
    scc "runasroot" added to: ["system:serviceaccount:kube-system:useroot"]



    pod deployment spec section
	spec:
      containers:
      - image: xxxxx/myproject/myapp:latest
        imagePullPolicy: Always
        name: myapp
        resources: {}
        securityContext:   <-- Add
          privileged: true  <---- this
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: 
        runAsUser: 0     <-- Add
      serviceAccount: useroot <-- Add
      serviceAccountName: useroot   <--- this
      terminationGracePeriodSeconds: 30