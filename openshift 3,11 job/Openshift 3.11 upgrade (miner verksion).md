# Openshift 3.11 upgrade (miner verksion)

## 사전 작업 



### 이미지  mirror

**imgae pull**

```
tags="v3.11.439 v3.11.219 v3.11.161"

for tag in `echo $tags` ; do
docker pull registry.redhat.io/openshift3/apb-base:$tag
docker pull registry.redhat.io/openshift3/apb-tools:$tag
docker pull registry.redhat.io/openshift3/automation-broker-apb:$tag
docker pull registry.redhat.io/openshift3/csi-attacher:$tag
docker pull registry.redhat.io/openshift3/csi-driver-registrar:$tag
docker pull registry.redhat.io/openshift3/csi-livenessprobe:$tag
docker pull registry.redhat.io/openshift3/csi-provisioner:$tag
docker pull registry.redhat.io/openshift3/grafana:$tag
docker pull registry.redhat.io/openshift3/kuryr-controller:$tag
docker pull registry.redhat.io/openshift3/kuryr-cni:$tag
docker pull registry.redhat.io/openshift3/local-storage-provisioner:$tag
docker pull registry.redhat.io/openshift3/manila-provisioner:$tag
docker pull registry.redhat.io/openshift3/mariadb-apb:$tag
docker pull registry.redhat.io/openshift3/mediawiki:$tag
docker pull registry.redhat.io/openshift3/mediawiki-apb:$tag
docker pull registry.redhat.io/openshift3/mysql-apb:$tag
docker pull registry.redhat.io/openshift3/ose-ansible-service-broker:$tag
docker pull registry.redhat.io/openshift3/ose-cli:$tag
docker pull registry.redhat.io/openshift3/ose-cluster-autoscaler:$tag
docker pull registry.redhat.io/openshift3/ose-cluster-capacity:$tag
docker pull registry.redhat.io/openshift3/ose-cluster-monitoring-operator:$tag
docker pull registry.redhat.io/openshift3/ose-console:$tag
docker pull registry.redhat.io/openshift3/ose-configmap-reloader:$tag
docker pull registry.redhat.io/openshift3/ose-control-plane:$tag
docker pull registry.redhat.io/openshift3/ose-deployer:$tag
docker pull registry.redhat.io/openshift3/ose-descheduler:$tag
docker pull registry.redhat.io/openshift3/ose-docker-builder:$tag
docker pull registry.redhat.io/openshift3/ose-docker-registry:$tag
docker pull registry.redhat.io/openshift3/ose-efs-provisioner:$tag
docker pull registry.redhat.io/openshift3/ose-egress-dns-proxy:$tag
docker pull registry.redhat.io/openshift3/ose-egress-http-proxy:$tag
docker pull registry.redhat.io/openshift3/ose-egress-router:$tag
docker pull registry.redhat.io/openshift3/ose-haproxy-router:$tag
docker pull registry.redhat.io/openshift3/ose-hyperkube:$tag
docker pull registry.redhat.io/openshift3/ose-hypershift:$tag
docker pull registry.redhat.io/openshift3/ose-keepalived-ipfailover:$tag
docker pull registry.redhat.io/openshift3/ose-kube-rbac-proxy:$tag
docker pull registry.redhat.io/openshift3/ose-kube-state-metrics:$tag
docker pull registry.redhat.io/openshift3/ose-metrics-server:$tag
docker pull registry.redhat.io/openshift3/ose-node:$tag
docker pull registry.redhat.io/openshift3/ose-node-problem-detector:$tag
docker pull registry.redhat.io/openshift3/ose-operator-lifecycle-manager:$tag
docker pull registry.redhat.io/openshift3/ose-ovn-kubernetes:$tag
docker pull registry.redhat.io/openshift3/ose-pod:$tag
docker pull registry.redhat.io/openshift3/ose-prometheus-config-reloader:$tag
docker pull registry.redhat.io/openshift3/ose-prometheus-operator:$tag
docker pull registry.redhat.io/openshift3/ose-recycler:$tag
docker pull registry.redhat.io/openshift3/ose-service-catalog:$tag
docker pull registry.redhat.io/openshift3/ose-template-service-broker:$tag
docker pull registry.redhat.io/openshift3/ose-tests:$tag
docker pull registry.redhat.io/openshift3/ose-web-console:$tag
docker pull registry.redhat.io/openshift3/postgresql-apb:$tag
docker pull registry.redhat.io/openshift3/registry-console:$tag
docker pull registry.redhat.io/openshift3/snapshot-controller:$tag
docker pull registry.redhat.io/openshift3/snapshot-provisioner:$tag
docker pull registry.redhat.io/rhel7/etcd:3.2.28

docker pull registry.redhat.io/openshift3/metrics-cassandra:$tag
docker pull registry.redhat.io/openshift3/metrics-hawkular-metrics:$tag
docker pull registry.redhat.io/openshift3/metrics-hawkular-openshift-agent:$tag
docker pull registry.redhat.io/openshift3/metrics-heapster:$tag
docker pull registry.redhat.io/openshift3/metrics-schema-installer:$tag
docker pull registry.redhat.io/openshift3/oauth-proxy:$tag
docker pull registry.redhat.io/openshift3/ose-logging-curator5:$tag
docker pull registry.redhat.io/openshift3/ose-logging-elasticsearch5:$tag
docker pull registry.redhat.io/openshift3/ose-logging-eventrouter:$tag
docker pull registry.redhat.io/openshift3/ose-logging-fluentd:$tag
docker pull registry.redhat.io/openshift3/ose-logging-kibana5:$tag
docker pull registry.redhat.io/openshift3/prometheus:$tag
docker pull registry.redhat.io/openshift3/prometheus-alertmanager:$tag
docker pull registry.redhat.io/openshift3/prometheus-node-exporter:$tag
docker pull registry.redhat.io/cloudforms46/cfme-openshift-postgresql
docker pull registry.redhat.io/cloudforms46/cfme-openshift-memcached
docker pull registry.redhat.io/cloudforms46/cfme-openshift-app-ui
docker pull registry.redhat.io/cloudforms46/cfme-openshift-app
docker pull registry.redhat.io/cloudforms46/cfme-openshift-embedded-ansible
docker pull registry.redhat.io/cloudforms46/cfme-openshift-httpd
docker pull registry.redhat.io/cloudforms46/cfme-httpd-configmap-generator
docker pull registry.redhat.io/rhgs3/rhgs-server-rhel7
docker pull registry.redhat.io/rhgs3/rhgs-volmanager-rhel7
docker pull registry.redhat.io/rhgs3/rhgs-gluster-block-prov-rhel7
docker pull registry.redhat.io/rhgs3/rhgs-s3-server-rhel7

## s2i, 및 기타 application base 이미지로 보이는데 아래의 이미지도 필요한지 확인이 필요합니다.
docker pull registry.redhat.io/jboss-amq-6/amq63-openshift:$tag
docker pull registry.redhat.io/jboss-datagrid-7/datagrid71-openshift:$tag
docker pull registry.redhat.io/jboss-datagrid-7/datagrid71-client-openshift:$tag
docker pull registry.redhat.io/jboss-datavirt-6/datavirt63-openshift:$tag
docker pull registry.redhat.io/jboss-datavirt-6/datavirt63-driver-openshift:$tag
docker pull registry.redhat.io/jboss-decisionserver-6/decisionserver64-openshift:$tag
docker pull registry.redhat.io/jboss-processserver-6/processserver64-openshift:$tag
docker pull registry.redhat.io/jboss-eap-6/eap64-openshift:$tag
docker pull registry.redhat.io/jboss-eap-7/eap71-openshift:$tag
docker pull registry.redhat.io/jboss-webserver-3/webserver31-tomcat7-openshift:$tag
docker pull registry.redhat.io/jboss-webserver-3/webserver31-tomcat8-openshift:$tag
docker pull registry.redhat.io/openshift3/jenkins-2-rhel7:$tag
docker pull registry.redhat.io/openshift3/jenkins-agent-maven-35-rhel7:$tag
docker pull registry.redhat.io/openshift3/jenkins-agent-nodejs-8-rhel7:$tag
docker pull registry.redhat.io/openshift3/jenkins-slave-base-rhel7:$tag
docker pull registry.redhat.io/openshift3/jenkins-slave-maven-rhel7:$tag
docker pull registry.redhat.io/openshift3/jenkins-slave-nodejs-rhel7:$tag
docker pull registry.redhat.io/rhscl/mongodb-32-rhel7:$tag
docker pull registry.redhat.io/rhscl/mysql-57-rhel7:$tag
docker pull registry.redhat.io/rhscl/perl-524-rhel7:$tag
docker pull registry.redhat.io/rhscl/php-56-rhel7:$tag
docker pull registry.redhat.io/rhscl/postgresql-95-rhel7:$tag
docker pull registry.redhat.io/rhscl/python-35-rhel7:$tag
docker pull registry.redhat.io/redhat-sso-7/sso70-openshift:$tag
docker pull registry.redhat.io/rhscl/ruby-24-rhel7:$tag
docker pull registry.redhat.io/redhat-openjdk-18/openjdk18-openshift:$tag
docker pull registry.redhat.io/redhat-sso-7/sso71-openshift:$tag
docker pull registry.redhat.io/rhscl/nodejs-6-rhel7:$tag
docker pull registry.redhat.io/rhscl/mariadb-101-rhel7:$tag
done
```

