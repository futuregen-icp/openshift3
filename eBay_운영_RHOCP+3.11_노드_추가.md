## eBay 운영 RHOCP 3.11 노드 추가 (a1 Cluster)

### 1. 사전 환경 확인
##### 1.1 OS 종류 및 버전 확인
```
## 증설 대상 서버의 OS 종류 및 버전을 확인합니다. 
cat /etc/redhat-release
```
##### 1.2 OS 계정 확인
```
## 증설 대상 서버의 OS 계정을 확인합니다. 
id 
## 노드 증설시 사용되는 Ansible 수행 대상 계정 및 해당 계정의 sudo(nopasswd) 권한을 확인합니다. 
cat /etc/sudoers 
```
##### 1.3 Hostname 확인 
```
## 해당 서버의 hostname을 확인합니다. 
** 각 서버의 hostname은 FQDN으로 설정되어야 함 
hostnamectl 
hostname -f
```
##### 1.4 DNS 설정 확인 
```
## 해당 서버에 DNS 서버가 설정되어 있는지 확인합니다. 
cat /etc/resolv.conf 
## 해당 서버가 DNS 서버에 등록되어 있는지 확인합니다. 
dig <hostname> +short
```
##### 1.5 NetworkManager 활성화 확인 
```
## NetworkManager가 활성화 되어 있는지 확인합니다. 
systemctl status NetworkManager
```
##### 1.6 SELinux 설정 확인(enforcing)
```
## SELinux 설정이 enforcing으로 되어 있는지 확인합니다. 
getenforce 
cat /etc/selinux/config 
** 해당 클러스터의 경우 담당자의 요청에 의해 노드 증설 완료 후 SELinux를 Permissive로 설정
```
##### 1.7 YUM 저장소 등록 확인
```
## 기존 노드에 있는 repo파일 동일 구성
cat /etc/yum.repos.d/ocp.repo 
yum clean all; yum repolist 
```
##### 1.8 GPU 드라이버 설치 확인 
```
yum list installed | egrep “nvidia-driver|nvidia-driver-cuda|nvidia-modprobe” 
```
### 2. 사전 작업
##### 2.1 SSH-KEY 생성 및 복사
```
## 노드 증설 ansible 수행서버(Master01)에서 각 노드로 ssh public key를 복사합니다. 
$ ssh-key-copy -I ~/.ssh/id_rsa.pub <node hoetname> 
```
##### 2.2 Network 설정
```
## ipv4 addr, gateway, dns, search domain 등 Network 설정을 확인합니다. 
ip -4 a 
cat /etc/sysconfig/network-script/ifcfg-<interface name> 
** 생성된 OS에는 기본 network 설정은 되어 있었고 network script에서 DNS와 Search Domain이 주석으로 되어 있어 해당 부분만 해제
```
##### 2.3 최신 패키지로 업데이트 진행 (Playbook 실행)
```
## 최신으로 패키지를 업데이트 한 후 서버를 재기동 합니다. 
yum update 
```
##### 2.4 기본 패키지 설치 (Playbook 실행)
```
## OpenShift 노드 추가를 위한 기본 패키지를 설치합니다. 
yum install wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos 
psacct
```
##### 2.5 Docker 설치 (Playbook 실행)
```
## docker를 설치합니다. 
yum install docker 
```
##### 2.6 installed nvidia-container-toolkit (Playbook 실행)
```
## nvidi container toolkit 설치
sudo yum install nvidia-container-toolkit -y
```
##### 2.7 OS 환경설정 동일화 작업 (Playbook 실행)
```
## 기존 클러스터의 노드들에 설정된 동기화가 필요한 OS 설정을 적용합니다. 
** 고객의 요청으로 진행하였고 기존 설정 파일 백업 진행함(해당 클러스터에 의존적인 내용) 
 (위치 : /home/ebaycloud/.futuregen/os_conf_bak_200213) 
## 설정파일 항목은 다음과 같습니다. 
- /etc/sysctl.conf 
- /etc/systemd/journald.conf 
- /etc/sysconfig/docker 
- /etc/dnsmagq.d/origin.dns.conf 
- /etc/dnsmasq.d/ignore.conf 
- /etc/security/limits.d/20-nproc.conf 
- /usr/lib/systemd/system/rpcbind.socket 
## 설정 대상 노드 리부팅
```
##### 2.9 OS 재기동 
```
reboot
```
##### 2.10 pre settings Playbook 
```
- hosts: allserver
  name: Fusion node pre settings
  become: yes
  remote_user: ebaycloud
  tasks:        
    - name: report 4.3 Put SELinux in permissive mode
      selinux:
        policy: targeted
        state: enforcing

    - name: report 2.5 Install Docker
      yum:
        name: "{{ item }}"
        state: present
      with_items:
        - docker-1.13.1-109.gitcccb291.el7_7

    - name: report 2.6 Install nvidia-container-toolkit
      yum:
        name: "{{ item }}"
        state: present
      with_items:
        - nvidia-container-toolkit-1.0.5-2
      when: "inventory_hostname in groups.gpunodes"

    - name: report 2.7 OS Parameter
      copy: src={{ item.src }} dest={{ item.dest }}
      with_items:
        - { src: '/home/ebaycloud/.futuregen/os_conf_bak_200213/sysctl.conf', dest: '/etc/sysctl.conf' }
        - { src: '/home/ebaycloud/.futuregen/os_conf_bak_200213/journald.conf', dest: '/etc/systemd/journald.conf' }
        - { src: '/home/ebaycloud/.futuregen/os_conf_bak_200213/docker', dest: '/etc/sysconfig/docker' }
        - { src: '/home/ebaycloud/.futuregen/os_conf_bak_200213/origin.dns.conf', dest: '/etc/dnsmagq.d/origin.dns.conf' }
        - { src: '/home/ebaycloud/.futuregen/os_conf_bak_200213/ignore.conf', dest: '/etc/dnsmasq.d/ignore.conf' }
        - { src: '/home/ebaycloud/.futuregen/os_conf_bak_200213/20-nproc.conf', dest: '/etc/security/limits.d/20-nproc.conf' }
        - { src: '/home/ebaycloud/.futuregen/os_conf_bak_200213/rpcbind.socket', dest: '/usr/lib/systemd/system/rpcbind.socket' }
```
### 3. 노드 추가 작업
##### 3.1 Ansible Playbook Inventory 파일 편집
```
vi hosts
###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
etcd
nodes
new_nodes # <- add
...
...
[masters]
ip-10-0-0-130.ap-northeast-2.compute.internal
[etcd]
ip-10-0-0-130.ap-northeast-2.compute.internal
[nodes]
ip-10-0-0-130.ap-northeast-2.compute.internal openshift_node_group_name='node-config-master'
ip-10-0-0-72.ap-northeast-2.compute.internal openshift_node_group_name='node-config-compute'

[new_nodes] # <- add
ip-10-0-0-183.ap-northeast-2.compute.internal openshift_node_group_name='node-config-compute'
```
##### 3.2 Openshift 노드 추가
```
ansible-playbook -i hosts playbooks/openshift-node/scaleup.yml
```
##### 3.3 Ansible Playbook Inventory 파일 편집
```
## 추가된 노드을 항목을 [nodes]에 기입 및 [new_nodes] 항목 삭제
```
### 4. 후 작업
##### 4.1 노드 분리를 위한 노드 Label 설정
```
## CPU 사용 노드 Label 설정 
oc label node/<node name> region=cpu 
## GPU 사용 노드 Label 설정 
oc label node/<node name> region=gpu 
oc label node/<node name> openshift.com/gpu-accelerator=true
```
##### 4.2 방화벽 허용 포트 설정 (Playbook 실행)
```
#######기존 노드 /etc/sysconfig/iptables 파일 copy
## SMB/CIFS 및 SNMP에 대한 방화벽 설정을 추가합니다. 
** 고객의 요청으로 해당 포트들에 대한 방화벽 설정을 진행(해당 클러스터에 의존적인 내용) 
## 방화벽 설정은 /etc/sysconfig/iptables 파일을 직접 수정하였고 하기 내용을 추가한 후 서버를 재기동하여 적
용 내용을 확인 하였습니다. 
-A INPUT -p udp -m udp --dport 137 -j ACCEPT (SMB, CIFS) 
-A OUPUT -p udp -m udp --sport 137 -j ACCEPT (SMB, CIFS) 
-A OS_FIREWALL_ALLOW -p udp -m udp --dport 161 -j ACCEPT (SNMP)
```
##### 4.3 SELinux 설정  (Playbook 실행)
```
## /etc/selinux/config에서 SELINUX를 Permissive로 설정 
## setenforce를 통해 Permissive로 설정 
sudo setenforce 0
```
##### 4.4 crontab 동일 세팅 (Playbook 실행)
```
## 기존 서버의 Crontab 동기화
```
##### 4.5 기타 서비스 active 상태 확인 
```
## rpcbind 및 snmpd에 대한 서비스 상태를 확인합니다. 
** 고객의 요청으로 해당 서비스에 대한 enable 및 active 상태 확인(해당 클러스터에 의존적인 내용) 
systemctl status rpcbind 
systemctl status snmpd
```
##### 4.6 GPU node 설정 확인
```
## DaemonSet POD 확인
oc get pods -n kube-system 
## GPU 할당 확인
oc describe node <gpu-node> | egrep ' Capacity|Allocatable|gpu' 
```

##### 4.7 follow up  Playbook 
```
- hosts: allserver
  name: Fusion node follow-up task
  become: yes
  remote_user: ebaycloud
  tasks:
    - name: report 4.2 modefy iptables
      copy: src=/home/ebaycloud/.futuregen/os_conf_bak_200213/iptables dest=/etc/sysconfig/iptables
      
    - name: report 4.2 restart iptables
      service: name=iptables state=restarted

    - name: report 4.3 Put SELinux in permissive mode
      selinux:
        policy: targeted
        state: permissive

```
- inventory file
```
[allserver:children]
gpunodes
cpunodes

[gpunodes]

[cpunodes]


```
