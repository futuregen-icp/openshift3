# 설치 시 pull Err 발생

Created: Apr 09, 2020 9:48 PM
Tags: deploy-registry

## 아래내용 추가하지 않을 경우 registry, router 부터 ErrImgPull 발생 
    openshift_additional_registry_credentials=[{'host':'registry.ocp3-11.fu.te:5000','user':'admin','password':'admin','test_login':'False'},{'host':'registry.ocp3-11.fu.te','user':'admin','password':'admin','test_login':'False'}]

registry  에 인증이 걸려 있는 경우 다음과 같이 credentials을 작성하여 준다