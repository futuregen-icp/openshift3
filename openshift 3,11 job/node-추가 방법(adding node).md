## Node 추가 방법 

	openshift_is_atomic=true 에 대한 설정이 되어 있는 지 확인 없다면 추가 
	atomic을 사용하였는지 아닌지 확인하여  true|false
	
	노드의 호스트네임이 아니라 크러스터의 호스트 네임으로 할 것 
	openshift_master_cluster_hostname	
	openshift_master_cluster_public_hostname
    ex) ocp-api.ocp3-11.fu.te

### inventory file 수정 
기존 설치한 inventory file 수정 


	###########################################################################
	### OpenShift Hosts
	###########################################################################
	[OSEv3:children]
	masters
	new_masters
	etcd
	nodes
	new_nodes
	lb

	[masters]
	master01.ocp3-11.fu.te openshift_ip=192.168.40.151 openshift_public_ip=192.168.40.151 openshift_public_hostname=master01.ocp3-11.fu.te

	[new_masters]
	master02.ocp3-11.fu.te openshift_ip=192.168.40.152 openshift_public_ip=192.168.40.152 openshift_public_hostname=master02.ocp3-11.fu.te openshift_node_group_name='node-config-master'

	[etcd]
	master01.ocp3-11.fu.te openshift_ip=192.168.40.151 openshift_public_ip=192.168.40.151 openshift_public_hostname=master01.ocp3-11.fu.te

	[nodes]
	## Master
	master01.ocp3-11.fu.te openshift_ip=192.168.40.151 openshift_public_ip=192.168.40.151 openshift_public_hostname=master01.ocp3-11.fu.te

	### Infra
	infra01.ocp3-11.fu.te openshift_ip=192.168.40.181 openshift_public_ip=192.168.40.181  openshift_public_hostname=infra01.ocp3-11.fu.te openshift_node_group_name='node-config-infra'

	### Node
	worker01.ocp3-11.fu.te openshift_ip=192.168.40.221 openshift_public_ip=192.168.40.221 openshift_public_hostname=worker01.ocp3-11.fu.te openshift_node_group_name='node-config-compute'
	worker02.ocp3-11.fu.te openshift_ip=192.168.40.222 openshift_public_ip=192.168.40.222 openshift_public_hostname=worker02.ocp3-11.fu.te openshift_node_group_name='node-config-compute'

	[new_nodes]
	## Master
	master02.ocp3-11.fu.te openshift_ip=192.168.40.152 openshift_public_ip=192.168.40.152 openshift_public_hostname=master02.ocp3-11.fu.te openshift_node_group_name='node-config-master'

	[lb]
	lb01.ocp3-11.fu.te


### Master  node  add 
group name check 및  config


	ansible-playbook -i /opt/ocp3-11/hosts.v01-lb01-master playbooks/openshift-master/openshift_node_group.yml 

node scaleup

	ansible-playbook -i /opt/ocp3-11/hosts.v01-lb01-master playbooks/openshift-master/scaleup.yml
	
### worker node  add 
group name check 및  config

	ansible-playbook -i /opt/ocp3-11/hosts.v01-lb01-node playbooks/openshift-master/openshift_node_group.yml 

node scaleup

	ansible-playbook -i /opt/ocp3-11/hosts.v01-lb01-node  playbooks/openshift-node/scaleup.yml
