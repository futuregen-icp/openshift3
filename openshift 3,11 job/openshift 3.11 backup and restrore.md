# openshift 3.11 backup and restrore

## openshift 3.11 backup



### etcd backup

**상태 확인**

```
etcdctl3 --cert="/etc/etcd/peer.crt" \
          --key=/etc/etcd/peer.key \
          --cacert="/etc/etcd/ca.crt" \
          --endpoints="https://master01.ocp3.home.igotit.co.kr:2379,https://master02.ocp3.home.igotit.co.kr:2379,https://master03.ocp3.home.igotit.co.kr:2379" \
          endpoint health


etcdctl3 member list
```



**백업**

```
mkdir /backup/etcd/

export ETCD_POD_MANIFEST="/etc/origin/node/pods/etcd.yaml"
export ETCD_EP=$(grep https ${ETCD_POD_MANIFEST} | cut -d '/' -f3)
oc login -u system:admin
export ETCD_POD=$(oc get pods -n kube-system | grep -o -m 1 '^master-etcd\S*')
oc project kube-system
oc exec ${ETCD_POD} -c etcd -- /bin/bash -c "ETCDCTL_API=3 etcdctl \
    --cert /etc/etcd/peer.crt \
    --key /etc/etcd/peer.key \
    --cacert /etc/etcd/ca.crt \
    --endpoints $ETCD_EP \
    snapshot save /var/lib/etcd/snapshot.db"

cp -aR /var/lib/etcd/snapshot.db /backup/etcd/
```



### master config  backup 

```
MYBACKUPDIR=/backup/$(hostname)/$(date +%Y%m%d)
sudo mkdir -p ${MYBACKUPDIR}/etc/sysconfig
sudo mkdir -p ${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors
sudo cp -aR /etc/sysconfig/{iptables,docker-*,flanneld} ${MYBACKUPDIR}/etc/sysconfig/
sudo cp -aR /etc/dnsmasq* /etc/cni ${MYBACKUPDIR}/etc/
sudo cp -aR /etc/pki/ca-trust/source/anchors/* ${MYBACKUPDIR}/etc/pki/ca-trust/source/anchors/
```



### node config  backup 

```
MYBACKUPDIR=/backup/$(hostname)/$(date +%Y%m%d)
sudo mkdir -p ${MYBACKUPDIR}
rpm -qa | sort | sudo tee $MYBACKUPDIR/packages.txt
```





## openshift 3.11 restore



### stopping  pod for etcd restored

**run on  all  master node **

```
mkdir -p /etc/origin/node/pods-stopped
mv /etc/origin/node/pods/* /etc/origin/node/pods-stopped

docker ps | grep master-etcd
```



###  instll the etcdctl

```
yum install etcd
systemctl mask etcd
```



### clear the /var/lib/etcd

```
mv /var/lib/etcd/member  /var/lib/etcd/etcd-backup-$(date +%d-%m-%y)
ls -l /var/lib/etcd
```



### Running restrore

```
source /etc/etcd/etcd.conf
export ETCDCTL_API=3
echo -e "$ETCD_INITIAL_CLUSTER \n$ETCD_INITIAL_CLUSTER_TOKEN"

ETCD_INITIAL_CLUSTER=master1.example.com=https://10.0.88.11:2380,master2.example.com=https://10.0.88.22:2380,master3.example.com=https://10.0.88.33:2380 
```

ETCD_INITIAL_CLUSTER은 백업의  etcd,conf에서 확인



**snapshit**

```
etcdctl snapshot restore /tmp/snapshot.db \
  --name $ETCD_NAME \
  --initial-cluster $ETCD_INITIAL_CLUSTER \
  --initial-cluster-token $ETCD_INITIAL_CLUSTER_TOKEN \
  --initial-advertise-peer-urls $ETCD_INITIAL_ADVERTISE_PEER_URLS \
  --data-dir /var/lib/etcd/restore
```



**hot copy backup**

```
etcdctl snapshot restore /tmp/db  \
  --name $ETCD_NAME \
  --data-dir /var/lib/etcd/restore \
  --initial-cluster $ETCD_INITIAL_CLUSTER \
  --initial-cluster-token $ETCD_INITIAL_CLUSTER_TOKEN \
  --initial-advertise-peer-urls $ETCD_INITIAL_ADVERTISE_PEER_URLS \
  --skip-hash-check=true 
```



**member restore**

```
mv /var/lib/etcd/restore/member /var/lib/etcd
rm -r /var/lib/etcd/restore
restorecon -Rv /var/lib/etcd
```



**restored etcd pod**

```
mv /etc/origin/node/pods-stopped/etcd.yaml /etc/origin/node/pods/
docker ps -a | grep etcd

<< docker start <container-id-of-POD> <container-id-of-normal-container> >>
```



**confirm health of etcd**

```
ETCD_ALL_ENDPOINTS=` etcdctl3 --write-out=fields   member list | awk '/ClientURL/{printf "%s%s",sep,$3; sep=","}'`
etcdctl3 --endpoints=$ETCD_ALL_ENDPOINTS  endpoint status  --write-out=table 
```



**restrore of pods**

```
mv /etc/origin/node/pods-stopped/* /etc/origin/node/pods/
```



**Check the health of the cluster**.

```
oc get nodes,pods -n kube-system
```