**image tag**

```
for i in `docker images | grep redhat | grep -v v3.11.219 | grep -v v3.11.439 | awk '{print $1":"$2}'` ; 
do 
	tra="`echo $i |sed 's/registry.redhat.io/bastion.ocp3d.test.fu.igotit.co.kr:5000/g'`" ; 
	echo docker tag $i $tra ; 
done
```

**image push**

```
for i in `docker images | grep bastion.ocp3.home.igotit.co.kr | awk '{print $1":"$2}'`
do
        echo docker push $i  checkpoint
        docker push $i
done
```



### yum repository 최신으로 갱신

**mirror repository**

```
for repo in rhel-7-server-rpms rhel-7-server-extras-rpms rhel-7-server-ansible-2.9-rpms rhel-7-server-ose-3.11-rpms
do
  reposync --gpgcheck -lm --repoid=${repo} --download_path=/var/ftp/pub/repos
  createrepo -v /var/ftp/pub/repos/${repo} -o /var/ftp/pub/repos/${repo}
done
```



### 운영중인 클러스터 백업 

**master node backup**

```
Master, infra, worker node 백업 
MYBACKUPDIR="~/OCP3.11.161/"

### master 
#### ocp file
if [ ! -d "${MYBACKUPDIR}" ] ; then mkdir ${MYBACKUPDIR} ; fi
if [ ! -d "${MYBACKUPDIR}/etc/sysconfig" ] ; then mkdir -p ${MYBACKUPDIR}/etc/sysconfig ; fi
cp -aR /etc/origin ${MYBACKUPDIR}/etc
cp -aR /etc/sysconfig/ ${MYBACKUPDIR}/etc/sysconfig/

#### other file
if [ ! -d "${MYBACKUPDIR}/etc/sysconfig" ] ; then mkdir -p ${MYBACKUPDIR}/etc/sysconfig ; fi
if [ ! -d "${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors" ] ; then mkdir -p ${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors ; fi
cp -aR /etc/sysconfig/{iptables,docker-*,flanneld}  ${MYBACKUPDIR}/etc/sysconfig/
cp -aR /etc/dnsmasq* /etc/cni ${MYBACKUPDIR}/etc/
cp -aR /etc/pki/ca-trust/source/anchors/* ${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors/

#### custom file
if [ ! -d "${MYBACKUPDIR}/etc/systemd" ] ; then mkdir -p ${MYBACKUPDIR}/etc/systemd ; fi
if [ ! -d "${MYBACKUPDIR}/etc/security/limits.d/" ] ; then mkdir -p ${MYBACKUPDIR}/etc/security/limits.d/ ; fi
if [ ! -d "${MYBACKUPDIR}/usr/lib/systemd/system/" ] ; then mkdir -p ${MYBACKUPDIR}/usr/lib/systemd/system/ ; fi
if [ ! -d "${MYBACKUPDIR}/etc/selinux/" ] ; then mkdir -p ${MYBACKUPDIR}/etc/selinux/ ; fi
cp -aR  /etc/sysctl.conf ${MYBACKUPDIR}/etc/sysctl.conf
cp -aR  /etc/systemd/journald.conf ${MYBACKUPDIR}/etc/systemd/journald.conf
cp -aR  /etc/sysconfig/docker ${MYBACKUPDIR}/etc/sysconfig/docker
## cp -aR  /etc/dnsmasq.d/origin.dns.conf ${MYBACKUPDIR}/etc/dnsmasq.d/origin.dns.conf
## cp -aR  /etc/dnsmasq.d/ignore.conf ${MYBACKUPDIR}/etc/dnsmasq.d/ignore.conf
cp -aR  /etc/security/limits.d/20-nproc.conf ${MYBACKUPDIR}/etc/security/limits.d/20-nproc.conf
cp -aR  /usr/lib/systemd/system/rpcbind.socket ${MYBACKUPDIR}/usr/lib/systemd/system/rpcbind.socke
cp -aR  /etc/selinux/config ${MYBACKUPDIR}/etc/selinux/config

### packagelist
rpm -qa | sort > ${MYBACKUPDIR}/packages.txt

### registry certificates
tar cvfpz etc_docker_cert.tar.gz /etc/docker/certs.d/ 
```

