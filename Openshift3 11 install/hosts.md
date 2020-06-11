# hosts

    ###########################################################################
    ### Global cluster
    ###########################################################################
    [OSEv3:vars]
    ##-------------------------------------------------------------------------
    ## Ansible
    ##-------------------------------------------------------------------------
    ansible_user=root
    ansible_become=yes
    debug_level=2
    
    ###########################################################################
    ### OpenShift Basic Vars
    ###########################################################################
    openshift_deployment_type=openshift-enterprise
    openshift_release=v3.11.188
    openshift_image_tag=v3.11.188
    openshift_pkg_version=-3.11.188
    
    # Configuring Certificate Validity
    openshift_hosted_registry_cert_expire_days=3650
    openshift_ca_cert_expire_days=3650
    openshift_node_cert_expire_days=3650
    openshift_master_cert_expire_days=3650
    etcd_ca_default_days=3650
    openshift_certificate_expiry_warning_days=2920
    openshift_certificate_expiry_fail_on_warn=2920
    
    # Configuring Cluster Pre-install Checks
    openshift_disable_check=memory_availability,disk_availability,docker_storage,docker_storage_driver,docker_image_availability,package_version,package_availability,package_update
    
    # Node Groups
    openshift_node_groups=[{'name': 'node-config-master', 'labels': ['node-role.kubernetes.io/master=true']}, {'name': 'node-config-infra', 'labels': ['node-role.kubernetes.io/infra=true']}, {'name': 'node-config-monitoring', 'labels': ['node-role.kubernetes.io/monitoring=true']}, {'name': 'node-config-compute', 'labels': ['node-role.kubernetes.io/compute=true'], 'edits': [{ 'key': 'kubeletArguments.pods-per-core','value': ['20']}]}]
    
    # Configure logrotate scripts
    logrotate_scripts=[{"name": "syslog", "path": "/var/log/cron\n/var/log/maillog\n/var/log/messages\n/var/log/secure\n/var/log/spooler\n", "options": ["daily", "rotate 7","size 500M", "compress", "sharedscripts", "missingok"], "scripts": {"postrotate": "/bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true"}}]
    
    ###########################################################################
    ### OpenShift Registries Locations
    ###########################################################################
    #oreg_url=fusione1r01gm.gmarket.nh:5000/openshift3/ose-${component}:${version}
    oreg_url=registry.ocp3-11.fu.te:5000/openshift3/ose-${component}:${version}
    osm_etcd_image=registry.ocp3-11.fu.te:5000/rhel7/etcd:3.2.26
    #oreg_auth_user=admin
    #oreg_auth_password=admin
    openshift_examples_modify_imagestreams=true
    openshift_docker_additional_registries=registry.ocp3-11.fu.te:5000
    openshift_docker_insecure_registries=registry.ocp3-11.fu.te:5000
    openshift_docker_options="--insecure-registry 172.40.0.0/16 -l warn --log-driver json-file --log-opt max-size=10M --log-opt max-file=3"
    openshift_additional_registry_credentials=[{'host':'registry.ocp3-11.fu.te:5000','user':'admin','password':'admin','test_login':'False'},{'host':'registry.ocp3-11.fu.te','user':'admin','password':'admin','test_login':'False'}]
    
    # Set this line to enable NFS
    openshift_enable_unsupported_configurations=true
    
    ###########################################################################
    ### OpenShift Master Vars
    ###########################################################################
    openshift_master_api_port=8443
    openshift_master_console_port=8443
    openshift_master_cluster_hostname=master01.ocp3-11.fu.te
    openshift_master_cluster_public_hostname=master01.ocp3-11.fu.te
    # openshift_master_named_certificates=[{"certfile": "/home/ebaycloud/named_certificates/STAR.ebaykorea.com/STAR_ebaykorea_com.crt", "keyfile": "/home/ebaycloud/named_certificates/STAR.ebaykorea.com/STAR_ebaykorea_com.key", "names": ["fusiona1.ebaykorea.com"], "cafile": "/home/ebaycloud/named_certificates/STAR.ebaykorea.com/STAR_ebaykorea_com.ca-bundle"}]
    openshift_master_default_subdomain=apps.ocp3-11.fu.te
    # openshift_master_overwrite_named_certificates=true
    openshift_master_cluster_method=native
    
    ###########################################################################
    ### OpenShift Network Vars
    ###########################################################################
    openshift_portal_net=172.40.0.0/16
    osm_cluster_network_cidr=10.140.0.0/14
    os_sdn_network_plugin_name='redhat/openshift-ovs-subnet'
    #os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
    
    ###########################################################################
    ### OpenShift Authentication Vars
    ###########################################################################
    # LDAP AND HTPASSWD Authentication (download ipa-ca.crt first)
    #openshift_master_identity_providers=[{'name': 'ldap', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider','attributes': {'id': ['dn'], 'email': ['mail'], 'name': ['cn'], 'preferredUsername': ['uid']}, 'bindDN': 'uid=admin,cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com', 'bindPassword': 'r3dh4t1!', 'ca': '/etc/origin/master/ipa-ca.crt','insecure': 'false', 'url': 'ldaps://ipa.shared.example.opentlc.com:636/cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com?uid?sub?(memberOf=cn=ocp-users,cn=groups,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com)'},{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
    
    # Just LDAP
    #openshift_master_identity_providers=[{'name': 'ldap_provider', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider','attributes': {'id': ['sAMAccountName'], 'email': ['mail'], 'name': ['name'], 'preferredUsername': ['sAMAccountName']}, 'bindDN': 'fusion@ebaykorea.corp', 'bindPassword': '', 'insecure': 'true', 'url': 'ldap://ebaykorea.corp/dc=ebaykorea,dc=corp?sAMAccountName?sub?(&(objectClass=user)(objectCategory=person))'}]
    
    # Just HTPASSWD
    openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
    
    # LDAP and HTPASSWD dependencies
    #Command => htpasswd -c /root/htpasswd.openshift ocpadmin
    #openshift_master_htpasswd_file=/home/ebaycloud/htpasswd.openshift
    #openshift_master_ldap_ca_file=/root/ipa-ca.crt
    
    ###########################################################################
    ### OpenShift Router and Registry Vars
    ###########################################################################
    # Router
    openshift_router_selector='node-role.kubernetes.io/infra=true'
    openshift_hosted_router_replicas=2
    # openshift_hosted_router_certificate={"certfile": "/home/ebaycloud/named_certificates/STAR.a1.ebaykorea.com/STAR_a1_ebaykorea_com.crt", "keyfile": "/home/ebaycloud/named_certificates/STAR.a1.ebaykorea.com/STAR_a1_ebaykorea_com.key", "cafile": "/home/ebaycloud/named_certificates/STAR.a1.ebaykorea.com/STAR_a1_ebaykorea_com.ca-bundle"}
    
    # Registry
    openshift_registry_selector='node-role.kubernetes.io/infra=true'
    openshift_hosted_registry_replicas=1
    openshift_hosted_registry_pullthrough=true
    openshift_hosted_registry_acceptschema2=true
    openshift_hosted_registry_enforcequota=true
    
    ###########################################################################
    ### OpenShift Service Catalog Vars
    ###########################################################################
    #openshift_enable_service_catalog=true
    #template_service_broker_install=true
    #openshift_template_service_broker_namespaces=['openshift']
    #ansible_service_broker_install=true
    #ansible_service_broker_local_registry_whitelist=['.*-apb$']
    
    openshift_enable_service_catalog=false
    template_service_broker_install=false
    ansible_service_broker_install=false
    
    #template_service_broker_remove=true
    #ansible_service_broker_install=true
    #openshift_service_catalog_remove=true
    
    
    #########################
    # Prometheus Metrics
    #########################
    openshift_cluster_monitoring_operator_install=true
    openshift_cluster_monitoring_operator_node_selector={"node-role.kubernetes.io/infra": "true"}
    
    openshift_cluster_monitoring_operator_prometheus_storage_enabled=true
    openshift_cluster_monitoring_operator_prometheus_storage_capacity=500Gi
    
    openshift_cluster_monitoring_operator_alertmanager_storage_enabled=true
    openshift_cluster_monitoring_operator_alertmanager_storage_capacity=20Gi
    
    
    ########################
    # Cluster Metrics
    ########################
    openshift_metrics_install_metrics=true
    openshift_metrics_cassandra_storage_type=pv
    openshift_metrics_cassandra_pvc_size=500Gi
    openshift_metrics_hawkular_nodeselector={"node-role.kubernetes.io/infra": "true"}
    openshift_metrics_cassandra_nodeselector={"node-role.kubernetes.io/infra": "true"}
    openshift_metrics_heapster_nodeselector={"node-role.kubernetes.io/infra": "true"}
    
    
    #######################
    # Cluster Logging
    ########################
    openshift_logging_install_logging=true
    openshift_logging_install_eventrouter=True
    openshift_logging_es_pvc_dynamic=false
    
    openshift_logging_image_prefix=registry.ocp3-11.fu.te:5000/openshift3
    openshift_logging_image_version=v3.11.188
    openshift_logging_curator_nodeselector={"node-role.kubernetes.io/infra":"true"}
    openshift_logging_kibana_nodeselector={"node-role.kubernetes.io/infra":"true"}
    openshift_logging_es_nodeselector={"node-role.kubernetes.io/infra":"true"}
    
    #openshift_logging_storage_kind=nfs
    #openshift_logging_storage_access_modes=['ReadWriteOnce']
    #openshift_logging_storage_host=fulab1-02-nfs.ocp3-11.fu.te
    #openshift_logging_storage_nfs_directory=/volumes
    #openshift_logging_storage_nfs_options='*(rw,root_squash)'
    #openshift_logging_storage_volume_name=logging
    #openshift_logging_storage_volume_size=10Gi
    #openshift_logging_storage_labels={'storage': 'logging'}
    # openshift_logging_es_pvc_storage_class_name=''
    openshift_logging_es_memory_limit=8Gi
    openshift_logging_es_cluster_size=1
    openshift_logging_curator_default_days=2
    openshift_logging_eventrouter_cpu_limit=200m
    
    openshift_logging_use_ops=true
    openshift_logging_kibana_hostname=kibana.apps.ocp3-11.fu.te
    openshift_logging_master_url=https://master01.ocp3-11.fu.te:8443
    openshift_logging_master_public_url=https://master01.ocp3-11.fu.te:8443
    openshift_logging_kibana_ops_hostname=kibana-ops.apps.ocp3-11.fu.te
    # openshift_logging_es_pvc_storage_class_name=''
    openshift_logging_es_pvc_size=20Gi
    openshift_logging_es_ops_pvc_size=20Gi
    openshift_logging_curator_cpu_request=500m
    openshift_logging_curator_ops_cpu_request=50m
    openshift_logging_kibana_cpu_request=50m
    openshift_logging_kibana_proxy_cpu_request=50m
    openshift_logging_kibana_ops_cpu_request=50m
    openshift_logging_kibana_ops_proxy_cpu_request=50m
    openshift_logging_fluentd_cpu_request=50m
    openshift_logging_es_cpu_request=500m
    openshift_logging_es_ops_cpu_request=250m
    openshift_logging_mux_cpu_request=50m
    openshift_logging_curator_memory_limit=256Mi
    openshift_logging_curator_ops_memory_limit=256Mi
    openshift_logging_kibana_memory_limit=450Mi
    openshift_logging_kibana_proxy_memory_limit=64Mi
    openshift_logging_kibana_ops_memory_limit=300Mi
    openshift_logging_kibana_ops_proxy_memory_limit=64Mi
    openshift_logging_fluentd_memory_limit=512Mi
    openshift_logging_mux_memory_limit=256Mi
    openshift_logging_curator_nodeselector={'node-role.kubernetes.io/infra':'true'}
    openshift_logging_curator_ops_nodeselector={'node-role.kubernetes.io/infra':'true'}
    openshift_logging_kibana_nodeselector={'node-role.kubernetes.io/infra':'true'}
    openshift_logging_kibana_ops_nodeselector={'node-role.kubernetes.io/infra':'true'}
    openshift_logging_es_nodeselector={'node-role.kubernetes.io/infra':'true'}
    openshift_logging_es_ops_nodeselector={'node-role.kubernetes.io/infra':'true'}
    openshift_logging_eventrouter_nodeselector={"node-role.kubernetes.io/infra": "true"}
    
    openshift_logging_es_cluster_size=1
    openshift_logging_es_ops_cluster_size=1
    openshift_logging_es_memory_limit=2G
    openshift_logging_es_ops_memory_limit=2G
    
    
    ###########################################################################
    ### OpenShift Hosts
    ###########################################################################
    [OSEv3:children]
    masters
    etcd
    nodes
    lb
    
    [masters]
    master01.ocp3-11.fu.te openshift_ip=192.168.40.151 openshift_public_ip=192.168.40.151 openshift_public_hostname=master01.ocp3-11.fu.te
    
    [etcd]
    master01.ocp3-11.fu.te openshift_ip=192.168.40.151 openshift_public_ip=192.168.40.151 openshift_public_hostname=master01.ocp3-11.fu.te
    
    [nodes]
    ## Master
    master01.ocp3-11.fu.te openshift_ip=192.168.40.151 openshift_public_ip=192.168.40.151 openshift_public_hostname=master01.ocp3-11.fu.te openshift_node_group_name='node-config-master'
    
    ### Infra
    infra01.ocp3-11.fu.te openshift_ip=192.168.40.181 openshift_public_ip=192.168.40.181  openshift_public_hostname=infra01.ocp3-11.fu.te openshift_node_group_name='node-config-infra'
    
    ### Node
    worker01.ocp3-11.fu.te openshift_ip=192.168.40.221 openshift_public_ip=192.168.40.221 openshift_public_hostname=worker01.ocp3-11.fu.te openshift_node_group_name='node-config-compute'
    worker02.ocp3-11.fu.te openshift_ip=192.168.40.222 openshift_public_ip=192.168.40.222 openshift_public_hostname=worker02.ocp3-11.fu.te openshift_node_group_name='node-config-compute'
    
    [lb]
    lb01.ocp3-11.fu.te
