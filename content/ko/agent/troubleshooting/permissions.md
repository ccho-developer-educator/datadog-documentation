---
aliases:
- /ko/agent/faq/how-to-solve-permission-denied-errors
- /ko/agent/faq/why-don-t-i-see-the-system-processes-open-file-descriptors-metric
- /ko/agent/faq/cannot-open-an-http-server-socket-error-reported-errno-eacces-13
further_reading:
- link: /agent/troubleshooting/debug_mode/
  tag: 설명서
  text: Agent 디버그 모드
- link: /agent/troubleshooting/send_a_flare/
  tag: 설명서
  text: Agent Flare 보내기
title: 권한 허용 문제
---

에이전트가 호스트에서 데이터를 수집하려면 특정 권한 세트가 필요합니다. 일반적으로 발생하는 권한 허용 문제와 해결법을 아래로 알려드리겠습니다.

## Agent 로깅 권한 문제

Datadog Agent를 호스트에서 실행할 때 접근 권한과 관련하여 아래와 같은 문제가 발생할 수 있습니다. 이러한 문제는 Agent의 적절한 로깅 기능을 저해할 수 있습니다.

```text
IOError: [Errno 13] Permission denied: '/var/log/datadog/supervisord.log'
```

Agent 로그 파일과 로그 파일이 포함된 디렉터리의 소유자가 Datadog Agent 사용자(`dd-agent`)인지 확인하세요. 소유자와 Agent 사용자가 일치하지 않는 경우 Agent는 로그 엔트리를 파일에 작성할 수 없습니다. 파일의 소유 정보를 표시하기 위해 Unix 시스템에서 이용할 수 있는 명령어는 다음과 같습니다.

```text
ls -l /var/log/datadog/

total 52300
-rw-r--r-- 1 dd-agent dd-agent 5742334 Jul 31 11:49 collector.log
-rw-r--r-- 1 dd-agent dd-agent 10485467 Jul 28 02:45 collector.log.1
-rw-r--r-- 1 dd-agent dd-agent 1202067 Jul 31 11:48 dogstatsd.log
-rw-r--r-- 1 dd-agent dd-agent 10485678 Jul 28 07:04 dogstatsd.log.1
-rw-r--r-- 1 dd-agent dd-agent 4680625 Jul 31 11:48 forwarder.log
-rw-r--r-- 1 dd-agent dd-agent 10485638 Jul 28 07:09 forwarder.log.1
-rw-r--r-- 1 dd-agent dd-agent 1476 Jul 31 11:37 jmxfetch.log
-rw-r--r-- 1 dd-agent dd-agent 31916 Jul 31 11:37 supervisord.log
-rw-r--r-- 1 dd-agent dd-agent 110424 Jul 31 11:48 trace-agent.log
-rw-r--r-- 1 dd-agent dd-agent 10000072 Jul 28 08:29 trace-agent.log.1
```

`dd-agent` 사용자가 이 파일을 소유하지 **않은** 경우, 아래 명령어로 소유 주체를 변경한 후 [Agent를 재시작하세요][1].

```text
sudo chown -R dd-agent:dd-agent /var/log/datadog/
```

[Agent 로그 위치를 자세히 알아보려면 여기를 참조하세요][2].

## Agent 소켓 권한 문제

Agent를 시작할 때 다음과 같은 소켓 권한 허용 문제가 발생할 수 있습니다.

```text
Starting Datadog Agent (using supervisord):Error: Cannot open an HTTP server: socket.error reported errno.EACCES (13)
```

언뜻 보기에 적절한 소켓이 이미 사용 중이어서 Agent에서 연결하지 못하는 것처럼 보일 수도 있습니다. 그러나 [남은 Agent 프로세스 중 지연된 것이 없다][3]는 점을 확인했고, Agent용으로 [적절한 포트][4]가 사용 가능하더라도 위와 같은 오류가 지속될 수 있습니다.

Linux 호스트의 경우 정상적으로 부팅하려면 `/opt/datadog-agent/run` 디렉터리의 주인이 `dd-agent`여야 합니다. 드문 경우지만 디렉터리 소유자가 `dd-agent` 이외로 변경되는 사례가 존재합니다. Agent 시작 시 위와 같은 오류가 발생하는 이유도 그 때문입니다. 다음 명령어를 실행하여 디렉터리 소유권을 다시 확인하세요.

