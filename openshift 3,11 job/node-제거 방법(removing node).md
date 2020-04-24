## 노드 제거 방법 
### node schedule 중지 

	oc adm manage-node  worker03.ocp3-11.fu.te  --schedulable=false

### pod 이동하기 
replication controllers를 지원 하는 pod만 가능 

	oc adm drain <node> [--pod-selector=<pod_selector>] [--grace-period=-1] [--delete-local-data=true]
    [--ignore-daemonsets]
    
    oc adm drain worker03.ocp3-11.fu.te  --force=true --grace-period=-1 --delete-local-data=true --ignore-daemonsets

--force=true				 : 강제로 이동  
--grace-period=-1        : 각포드의 종료 시간  
--timeout=5s             : 대기시간  
--delete-local-data=true : emptyDir등 로칼 데이타 삭제  
--ignore-daemonsets      : daemonsets은 무시  

	worker node :  node-exporter-54882, sync-jvcmz, ovs-t92vw, sdn-6cx9p
	master node :  node-exporter-2dmqg, sync-kd4cd, ovs-9n6vp, sdn-x6f44

    

### node 삭제 

	oc delete node worker03.ocp3-11.fu.te

### uninstall

invetory file
	
	-- node --
	[OSEv3:vars]
	##-------------------------------------------------------------------------
	## Ansible
	##-------------------------------------------------------------------------
	ansible_user=root
	ansible_become=yes
	debug_level=2
	
	[OSEv3:children]
	nodes
	
	[nodes]
	worker03.ocp3-11.fu.te openshift_ip=192.168.40.223 openshift_public_ip=192.168.40.223 openshift_public_hostname=worker03.ocp3-11.fu.te openshift_node_group_name='node-config-compute'

	-- master --
	[OSEv3:vars]
	##-------------------------------------------------------------------------
	## Ansible
	##-------------------------------------------------------------------------
	ansible_user=root
	ansible_become=yes
	debug_level=2

	[OSEv3:children]
	masters
	nodes
	
	[masters]
	master02.ocp3-11.fu.te openshift_ip=192.168.40.152 openshift_public_ip=192.168.40.152 openshift_public_hostname=master02.ocp3-11.fu.te
	
	[nodes]
	master02.ocp3-11.fu.te openshift_ip=192.168.40.152 openshift_public_ip=192.168.40.152 openshift_public_hostname=master02.ocp3-11.fu.te openshift_node_group_name='node-config-master'


commend

	-- node --
	ansible-playbook -i /opt/ocp3-11/hosts.v01-delete-node.v01 playbooks/adhoc/uninstall.yml
	-- master --
	ansible-playbook -i /opt/ocp3-11/hosts.v01-delete-master.v01 playbooks/adhoc/uninstall.yml
 
	
 

## etcd 제거 방법 
### failed member identifier 확인 
	
	etcdctl -C https://192.168.40.151:2379 \
	  --ca-file=/etc/etcd/ca.crt     \
	  --cert-file=/etc/etcd/peer.crt     \
	  --key-file=/etc/etcd/peer.key cluster-health

	member 9d577910354c663 is healthy: got healthy result from https://192.168.40.151:2379
	member bdd51206ddad9ca3 is healthy: got healthy result from https://192.168.40.152:2379

### failed member remove
	
	  etcdctl -C https://192.168.40.151:2379 \
	    --ca-file=/etc/etcd/ca.crt     \
	    --cert-file=/etc/etcd/peer.crt     \
	    --key-file=/etc/etcd/peer.key member remove bdd51206ddad9ca3

### 데이타 삭제
	
	 rm -rf /var/lib/etcd/*