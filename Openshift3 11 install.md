# Openshift3.11 install

## Subscription 등록(전체 서버. disconnect 환경인 경우 외부 연결되는 서버에만)

    subscription-manager register --username=locli5427 --password=Hellena1^^
    subscription-manager refresh
    subscription-manager list --available --matches '*OpenShift*'
    subscription-manager attach --pool=8a85f99c707807c801709f913ded7153
    subscription-manager repos     --enable="rhel-7-server-rpms"     \
                                   --enable="rhel-7-server-extras-rpms"     \
                                   --enable="rhel-7-server-ose-3.11-rpms"     \
                                   --enable="rhel-7-server-ansible-2.9-rpms"

## repo server 구성 (테스트시 bastion또는 gate서버에 설정)

### 필수 패키지 설치

    sudo yum -y install yum-utils createrepo docker git vsftpd

### repo 동기화 및 구성

    for repo in rhel-7-server-rpms rhel-7-server-extras-rpms rhel-7-server-ansible-2.9-rpms rhel-7-server-ose-3.11-rpms
    do
      reposync --gpgcheck -lm --repoid=${repo} --download_path=/var/ftp/pub/repos
      createrepo -v /var/ftp/pub/repos/${repo} -o /var/ftp/pub/repos${repo}
    done

### repo 설정

    cat /etc/yum.repos.d/ocp3-11.repo
    [rhel-7-server-rpms]
    name=rhel-7-server-rpms
    baseurl=ftp://10.0.0.120/pub/repos/rhel-7-server-rpms
    enabled=1
    gpgcheck=0
    [rhel-7-server-extras-rpms]
    name=rhel-7-server-extras-rpms
    baseurl=ftp://10.0.0.120/pub/repos/rhel-7-server-extras-rpms
    enabled=1
    gpgcheck=0
    [rhel-7-server-ansible-2.9-rpms]
    name=rhel-7-server-ansible-2.9-rpms
    baseurl=ftp://10.0.0.120/pub/repos/rhel-7-server-ansible-2.9-rpms
    enabled=1
    gpgcheck=0
    [rhel-7-server-ose-3.11-rpms]
    name=rhel-7-server-ose-3.11-rpms
    baseurl=ftp://10.0.0.120/pub/repos/rhel-7-server-ose-3.11-rpms
    enabled=1
    gpgcheck=0
    
    systemctl start vsftpd 
    systemctl enable vsftd 

## 방화벽 설정 (테스트시 bastion또는 gate서버에 설정)

### firewalld 설정  (1차 기본 설정)

    firewall-cmd --permanent --zone=external --add-service=ftp
    firewall-cmd --permanent --zone=external --add-service=dns
    firewall-cmd --permanent --zone=external --add-service=tftp
    firewall-cmd --permanent --zone=external --add-service=ssh
    firewall-cmd --permanent --zone=external --add-service=dhcp
    firewall-cmd --permanent --zone=external --add-service=proxy-dhcp
    firewall-cmd --permanent --zone=external --add-service=http
    firewall-cmd --permanent --zone=external --add-service=https
    firewall-cmd --permanent --zone=external --add-port=5000/tcp
    
    firewall-cmd --permanent --zone=internal --add-service=ftp
    firewall-cmd --permanent --zone=internal --add-service=dns
    firewall-cmd --permanent --zone=internal --add-service=tftp
    firewall-cmd --permanent --zone=internal --add-service=ssh
    firewall-cmd --permanent --zone=internal --add-service=dhcp
    firewall-cmd --permanent --zone=internal --add-service=proxy-dhcp
    firewall-cmd --permanent --zone=internal --add-service=http
    firewall-cmd --permanent --zone=internal --add-service=https
    firewall-cmd --permanent --zone=internal --add-port=5000/tcp
    
    firewall-cmd --reload
    

### IP Masquerade 설정(옵션)-(restricted natework 필수)

    ### ip forword를 위한 kernel 설정
    sysctl -w net.ipv4.ip_forward=1
    echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.d/ip_forward.conf
    
    ### 인터페이스 설정
    firewall-cmd --set-default-zone=external
    firewall-cmd --permanent --zone=internal --add-interface=ens224
    
    ### ip Masquerade 설정
    firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o ens192 -j MASQUERADE -s 192.168.40.0/24
    firewall-cmd --reload

## DNS 설정 (테스트시 bastion또는 gate서버에 설정)

### DNS 설치

    yum  install bind

