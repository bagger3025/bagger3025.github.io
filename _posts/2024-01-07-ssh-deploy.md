---
title: ssh-deploy 사용할 때 생긴 문제점
tags:
  - CI/CD
  - ssh
  - 문제해결
post_time: 2024-01-07T14:00:13 +09:00
edit_time: 2024-01-07T14:37:14+09:00
---
# 사용한 툴
## ssh-deploy

ssh로 rsync를 이용해 코드를 올릴 수 있는 github action이다.

[github](https://github.com/easingthemes/ssh-deploy)에 나와 있는대로 몇 가지 설정을 넘기면 코드를 올릴 수 있다.

## authorized_keys

PEM 포맷으로 만든 public-private 키 쌍 중 public키를 서버에 등록해놓으면 private 키를 가진 사람은 비밀번호 없이 로그인할 수 있다.

그런 public 키를 `authorized_keys` 파일에 적어 놓으면 된다.

# 문제상황

## 1. remote host를 알 수 없다

[github](https://github.com/easingthemes/ssh-deploy)에 있는 예시만으로는 포트를 재지정하지 않는다. 저장소에 있는 파일 전체를 복사할 예정이고 예외처리할 폴더는 없는 점도 고려하여 다음과 같이 yml파일을 다시 작성하였다.

```yml
    - name: Deploy to Server
      uses: easingthemes/ssh-deploy@main
      with:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          ARGS: "-rlgoDzvc -i --delete"
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: ${{ secrets.REMOTE_TARGET }}
          REMOTE_PORT: ${{ secrets.REMOTE_PORT }}
```

## 2. Permission Denied

![](img/Pasted%20image%2020240107141446.png)

github action뿐 아니라 실제로 갖고 있는 private key로도 로그인시도 했을 때 비밀번호를 요구한다는 점에서 제대로 연동이 안 되었다는 것을 파악했다.

하지만 ssh를 key로 로그인하는 방법을 알려주는 모든 글에서는 이렇게 하면 된다고 써져 있었다.

## 3. 패스워드 요구

키로 로그인이 되어야 함에도 패스워드를 요구하는 것이 문제였다. 다음 두 글을 통해 알아낸 해결 방법은 다음과 같다.

- [글 1](https://serverfault.com/questions/380214/ssh-server-wont-recognize-authorized-keys)
	- `/etc/ssh/sshd_config`에 들어가서 설정을 바꾼다.
- [글 2](https://unix.stackexchange.com/questions/36540/why-am-i-still-getting-a-password-prompt-with-ssh-with-public-key-authentication)
	- 홈 디렉토리의 `.ssh`의 권한을 700, 그 폴더 안의 `authorized_keys`의 권한을 600으로 만든다.

이렇게 바꿨는데도 해결되지 않았다.

## 4. 디버그 모드

3번의 두 번째 게시글에서 debug모드로 sshd의 로그를 볼 수 있다는 이야기를 보았다. 게시글에서 설명하는 직접 실행하는 방법은 되지 않았다. 그래서 `sshd_config` 파일의 LogLevel을 Debug로 바꾸고 [이 글](https://serverfault.com/questions/130482/how-to-check-sshd-log)의 도움을 받아 로그를 확인했다.

결국 `cat /var/log/auth.log | grep sshd` 명령어를 사용해 홈 디렉토리의 경로가 문제였다는 것을 알게 되었고, 정상적으로 바꾸자 잘 되었다.

# 추가 문제사항

private key 뒤에 개행이 있어야 한다([링크](https://github.com/easingthemes/ssh-deploy/issues/175)). 글에서와 같이 libcrypto에 문제가 있고 Permission denied가 일어났을 때 해결 방법이다.

# 참고자료

괄호 안의 내용은 구글 검색문구이다.

- [ssh-deploy github](https://github.com/easingthemes/ssh-deploy)
- [Deployment Failed, Permission denied(publickey,password)](https://github.com/easingthemes/ssh-deploy/issues/175)
- (`linux authorized keys not working`): [SSH server won't recognize authorized_keys](https://serverfault.com/questions/380214/ssh-server-wont-recognize-authorized-keys)
- (`ssh prompt for password when using key`): [Why am I still getting a password prompt with ssh with public key authentication?](https://unix.stackexchange.com/questions/36540/why-am-i-still-getting-a-password-prompt-with-ssh-with-public-key-authentication)
- (`sshd log file path`): [How to check sshd log?](https://serverfault.com/questions/130482/how-to-check-sshd-log)
