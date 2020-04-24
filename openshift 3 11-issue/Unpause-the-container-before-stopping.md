### 아래와 같은 에러 발생 
  master-restart api
  Error response from daemon: Cannot stop container dddc0260161d: Container dddc0260161d531b6aec29a5c29636794655568609aa576e3da088642530371c is paused. Unpause the container before stopping

  journalctl -xe
  level=error msg="Handler for POST /v1.26/containers/dddc0260161d

  docker stop 
  Error response from daemon: Cannot stop container dddc0260161d: Container dddc0260161d531b6aec29a5c29636794655568609aa576e3da088642530371c is paused. Unpause the container before stopping


### 해결 
  docker unpause dddc0260161d

