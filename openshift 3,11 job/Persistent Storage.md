## Persistent Storage

pv 설정시 참고 사항 

	capacity:
	  storage:   5Gi -사용량 설정 

	accessModes : ReadWriteMany - 여러개의 컨테이너에서만 사용  
				  ReadOnlyMany - 여러개의 컨테이너에서만 사용 
				  ReadWriteOnce - 하나의 컨테이너에서만 사용 

    persistentVolumeReclaimPolicy (볼륨 반환 정책을 나타냄)
			Retain(보존) -- 수동 반환
			Recycle(재활용) -- 기본 스크럽 (rm -rf /thevolume/*)
			Delete(삭제) -- AWS EBS, GCE PD, Azure Disk 또는 OpenStack Cinder 볼륨과 
							같은 관련 스토리지 자산이 삭제됨

현재 NFS 및 HostPath만 재활용을 지원한다. AWS EBS, GCE PD, Azure Disk 및 Cinder 볼륨은 삭제를 지원한다.

### NFS strage

***df -h*** 

	```
	nfs.ocp3-11.fu.te:/ocp/Logging  500G   26G  474G   6% /ocp/Logging
	nfs.ocp3-11.fu.te:/ocp/volumes  500G   26G  474G   6% /ocp/volumes
	nfs.ocp3-11.fu.te:/ocp/Metrics  500G   26G  474G   6% /ocp/Metrics
	```

***nfs server 설정***

	nfs server 설정 
	# cat /etc/exports
	/ocp/volumes *(rw,async,all_squash)
	/ocp/Logging *(rw,async,all_squash)
	/ocp/Metrics *(rw,async,all_squash)	

	selinux 설정 
	# setsebool -P virt_use_nfs 1

	사용할 디렉토리 권한 설정 
	mkdir /ocp/Logging/logging-es-0
	chown nfsnobody.nfsnobody /ocp/Logging/logging-es-0 -R

***pv config yaml***

	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: logging-es-0 
	spec:
	  capacity:
	    storage: 5Gi 
	  accessModes:
	  - ReadWriteMany 
	  nfs: 
	    path: /ocp/Logging/logging-es-0
	    server: nfs.ocp3-11.fu.te 
	  persistentVolumeReclaimPolicy: Retain

***pvc config yaml***

	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: logging-es-0
	spec:
	  accessModes:
	    - ReadWriteMany 
	  resources:
	    requests:
	      storage: 1Gi

 
※ pvc는 자신에게 설정된 storage  size 보다 같거나 큰 볼륨을 bindding 한다.
  이름이 동일하다고 하여    bindding 하는 것은 아니다.

※ openshift 3.x 버전에서는 모든 노트에  nfs가 마운트 되어 있어야야 한다.

***특정 pv와 pvc를 연결 할 경우***

	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: logging-es-0
	spec:
	  accessModes:
	  - ReadWriteOnce
	  capacity:
	    storage: 500Gi
	  claimRef:
	    apiVersion: v1
	    kind: PersistentVolumeClaim
	    name: logging-es-0
	    namespace: default
	  nfs:
	    path: /ocp/Logging/logging-es-0
	    server: nfs.ocp3-11.fu.te
	  persistentVolumeReclaimPolicy: Retain


***nfs volume 사용시 pod 설정***  

	apiVersion: v1
	kind: Pod
	metadata:
	  name: hello-nfs-pod 
	  labels:
	    name: hello-nfs-pod
	spec:
	  containers:
	    - name: hello-nfs-pod
	      image: openshift/hello-openshift 
	      ports:
	        - name: web
	          containerPort: 80
	      volumeMounts:
	        - name: nfsvol 
	          mountPath: /usr/share/nginx/html 
	  securityContext:
	      runAsUser: 65534
	      supplementalGroups: [100003] 
	      privileged: false
	  volumes:
	    - name: nfsvol
	      persistentVolumeClaim:
	        claimName: logging-es-0


```

	runAsUser: 65534
           nfsnobody의 uid가 65534 이다 
           pod 구동을 위의 아이디로 하거나 nfs 볼륨을 특정 UID로 하면 볼륨 사용 권한문제 해결 가능하다
           permition을 777로 구성하는 경우 볼륨 사용 권한문제 해결 가능하다

	supplementalGroups: [100003]  
           nfs volume의 GID를 100003줄 경우 구동되는 Pod와 동일한 권한 임으로 그룹권한이 7일 경우 권한문제 해결 가능하다
  
```

### Gluster strage

nfs 처럼 네트워크를 통해 볼륨을 공유하여 사용하지만  Gluster는 분산 구조를 지향하기에 
endpoint에 대한 정보가 있어야 한다.

***서비스 및 endpoint 설정***

	cat <<EOF > gluster-endpoints.yaml
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: glusterfs-cluster 
	spec:
	  ports:
	  - port: 1
	---
	apiVersion: v1
	kind: Endpoints
	metadata:
	  name: glusterfs-cluster 
	subsets:
	  - addresses:
	      - ip: 192.168.1.91 
	    ports:
	      - port: 1 
	  - addresses:
	      - ip: 192.168.1.92 
	    ports:
	      - port: 1 
	  - addresses:
	      - ip: 192.168.1.93 
	    ports:
	      - port: 1 
	EOF

***서비스 및 endpoint 생성***

	$ oc create -f gluster-endpoints.yaml


***서비스 및 endpoint 확인***

	$ oc get services
	NAME                       CLUSTER_IP       EXTERNAL_IP   PORT(S)    SELECTOR        AGE
	glusterfs-cluster          172.30.205.34    <none>        1/TCP      <none>          44s

	$ oc get endpoints
	NAME                ENDPOINTS                                               AGE
	docker-registry     10.1.0.3:5000                                           4h
	glusterfs-cluster   192.168.1.91:1,192.168.1.92:1,192.168.1.93:1   			11s
	kubernetes          172.16.35.3:8443                                        4d

***pre- set***

	$ mkdir -p /mnt/glusterfs/myVol1
	$ mount -t glusterfs 192.168.122.221:/myVol1 /mnt/glusterfs/myVol1
	$ ls -lnZ /mnt/glusterfs/
	drwxrwx---. 592 590 system_u:object_r:fusefs_t:s0    myVol1

※ 볼륨 엑세스에 필요한 GiD와 그에 상응하는 권한 확인   

***pv***

	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: gluster-default-volume 
	  annotations:
	    pv.beta.kubernetes.io/gid: "590" 
	spec:
	  capacity:
	    storage: 2Gi 
	  accessModes: 
	    - ReadWriteMany
	  glusterfs:
	    endpoints: glusterfs-cluster 
	    path: myVol1 
	    readOnly: false
	  persistentVolumeReclaimPolicy: Retain

***pvc***

	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: gluster-claim  
	spec:
	  accessModes:
	  - ReadWriteMany      
	  resources:
	     requests:
	       storage: 1Gi    


### local volume (hostpath)

***pv:claimRef***

	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: postgresql
	  labels:
	    type: local
	spec:
	  accessModes:
	    - ReadWriteOnce
	  capacity:
	    storage: 1Gi
	  claimRef:
	    apiVersion: v1
	    kind: PersistentVolumeClaim
	    name: postgresql
	    namespace: myproject
	  hostPath:
	    path: "/mnt/data"

***pv***

	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: postgresql 
	  labels:
	    type: local
	spec:
	  storageClassName: manual 
	  capacity:
	    storage: 1Gi
	  accessModes:
	    - ReadWriteOnce 
	  persistentVolumeReclaimPolicy: Retain
	  hostPath:
	    path: "/mnt/data"

***pvc***

	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: postgresql
	spec:
	  accessModes:
	    - ReadWriteOnce
	  resources:
	    requests:
	      storage: 1Gi
	  storageClassName: manual