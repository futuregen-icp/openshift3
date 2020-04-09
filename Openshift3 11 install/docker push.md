# docker push

tagging

    <tag>=v3.11.188
    {major-tag}-v3.11
    $ docker tag registry.redhat.io/openshift3/ose-node:<tag> registry.ocp3-11.fu.te/openshift3/ose-node:<tag>
    $ docker tag registry.redhat.io/openshift3/ose-node:<tag> registry.ocp3-11.fu.te/openshift3/ose-node:{major-tag}
    
    $ docker tag registry.redhat.io/rhel7/etcd:3.2.26 registry.ocp3-11.fu.te/rhel7/etcd:3.2.26

push

    $ docker push registry.ocp3-11.fu.te/openshift3/ose-node:<tag>
    $ docker push registry.ocp3-11.fu.te/openshift3/ose-node:{major-tag}
    
    $ docker push registry.ocp3-11.fu.te/rhel7/etcd:3.2.26

작업 script

    tag1="v3.11.188"
    tag2="v3.11"
    
    for i in `docker images | grep redhat.io | awk '{print $1":"$2}'` ; do 
        name="`echo $i | cut -f: -d 1`" 
        tag="`echo $i | cut -f: -d 2`" 
        echo "docker tag $name:$tag " >> tag1.sh
        echo "docker tag $name:$tag " >> tag2.sh
        registry="`echo $name | sed -r \"s/redhat.io/ocp3-11.fu.te/g\"`"
        echo "docker push $registry:$tag1" >> push1.sh
    		echo "docker push $registry:$tag2" >> push2.sh 
    done
    
    nohup sh tag1.sh &
    nohup sh tag2.sh &
    nohup sh push1.sh &
    nohup sh push1.sh &