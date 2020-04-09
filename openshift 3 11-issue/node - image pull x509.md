# image pull x509 에러

Created: Apr 09, 2020 9:48 PM
Tags: docker pull, x509

인증서 복사 

    scp registry.ocp3-11.igotit.co.kr.key lb01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/       
    scp registry.ocp3-11.igotit.co.kr.key master01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/    
    scp registry.ocp3-11.igotit.co.kr.key infra01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/    
    scp registry.ocp3-11.igotit.co.kr.key worker01.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/    
    scp registry.ocp3-11.igotit.co.kr.key worker02.ocp3-11.fu.te:/etc/pki/ca-trust/source/anchors/
    
    trust root CA 등록 
    ansible OCP -a "update-ca-trust "
