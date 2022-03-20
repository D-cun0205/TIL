#### Vagrant

---

Vagrant 설치 후 vagrant up 명령어 입력 시 발생한 첫번째 에러

    There was an error while executing `VBoxManage`, a CLI used by Vagrant
    for controlling VirtualBox. The command and stderr is shown below.
    Command: ["startvm", "e7e060ee-6674-4813-8d6c-d94cc5d86688", "--type", "headless"]
    Stderr: VBoxManage: error: The virtual machine 'HashiCorp_default_1646694169356_29471'
    has terminated unexpectedly during startup with exit code 1 (0x1)
    VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component MachineWrap, interface IMachine

오래전에 설치한 virtualbox 를 그대로 사용하려고 하니 설정파일이 제거 되었는지  
정상적으로 실행이 되지 않아 오라클 에서 설치받아 uninstall, install 로 해결  
설치 후 vagrant up 명령어 입력 시 발생한 두번째 에러

    Vagrant was unable to mount VirtualBox shared folders. This is usually
    because the filesystem "vboxsf" is not available. This filesystem is
    made available via the VirtualBox Guest Additions and kernel module.
    Please verify that these guest additions are properly installed in the
    guest. This is not a bug in Vagrant and is usually caused by a faulty
    Vagrant box. For context, the command attempted was:

    mount -t vboxsf -o uid=1000,gid=1000,_netdev vagrant /vagrant
    The error output from the command was:
    mount: unknown filesystem type 'vboxsf'
    
물리 서버와 가상 서버의 폴더 마운트에 문제라고 하여 이를 해결하는 방법으로
vagrant-vbguest plugin 설치 이후 vagrant reload 하였는데 reload 에서
세번째 에러 발생

    The following SSH command responded with a non-zero exit status.
    Vagrant assumes that this means the command failed!
    umount: /mnt: not mounted

마운트 되지 않은 에러였는데 varant-vbguest plugin 설치 당시 버전이 0.30 이였으며
검색결과 버전호환 문제로 에러가 발생되는 사용자들이 많다는걸 확인
이를 해결하기 위한 방법으로 0.21 버전으로 uninstall/install 진행 후 생성된 에러들 해결

    1.vagrant plugin uninstall vagrant-vbguest
    2.vagrant plugin install-vbguest --plugin-version 0.21
    3.vagrant reload


#### 명령어
vagrant init : 프로비저닝을 위한 기초 파일 생성
vagrant up : Vagrantfile 을 읽어 들여 프로비저닝 진행
vagrant halt : 베이그런트에서 다루는 가상 머신 종류
vagrant destory : 베이그런트에서 관리하는 가상 머신 삭제
vagrant ssh : 베이그런트에서 관리하는 가상 머신에 ssh(Secure Shell) 접속
vagrant provision : 베이그런트에서 관리하는 가상 머신에 변경된 설정 적용

#### 옵션
-f : 가상 머신 강제 종료