### DNS 설정

    # vi /etc/named/named.conf
    options {
            listen-on port 53 { ANY; };
            listen-on-v6 port 53 { ANY; };
            directory       "/var/named";
            dump-file       "/var/named/data/cache_dump.db";
            statistics-file "/var/named/data/named_stats.txt";
            memstatistics-file "/var/named/data/named_mem_stats.txt";
            recursing-file  "/var/named/data/named.recursing";
            secroots-file   "/var/named/data/named.secroots";
            allow-query     { ANY; };
    
    # vi /etc/named.rfc1912.zones
    zone "ocp3-11.fu.te" IN {
            type master;
            file "ocp3-11.fu.te.zone";
            allow-update { none; };
    };
    
    zone "0.40.168.192.in-addr.arpa" IN {
            type master;
            file "ocp3-11.fu.te.rr";
            allow-update { none; };
    };
    
    # vi /var/named/ocp3-11.fu.te.zone
    $TTL 60
    @       IN SOA  dns.ocp3-11.fu.te. root.ocpi3-11.fu.te. (
                                            1       ; serial
                                            1D      ; refresh
                                            1H      ; retry
                                            1W      ; expire
                                            3H )    ; minimum
                    IN      NS      ns.ocp3-11.fu.te.;
                    IN      A       192.168.40.6;
    ns              IN      A       192.168.40.6;
    bastion         IN      A       192.168.40.6
    nfs             IN      A       192.168.40.26;
    registry        IN      A       192.168.40.6;
    ldap            IN      A       10.0.0.118 ;
    ;
    lb01            IN      A       192.168.40.7
    ;
    master01        IN      A       192.168.40.151;
    ;
    infra           IN      A       192.168.40.181;
    ;
    worker01        IN      A       192.168.40.221;
    worker02        IN      A       192.168.40.222;
    ;
    *.apps         IN      A       192.168.40.181;
    ;
    
    # vi /var/named/ocp3-11.fu.te.rr
    $TTL 20
    @       IN      SOA     ns.ocp3-11.fu.te.   root (
                            2019070700      ; serial
                            3H              ; refresh (3 hours)
                            30M             ; retry (30 minutes)
                            2W              ; expiry (2 weeks)
                            1W )            ; minimum (1 week)
            IN      NS      ns.ocp3-11.fu.te.
    ;
    ; syntax is "last octet" and the host must have fqdn with trailing dot
    151      IN      PTR     master01
    ;
    181     IN      PTR     infra01
    ;
    221      IN      PTR     worker01
    222      IN      PTR     worker02

## NFS 설정(별도의 nfs서버 구성)

    firewall-cmd --permanent --zone=public --add-service=nfs
        firewall-cmd --permanent --zone=public --add-service=portmapper
        firewall-cmd --permanent --zone=public --add-service=rpc-bind
        firewall-cmd --permanent --zone public --add-service mountd
        filewall-cmd --reload
    
    ### selinux 설정
    
        setsebool -P virt_use_nfs 1
        setsebool -P virt_sandbox_use_nfs 1
    
    ### disk device mount
    
        parted /dev/sdb
        	(parted) mklabel
        	New disk label type? gpt
        	(parted) mkpart
        	Partition name?  []? /data
        	File system type?  [ext2]? xfs
        	Start? 0
        	End? -0
        
        mkfs.xfs /dev/sdb1
        mkdir /ocp
        echo "UUID=b688463c-cbaa-4676-aedc-054c461ce6b6 /ocp xfs    defaults        0 0" >> /etc/fstab
        mount -a
        mkdir /ocp/Logging 
        mkdir /ocp/Metrics
        mkdir /ocp/volumes
        chown nfsnobody.nfsnobody /ocp-R
        chmod 777 /registry -R
    
    ### running
    
        systemctl start nfs rpcbind
        systemctl enable nfs rpcbind
    
    ### NFS export
    
        exportfs -av 
        /ocp/Logging *
        /ocp/Metrics *
        /ocp/volumes *

## Docker registry 구성

[클릭](https://github.com/futuregen-icp/openshift4/blob/master/INFRA/docker-registry/nginx%2Bdocker-registry%2Bdocker-registry-web/docker%20registry.md) 

registry.ocp3-11.ocp3-11.fu.te.crt 전서버의  /etc/pki/ca-trust/source/anchors/ 위치에 복사

### 설치 이미지 복제

pullsecret 

[링크 내용 참조](https://github.com/futuregen-icp/openshift4/blob/master/openshift4.3-online-insatll/Pullsecerts.md)

**pulling** 

[docker pull](Openshift3%2011%20install/docker%20pull.md)

**docker save  and load**

[docker save and load](Openshift3%2011%20install/docker%20save%20and%20load.md)

docker push

[docker push](Openshift3%2011%20install/docker%20push.md)

## bastion 설정

    yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
    yum -y update
    yum -y install openshift-ansible
    yum -y install docker
    
    # cat /etc/ans
    [master]
    192.168.40.151
    [infra]
    192.168.40.181
    [lb]
    192.168.40.7
    [node]
    192.168.40.221
    192.168.40.222
    [nfs]
    192.168.40.26
    [bastion]
    192.168.40.6
    [OCP]
    192.168.40.7
    192.168.40.26
    192.168.40.151
    192.168.40.181
    192.168.40.221
    192.168.40.222
    
    #(root) ssh-keygen
    
    #(root) ssh-copy-id -i '/root/.ssh/id_rsa.pub' bastion.ocp3-11.fu.te 
    #(root) ssh-copy-id -i '/root/.ssh/id_rsa.pub' nfs.ocp3-11.fu.te
    #(root) ssh-copy-id -i '/root/.ssh/id_rsa.pub' master01.ocp3-11.fu.te
    #(root) ssh-copy-id -i '/root/.ssh/id_rsa.pub' infra01.ocp3-11.fu.te
    #(root) ssh-copy-id -i '/root/.ssh/id_rsa.pub' worker01.ocp3-11.fu.te
    #(root) ssh-copy-id -i '/root/.ssh/id_rsa.pub' worker02.ocp3-11.fu.te
    
    $(core) ansible OCP -m ping
    192.168.40.6 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }
    192.168.40.26 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }
    192.168.40.221 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }
    192.168.40.181 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }
    192.168.40.151 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }
    192.168.40.222 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }

