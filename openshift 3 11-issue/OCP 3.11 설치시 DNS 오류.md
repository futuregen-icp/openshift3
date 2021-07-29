##  OCP 3.11 설치시 DNS 오류 

```
DNS 조회 오류
https://access.redhat.com/solutions/4828571
https://stackoverflow.com/questions/57719177/openshift-3-11-install-fails-unable-to-update-cni-config-no-networks-found-in
echo "server=192.168.1.254" > /etc/dnsmasq.d/origin-upstream-dns.conf
echo "server=192.168.1.254" >> /etc/dnsmasq.conf
systemctl restart dnsmasq
```



```
Reading environment variables from /etc/sysconfig/atomic-openshift-node
User "sa" set.
Context "default/api-ocp3-home-igotit-co-kr:443/system:admin" modified.
which: no openshift-sdn in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin)
I0728 06:35:36.659705   11133 start_network.go:193] Reading node configuration from /etc/origin/node/node-config.yaml
I0728 06:35:36.661356   11133 start_network.go:200] Starting node networking master02.ocp3.home.igotit.co.kr (v3.11.219)
W0728 06:35:36.661501   11133 server.go:195] WARNING: all flags other than --config, --write-config-to, and --cleanup are deprecated. Please begin using a config file ASAP.
I0728 06:35:36.661557   11133 feature_gate.go:230] feature gates: &{map[]}
I0728 06:35:36.662521   11133 transport.go:160] Refreshing client certificate from store
I0728 06:35:36.662577   11133 certificate_store.go:131] Loading cert/key pair from "/etc/origin/node/certificates/kubelet-client-current.pem".
I0728 06:35:36.673383   11133 node.go:151] Initializing SDN node of type "redhat/openshift-ovs-subnet" with configured hostname "master02.ocp3.home.igotit.co.kr" (IP ""), iptables sync period "30s"
F0728 06:35:36.675123   11133 start_network.go:106] could not start DNS, unable to read config file: open /etc/origin/node/resolv.conf: no such file or directory

https://access.redhat.com/solutions/3165971
echo "nameserver 192.168.1.254" >  /etc/origin/node/resolv.conf
```

