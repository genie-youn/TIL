개인 프로젝트로 만든 애플리케이션을 구동중인 gcp vm이 disk full이 발생함.

`df -h` 명령어로 디스크 사용량을 확인해보니 /dev/sda2가 가득 차 있었다.

```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        486M     0  486M   0% /dev
tmpfs           494M     0  494M   0% /dev/shm
tmpfs           494M  6.7M  488M   2% /run
tmpfs           494M     0  494M   0% /sys/fs/cgroup
/dev/sda2        30G   30G     0  100% /
/dev/sda1       200M   12M  189M   6% /boot/efi
tmpfs            99M     0   99M   0% /run/user/1000
tmpfs            99M     0   99M   0% /run/user/0
```

`du -h /` 로 어느 디렉토리에서 이렇게 많이먹나 확인

범인은 `/var/log/google-cloud-ops-agent/subagents/logging-module.log` 였고 gcp vm 모니터링을 위한 로깅 에이전트의 로그가 혼자 30G를 차지하고 있었던것..

생각없이 그냥 날려버려서 무슨 내용을 그렇게 로깅했는지 찾지는 못함.

잠시 모니터링 해봤지만 로그가 더 증가하진 않았다.

또 disk가 full되면 무슨 일로 30G를 무지성 로깅했는지 확인하고 원인을 해결해야겠다.

해당 파일에 rotate 할 수 있는 옵션이 있나 찾아봤지만 찾을 수 없었음.

정 안되면 주기적으로 지워주는 shell script를 작성하든 해야 할듯.
