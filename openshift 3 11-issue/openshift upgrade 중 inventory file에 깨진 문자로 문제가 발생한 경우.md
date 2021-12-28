# openshift upgrade 중 inventory file에 깨진 문자로 문제가 발생한 경우 



## 에러 사항 



```
TASK [Gathering Facts]
ok: [localhost]

TASK [set Facts]
ok: [localhost]

TASK [set Facts]
ok: [localhost]

TASK [fail]
skipping : [localhost]

PLAY  [Determine if service signer cert must be created]

TASK [Gathering Facts]
ok: [api01...]

TASK [Determine if service signer cert must be created]
ok: [api01...]

TASK [openshift_control_plane : verify API server]
FAILED - RETRYING: verify API server (120 retries left).
FAILED - RETRYING: verify API server (119 retries left).
FAILED - RETRYING: verify API server (118 retries left).
FAILED - RETRYING: verify API server (117 retries left).
FAILED - RETRYING: verify API server (116 retries left).
...
```



## 확인 사항 

```
 ansible-playbook -i /root/hosts  /usr/share/ansible/openshift-ansible/playbooks/byo/openshift_facts.yml -vvvv
```

디버깅 결과 확인 

```

                    "public_console_url": "https://worker01.ocp3.home.igotit.co.kr:8443/console",
                    "session_max_seconds": 3600,
                    "session_name": "ssn"
                },
                "node": {
                    "nodename": "worker01.ocp3.home.igotit.co.kr",
                    "sdn_mtu": "1450"
                }
            }
        },
        "changed": false,
        "failed": false
    }
}
ok: [worker02.ocp3.home.igotit.co.kr] => {
    "result": {
        "ansible_facts": {
            "openshift": {
                "common": {
                    "all_hostnames": [
                        "kubernetes.default",
                        "kubernetes.default.svc.cluster.local",
                        "kubernetes",
                        "openshift.default",
                        "openshift.default.svc",
                        "172.40.0.1",
                        "api.ocp3.home.igotit.co.kr",
                        "worker02.ocp3.home.igotit.co.kr",
                        "openshift.default.svc.cluster.local",
                        "192.168.1.128",
                        "kubernetes.default.svc",
                        "openshift"
                    ],
                    "config_base": "/etc/origin",
                    "dns_domain": "cluster.local",
                    "generate_no_proxy_hosts": true,
                    "hostname": "worker02.ocp3.home.igotit.co.kr",
                    "internal_hostnames": [
                        "kubernetes.default",
                        "kubernetes.default.svc.cluster.local",
                        "kubernetes",
                        "openshift.default",
                        "openshift.default.svc",
                        "172.40.0.1",
                        "worker02.ocp3.home.igotit.co.kr",
                        "openshift.default.svc.cluster.local",
                        "192.168.1.128",
                        "kubernetes.default.svc",
                        "openshift"
                    ],
                    "ip": "192.168.1.128",
                    "kube_svc_ip": "172.40.0.1",
                    "portal_net": "172.40.0.0/16",
                    "public_hostname": "worker02.ocp3.home.igotit.co.kr",
                    "public_ip": "192.168.1.128",
                    "raw_hostname": "worker02.ocp3.home.igotit.co.kr"
                },
                "current_config": {
                    "roles": [
                        "node",
                        "master"
                    ]
                },
                "master": {
                    "api_port": "8443",
                    "api_url": "https://api.ocp3.home.igotit.co.kr:8443",
                    "api_use_ssl": true,
                    "bind_addr": "0.0.0.0",
                    "cluster_hostname": "api.ocp3.home.igotit.co.kr",
                    "console_path": "/console",
                    "console_port": "8443",
                    "console_url": "https://api.ocp3.home.igotit.co.kr:8443/console",
                    "console_use_ssl": true,
                    "controllers_port": "8444",
                    "loopback_api_url": "https://worker02.ocp3.home.igotit.co.kr:8443",
                    "loopback_cluster_name": "worker02-ocp3-home-igotit-co-kr:8443",
                    "loopback_context_name": "default/worker02-ocp3-home-igotit-co-kr:8443/system:openshift-master",
                    "loopback_user": "system:openshift-master/worker02-ocp3-home-igotit-co-kr:8443",
                    "portal_net": "172.30.0.0/16",
                    "public_api_url": "https://worker02.ocp3.home.igotit.co.kr:8443",
                    "public_console_url": "https://worker02.ocp3.home.igotit.co.kr:8443/console",
                    "session_max_seconds": 3600,
                    "session_name": "ssn"
                },
                "node": {
                    "nodename": "worker02.ocp3.home.igotit.co.kr",
                    "sdn_mtu": "1450"
                }
            }
        },
        "changed": false,
        "failed": false
    }
}
META: ran handlers
META: ran handlers

PLAY RECAP **************************************************************************************************************************************************************************************************
infra01.ocp3.home.igotit.co.kr : ok=19   changed=0    unreachable=0    failed=0    skipped=23   rescued=0    ignored=0
infra02.ocp3.home.igotit.co.kr : ok=19   changed=0    unreachable=0    failed=0    skipped=23   rescued=0    ignored=0
localhost                  : ok=11   changed=0    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
master01.ocp3.home.igotit.co.kr : ok=48   changed=0    unreachable=0    failed=0    skipped=36   rescued=0    ignored=0
master02.ocp3.home.igotit.co.kr : ok=32   changed=0    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0
master03.ocp3.home.igotit.co.kr : ok=32   changed=0    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0
worker01.ocp3.home.igotit.co.kr : ok=19   changed=0    unreachable=0    failed=0    skipped=23   rescued=0    ignored=0
worker02.ocp3.home.igotit.co.kr : ok=19   changed=0    unreachable=0    failed=0    skipped=23   rescued=0    ignored=0


INSTALLER STATUS ********************************************************************************************************************************************************************************************
Initialization  : Complete (0:00:28)
[root@master01 init]#

```



### 해결 

```
 <master02.ocp3.home.igotit.co.kr> (0, '\r\n{"invocation": {"module_args": {"filter": "*", "gather_subset": ["all"], "fact_path": "/etc/ansible/facts.d", "gather_timeout": 10}}, "ansible_facts": {"ansible_fibre_channel_wwn": [], "module_setup": true, "ansible_distribution_version": "7.7", "ansible_distribution_file_variety": "RedHat", "ansible_env": {"LC_NUMERIC": "C", "LESSOPEN": "||/usr/bin/lesspipe.sh %s", "SSH_CLIENT": "192.168.1.121 49130 22", "SELINUX_USE_CURRENT_RANGE": "", "LOGNAME": "root", "USER": "root", "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin", "HOME": "/root", "LANG": "C", "TERM": "xterm", "SHELL": "/bin/bash", "SHLVL": "2", "XDG_RUNTIME_DIR": "/run/user/0", "SELINUX_ROLE_REQUESTED": "", "LC_ALL": "C", "XDG_SESSION_ID": "124", "_": "/usr/bin/python", "SSH_CONNECTION": "192.168.1.121 49130 192.168.1.122 22", "SSH_TTY": "/dev/pts/0", "SELINUX_LEVEL_REQUESTED": "", "PWD": "/root", "MAIL": "/var/mail/root", "LS_COLORS": "rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.:

 
 
 
 /etc/ansible/facts.d/openshift.fact 
 해당파일 삭제
```