```text
ls -al /opt/datadog-agent/run
```

파일 소유자가 `dd-agent`가 **아닌** 경우, 다음 명령어를 실행해 수정하세요

```text
sudo chown -R dd-agent:dd-agent /opt/datadog-agent/run
```

이렇게 변경한 후에는 [Agent 부팅 명령어][5]로 Agent를 시작할 수 있습니다. 위의 해결책을 시도했는데도 계속해서 문제가 발생하는 경우 [Datadog 지원팀][6]에 추가로 도움을 받으시기 바랍니다.

## 프로세스 메트릭 권한 문제

리눅스 OS에서 실행 중인 Agent에서 [프로세스 점검][7]을 활성화한 경우 기본적으로 `system.processes.open_file_descriptors` 메트릭이 수집되거나 보고되지 않습니다.
이 문제는 Agent 사용자 `dd-agent`가 아닌 다른 사용자가 실행한 프로세스 점검으로 프로세스를 모니터링할 때 발생합니다. 실제로 `dd-agent` 사용자는 Agent가 메트릭 데이터를 수집하기 위해 참조하는 `/proc` 전체 파일에 대한 접근 권한이 없습니다.

프로세스 점검 설정에서 `try_sudo` 옵션(에이전트 6.3 이상부터 사용 가능)을 활성화하고 적절한 `sudoers` 규칙을 추가하세요.

```text
dd-agent ALL=NOPASSWD: /bin/ls /proc/*/fd/
```

이렇게 하면 프로세스 점검에서 `sudo`를 활용해 `ls` 명령어를 실행하되, `/proc/*/fd/` 경로의 콘텐츠 목록에서만 실행합니다.

Datadog `error.log` 파일에서 `sudo: sorry, you must have a tty to run sudo`이라는 라인을 발견하면 sudoers 파일에서 `visudo`를 사용해 `Default requiretty` 라인을 주석 처리해야 합니다.

### 에이전트를 루트로 실행

`try_sudo`를 사용할 수 없는 경우, 대신 에이전트를 `root`로 실행할 수 있습니다.

<div class="alert alert-info">Linux에서 프로세스 daemon을 <code>루트</code>로 실행하는 것을 권고하지 않습니다. 에이전트는 오픈 소스이고 <a href="https://github.com/DataDog/datadog-agent">GitHub 리포지토리</a>에서 감사될 수 있습니다.</div>

에이전트를 `root`로 실행:
1. [에이전트를 중단][9]합니다.
2. `/etc/systemd/system/multi-user.target.wants/datadog-agent.service`를 열고 `[Service]` 속성 아래 `user`를 변경합니다.
3. [에이전트를 시작][10]합니다.

자세한 정보와 리눅스 머신에서 이용 가능한 메트릭의 기타 수집 방법이 궁금하신 분은 아래의 깃허브 관련 문제를 참조해주세요.

* https://github.com/DataDog/dd-agent/issues/853
* https://github.com/DataDog/dd-agent/issues/2033

## MacOS에서 에이전트를 시스템 daemon으로 실행할 때 권한 문제

에이전트를 `DD_SYSTEMDAEMON_INSTALL`과 `DD_SYSTEMDAEMON_USER_GROUP` 옵션을 사용해 daemon으로 시스템 전체 실행할 경우, `DD_SYSTEMDAEMON_USER_GROUP`에 사용한 사용자와 그룹이 유효하고 올바른 권한을 갖고 있는지 확인하세요.

## 참고 자료

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ko/agent/configuration/agent-commands/
[2]: /ko/agent/configuration/agent-log-files/
[3]: /ko/agent/faq/error-restarting-agent-already-listening-on-a-configured-port/
[4]: /ko/agent/faq/network/
[5]: /ko/agent/configuration/agent-commands/#start-the-agent
[6]: /ko/help/
[7]: /ko/integrations/process/
[9]: /ko/agent/configuration/agent-commands/#stop-the-agent
[10]: /ko/agent/configuration/agent-commands/#start-the-agent