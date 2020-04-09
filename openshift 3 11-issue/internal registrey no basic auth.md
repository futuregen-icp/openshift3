# internal registrey no basic auth

Created: Apr 09, 2020 10:00 PM

oc create secret generic --from-file=.dockerconfigjson=/root/.docker/config.json --type=[kubernetes.io/dockerconfigjson](http://kubernetes.io/dockerconfigjson) pullsecret

    
    "**- name: pullsecret"을  namespaces : 사용할 project name, default, openshift 에 추가** 
    
    [root@master01 ~]# oc get sa default -o yaml
    apiVersion: v1
    imagePullSecrets:
    - name: default-dockercfg-8dpwq
    **- name: pullsecret**
    kind: ServiceAccount
    metadata:
      creationTimestamp: "2020-04-08T23:23:12Z"
      name: default
      namespace: myproject
      resourceVersion: "108015"
      selfLink: /api/v1/namespaces/myproject/serviceaccounts/default
      uid: ea7e3fee-79ef-11ea-a543-005056893630
    secrets:
    - name: default-token-9c2cj
    - name: default-dockercfg-8dpwq