## 전체 서버 설정 동일

    필수 패키지 설치 
    yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
    yum -y update
    yum -y install openshift-ansible
    yum -y install docker
    
    인증서 복사 
    scp registry.ocp3-11.igotit.co.kr.key lb01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/
    scp registry.ocp3-11.igotit.co.kr.key master01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/
    scp registry.ocp3-11.igotit.co.kr.key infra01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/
    scp registry.ocp3-11.igotit.co.kr.key worker01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/
    scp registry.ocp3-11.igotit.co.kr.key worker02.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/
    
    trust root CA 등록 
    ansible OCP -a "update-ca-trust "
    
    selinux on 
    /etc/sysconfig/selinux
      ...
      SELINUX=enforcing
      ...
    

## ansable-playbook

### invetory file

hosts

[hosts](Openshift3%2011%20install/hosts.md)

    #oreg_auth_user=admin
    #oreg_auth_password=admin
    
    ## disconnect 환경에서 아래 내용 추가 ## 
    openshift_examples_modify_imagestreams=true
    openshift_docker_additional_registries=registry.ocp3-11.fu.te:5000
    openshift_docker_insecure_registries=registry.ocp3-11.fu.te:5000
    openshift_docker_options="--insecure-registry 172.40.0.0/16 -l warn --log-driver json-file --log-opt max-size=10M --log-opt max-file=3"
    
    ## 아래내용 추가하지 않을 경우 registry, router 부터 ErrImgPull 발생 
    openshift_additional_registry_credentials=[{'host':'registry.ocp3-11.fu.te:5000','user':'admin','password':'admin','test_login':'False'},{'host':'registry.ocp3-11.fu.te','user':'admin','password':'admin','test_login':'False'}]
    

### playboock 구동

사전점검

    # cd /usr/share/ansible/openshift-ansible
    # ansible-playbook -i /opt/ocp3-11/hsots playbooks/prerequisites.yml

배포

    # cd /usr/share/ansible/openshift-ansible
    # ansible-playbook -i /opt/ocp3-11/hsots playbooks/deploy_cluster.yml

제거 

    # cd /usr/share/ansible/openshift-ansible
    # ansible-playbook -i /opt/ocp3-11/hsots playbooks/uninstall.yml

153 oc get pod --all-namespaces
154 oc logs router-2-pp5n8
155 oc describe pod router-2-pp5n8
156 oc get pod
157 oc get pod -a
158 oc get all
159 oc delete router-1-deploy
160 oc delete router-1-deploy --all
161 oc delete replicationcontroller/router-1
162 oc delete pod/router-1-deploy
163 oc get all
164 oc get pod -o wide
165 oc delete pod router-2-pp5n8
166 oc get pod -o wide
167 oc edit pod router-2-deploy
168 oc adm router --replicas=
169 oc adm router --replicas=0
170 oc get pod -o wide
171 oc scale dc/router --replicas=1
172 oc get pod -o wide
173 oc scale dc/router --replicas=0
174 oc get pod -o wide
175 oc scale dc/router --replicas=1
176 oc get pod -o wide
177 oc scale dc/router --replicas=2
178 oc get pod -o wide
179 oc scale dc/router --replicas=3
180 oc get pod -o wide
181 oc scale dc/router --replicas=0
182 history
183 oc delete replicationcontroller/router-2
184 oc rollout latest dc/router
185 oc get pod -o wide
186 oc adm router
187 oc rollout latest dc/router
188 oc get pod -o wide
189 oc scale dc/router --replicas=1
190 oc get pod -o wide
191 oc get pod -o wide --all-namespaces
192 history

1. Delete the default router using the following command.

    $ oc delete all -l router=router

2. Create a new default router.

    $ oc adm router --replicas=1 --service-account=router