**node backup**

```
MYBACKUPDIR="~/OCP3.11.161/"
### node 
#### ocp file
if [ ! -d "${MYBACKUPDIR}" ] ; then mkdir ${MYBACKUPDIR} ; fi
if [ ! -d "${MYBACKUPDIR}/etc/sysconfig" ] ; then mkdir -p ${MYBACKUPDIR}/etc/sysconfig ; fi
cp -aR /etc/origin ${MYBACKUPDIR}/etc
cp -aR /etc/sysconfig/ ${MYBACKUPDIR}/etc/sysconfig/

#### other file
if [ ! -d "${MYBACKUPDIR}/etc/sysconfig" ] ; then mkdir -p ${MYBACKUPDIR}/etc/sysconfig ; fi
if [ ! -d "${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors" ] ; then mkdir -p ${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors ; fi
cp -aR /etc/sysconfig/{iptables,docker-*,flanneld}  ${MYBACKUPDIR}/etc/sysconfig/
cp -aR /etc/dnsmasq* /etc/cni ${MYBACKUPDIR}/etc/
cp -aR /etc/pki/ca-trust/source/anchors/* ${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors/

#### custom file
if [ ! -d "${MYBACKUPDIR}/etc/systemd" ] ; then mkdir -p ${MYBACKUPDIR}/etc/systemd ; fi
if [ ! -d "${MYBACKUPDIR}/etc/security/limits.d/" ] ; then mkdir -p ${MYBACKUPDIR}/etc/security/limits.d/ ; fi
if [ ! -d "${MYBACKUPDIR}/usr/lib/systemd/system/" ] ; then mkdir -p ${MYBACKUPDIR}/usr/lib/systemd/system/ ; fi
if [ ! -d "${MYBACKUPDIR}/etc/selinux/" ] ; then mkdir -p ${MYBACKUPDIR}/etc/selinux/ ; fi
cp -aR  /etc/sysctl.conf ${MYBACKUPDIR}/etc/sysctl.conf
cp -aR  /etc/systemd/journald.conf ${MYBACKUPDIR}/etc/systemd/journald.conf
cp -aR  /etc/sysconfig/docker ${MYBACKUPDIR}/etc/sysconfig/docker
## cp -aR  /etc/dnsmasq.d/origin.dns.conf ${MYBACKUPDIR}/etc/dnsmasq.d/origin.dns.conf
## cp -aR  /etc/dnsmasq.d/ignore.conf ${MYBACKUPDIR}/etc/dnsmasq.d/ignore.conf
cp -aR  /etc/security/limits.d/20-nproc.conf ${MYBACKUPDIR}/etc/security/limits.d/20-nproc.conf
cp -aR  /usr/lib/systemd/system/rpcbind.socket ${MYBACKUPDIR}/usr/lib/systemd/system/rpcbind.socke
cp -aR  /etc/selinux/config ${MYBACKUPDIR}/etc/selinux/config

### packagelist
rpm -qa | sort > ${MYBACKUPDIR}/packages.txt

### registry certificates
tar cvfpz etc_docker_cert.tar.gz /etc/docker/certs.d/ 
```

