# catalog  image pull err

Created: Apr 09, 2020 10:37 PM

disconneted 설치시 발생하며 외부레지스트에 접근이 또는 이미지가 없어 imageStream  에서 에러 발생 . 이는 외부 레지스트리로 이미지 push 이후 아래와 같이 이미지 import 진행한다.

	for i in `oc describe is -n openshift  | grep registry | grep \! | awk '{print $7}' | sed -r "s/\"//g" | sed -r "s/registry.ocp3-11.fu.te:5000//g"` ; do
	        echo docker pull registry.redhat.io$i ;
	        echo docker tag registry.redhat.io$i registry.ocp3-11.fu.te:5000$i ;
	        echo docker push registry.ocp3-11.fu.te:5000$i ;
	        # echo docker push docker-registry-default.apps.ocp3-11.fu.te$i ;
	done


하나의 이미지 스트림 내에 여러개의 이미지를 요청함으로 아래와 같이 이미지를 임포트 한다.

    oc import-image ruby -n openshift --all=true --insecure=true --confirm=true ;

    for i in `oc get is -n openshift | awk '{print $1}'` ;do  
      oc import-image $i  -n openshift --all=true --insecure=true --confirm=true ; 
    done
