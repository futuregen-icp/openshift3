### admin 패스워드를 잊어벼렸거나 admin아이디가 구성 되지 않았을 경우 

#### 아이디 패스워드 추가 
```
htpasswd -c /etc/origin/master/htpasswd admin
New password : 
Re password : 
```

#### 추가한 아이디에 cluster-admin role을 부여
```
oc adm policy add-cluster-role-to-user cluster-admin admin
```

```
master-restart api
master-restart controllers
```
