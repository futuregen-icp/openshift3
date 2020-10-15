### openshift 3.11 인증서 갱신 작업 

## 개요 

```
Openshift는 ssl 접속을 위해 인증서를 사용한다
masters (API server and controllers)
etcd
nodes
registry
router
````

### 인증서 만료 기간확인 방법 

```
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -v -i <inventory_file> playbooks/openshift-checks/certificate_expiry/easy-mode.yaml
```

**주의 점(?)**
```
logging , monitoring, service-catalog는  manaual 하게 갱신해야 한다
```

## 인증서 갱신 ##

    인증서 갱신시 재정의 가능하다 (해보지 않음)
	- openshift_public_hostname
	- openshift_public_ip
	- openshift_master_cluster_hostname
	- openshift_master_cluster_public_hostname

### Redeploying All Certificates Using the Current OpenShift Container Platform and etcd CA
	
	범위 
	etcd
	master services
	node services

```
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -i <inventory_file> playbooks/redeploy-certificates.yml
```

### Redeploying Master Certificates Only

```
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -i <inventory_file> playbooks/openshift-master/redeploy-certificates.yml
```

### Redeploying etcd Certificates Only

```
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -i <inventory_file> playbooks/openshift-etcd/redeploy-certificates.yml
```

### Redeploying Node Certificates

	- 노드 인증서는 자동으로 갱신된며 

### Redeploying Registry or Router Certificates Only

**Redeploying Registry Certificates Only **

```
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -i <inventory_file>  playbooks/openshift-hosted/redeploy-registry-certificates.yml
```

**Redeploying Router Certificates Only **

```
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -i <inventory_file> playbooks/openshift-hosted/redeploy-router-certificates.yml
````