**etcd backup**

```
### etcd backup
# etcdctl3 --cert="/etc/etcd/peer.crt" \
          --key=/etc/etcd/peer.key \
          --cacert="/etc/etcd/ca.crt" \
          --endpoints=https://master-0.example.com:2379,https://master-1.example.com:2379,https://master-2.example.com:2379 \
          endpoint health

https://master0.ocp3d.test.fu.igotit.co.kr.com:2379 is healthy: successfully committed proposal: took = 5.011358ms
https://master1.ocp3d.test.fu.igotit.co.kr.com:2379 is healthy: successfully committed proposal: took = 1.305173ms
https://master2.ocp3d.test.fu.igotit.co.kr.com:2379 is healthy: successfully committed proposal: took = 1.388772ms

# etcdctl3 member list
2a371dd20f21ca8d, started, master0.ocp3d.test.fu.igotit.co.kr.com, https://192.168.6.231:2380, https://192.168.6.231:2379
40bef1f6c79b3163, started, master1.ocp3d.test.fu.igotit.co.kr.com, https://192.168.6.232:2380, https://192.168.6.232:2379
95dc17ffcce8ee29, started, master2.ocp3d.test.fu.igotit.co.kr.com, https://192.168.6.233:2380, https://192.168.6.233:2379


$ export ETCD_POD_MANIFEST="/etc/origin/node/pods/etcd.yaml"
$ export ETCD_EP=$(grep https ${ETCD_POD_MANIFEST} | cut -d '/' -f3)
$ oc login -u system:admin
$ export ETCD_POD=$(oc get pods -n kube-system | grep -o -m 1 '^master-etcd\S*')
$ oc project kube-system

$ oc exec ${ETCD_POD} -c etcd -- /bin/bash -c "ETCDCTL_API=3 etcdctl \
    --cert /etc/etcd/peer.crt \
    --key /etc/etcd/peer.key \
    --cacert /etc/etcd/ca.crt \
    --endpoints $ETCD_EP \
    snapshot save /var/lib/etcd/snapshot.db"

$ mkdir {MYBACKUPDIR}/etcd-backup/
$ tar cvfpz ~OCP3.11.161/etcd.backup.tar.gz ${MYBACKUPDIR}/etcd-backup/
```



## **업그레이드 작업 내역** 

### 버전 

**3.11.161 -> 3.11.219**

```
## master, metirc, cluster-monitoring, route, registry upgrade
ansible-playbook -i /root/hosts playbooks/byo/openshift-cluster/upgrades/v3_11/upgrade_control_plane.yml

## worker(computer) node upgrade
ansible-playbook -i /root/hosts playbooks/byo/openshift-cluster/upgrades/v3_11/upgrade_nodes.yml

## logging upgrade
### 실제 es를 사용하지 않는 것으로 알고 있으며 이러한 이유로 데이터 flush 등의 적업은 별도로 진행하지 않았음
ansible-playbook -i /root/hosts playbooks/openshift-logging/config.yml
oc scale --replicas=0 deploymentconfig.apps.openshift.io/logging-es-data-master-5emo0nl9
oc scale --replicas=0 deploymentconfig.apps.openshift.io/logging-kibana

```

