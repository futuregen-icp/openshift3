# route가 Error 발생

Created: Apr 09, 2020 9:48 PM
Tags: router error

intra node 수만큼  route를 복제한다.

하지만 infra note가 히나인 경우 2개 이상 pod를 생성하려고하면 Error 발생하며 route가 다운된다

바로 다운되는 것은 아니지만 어느순간 모든 라우트가 다운된다

infra node 수 만틈 복제 

    1. Delete the default router using the following command.
    
        # oc delete all -l router=router
    
    2. Create a new default router.
    
        # oc adm router --replicas=1 --service-account=router
        //원하는 node에 배포시 --selector='<labels>'
        # oc adm router --replicas=1 --service-account=router --selector='kubernetes.io/hostname=infra01.ocp3-11.fu.te'
        에러 발생시 아래와 같이 
        # oc scale dc/router --replicas=1
