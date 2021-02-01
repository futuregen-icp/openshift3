# GPU Node 추가 



### GPU driver 설치 

**기본 패키지 설치**

```
sudo yum install -y tar bzip2 make automake gcc gcc-c++ pciutils elfutilslibelf-devel libglvnd-devel iptables firewalld vim bind-utils  wget
```



**Driver 설치를 위한 kernel devel 및 header 설치 작업 **

```
distribution=rhel7
ARCH=$( /bin/arch )
yum install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```



**nvidia  driver 설치 **

```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/vulkan-filesystem-1.1.97.0-1.el7.noarch.rpm
rpm -Uvh vulkan-filesystem-1.1.97.0-1.el7.noarch.rpm
yum install -y nvidia-driver-latest-dkms
```



**nvidia  cuda driver 설치 **

```
yum install -y yum-utils
yum-config-manager --add-repo  
http://developer.download.nvidia.com/compute/cuda/repos/$distribution/${ARCH}/cuda-$distribution.repo

yum clean expire-cache
```

> cuda 설치는 선택사항이며 host에 설치도 가능하며 container에 설치도 가능하다



## container runtime 설치 

### 1. nvidia container toolkit 설치 

​     [설정 가이드](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

```
curl https://nvidia.github.io/nvidia-docker/centos7/nvidia-docker.repo > /etc/yum.repos.d/nvidia-docker.repo
yum -y install nvidia-container-toolkit
```



### 2. nvidia-container-runtime-hook

정리중

**oci-nvidia-hook 설정 **

```
혹시 

 cat << EOF >> /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json
{
  "hook": "/usr/bin/nvidia-container-runtime-hook",
  "arguments": ["prestart"],
  "annotations": ["sandbox"],
  "stage": [ "prestart" ]
}
EOF




```



**nvidia-container-runtime-hook** 설정 

```
cat << EOF  >> /usr/libexec/oci/hooks.d/oci-nvidia-hook/usr/bin/nvidia-container-runtime-hook $@
EOF
```



## 사전에 node scale out 

[node 추가 방법 ](https://github.com/futuregen-icp/openshift3/blob/master/openshift%203%2C11%20job/node-%EC%B6%94%EA%B0%80%20%EB%B0%A9%EB%B2%95(adding%20node).md)



**GPU node에서 SELINUX=permissive**

```
setenforce 0

[root@all ~]# cat /etc/sysconfig/selinux
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

```



## GPU사용을 위한 DaemonSet 구성

### project 생성 

```
oc new-project nvidia-device-plugin
```



### service account 및 SCC 생성

```
git clone https://github.com/redhat-performance/openshift-psap.git
cd openshift-psap/playbooks/roles/nvidia-device-plugin/files/

oc create serviceaccount nvidia-deviceplugin
oc create -f nvidia-device-plugin-scc.yaml

[root@all ~]# oc get scc | grep nvidia
nvidia-deviceplugin   true      [*]       RunAsAny    RunAsAny           RunAsAny    RunAsAny    10         false            [*]
```



### DaemonSet 구성 

```
git clone https://github.com/NVIDIA/k8s-device-plugin.git
cd k8s-device-plugin/
vi nvidia-device-plugin.yml
...
    spec:
      nodeSelector:
        region: gpu
      containers:
...

oc create -f nvidia-device-plugin.yml

oc get pod --all-namespaces
...
kube-system                         master-etcd-all.okd3.home.igotit.co.kr          1/1       Running   0          1d
kube-system                         nvidia-device-plugin-daemonset-qpp5w            1/1       Running   0          21h
openshift-ansible-service-broker    asb-1-8sbvs                                     1/1       Running   0          1d
...

[root@all ~]# oc get daemonset --all-namespaces
NAMESPACE                           NAME                             DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                         AGE
...
kube-system                         nvidia-device-plugin-daemonset   1         1         1         1            1           region=gpu                            21h
openshift-monitoring                node-exporter                    2         2         2         2            2           beta.kubernetes.io/os=linux           1d

```



## 확인

```
root@all ~]#  oc describe node gworker.okd3.home.igotit.co.kr
Name:               gworker.okd3.home.igotit.co.kr
Roles:              compute
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=gworker.okd3.home.igotit.co.kr
                    node-role.kubernetes.io/compute=true
                    openshift.com/gpu-accelerator=true
                    region=gpu
...
...
Allocatable:
 cpu:             2
 hugepages-1Gi:   0
 hugepages-2Mi:   0
 memory:          7842356Ki
 nvidia.com/gpu:  1
 pods:            250
...
...
  Namespace                        Name                                    CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                        ----                                    ------------  ----------  ---------------  -------------
  kube-system                      nvidia-device-plugin-daemonset-qpp5w    0 (0%)        0 (0%)      0 (0%)           0 (0%)
  openshift-monitoring             node-exporter-tx85x                     10m (0%)      20m (1%)    20Mi (0%)        40Mi (0%)
  openshift-node-problem-detector  node-problem-detector-b5lxd             0 (0%)        0 (0%)      0 (0%)           0 (0%)
  openshift-node                   sync-z5k6c                              0 (0%)        0 (0%)      0 (0%)           0 (0%)
  openshift-sdn                    ovs-twr2t                               100m (5%)     0 (0%)      300Mi (3%)       0 (0%)
  openshift-sdn                    sdn-7lm4v                               100m (5%)     0 (0%)      200Mi (2%)       0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource        Requests    Limits
  --------        --------    ------
  cpu             210m (10%)  20m (1%)
  memory          520Mi (6%)  40Mi (0%)
  nvidia.com/gpu  0           0
Events:           <none>

```

