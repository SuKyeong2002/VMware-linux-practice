# 09. Linux System Logging & User Activity Practice

## 실습 개요
리눅스 시스템의 텍스트 로그 파일(`syslog`, `auth.log`)과 바이너리 로그 파일(`utmp`, `wtmp`, `btmp`, `lastlog`), 사용자 활동 추적 명령어의 체계와 배포판별 차이점을 검증합니다.

---

## 로그 파일 분류 및 명령어 체계

| 로그 파일명 | 파일 형식 | 어원 및 기록 내용 | 열람 명령어 | 특징 및 권한 |
| :--- | :--- | :--- | :--- | :--- |
| **`/var/run/utmp`** | 바이너리 | **U**ser tmp (현재 실시간 접속자 정보) | `w`, `who`, `users` | 부팅 시 초기화 / 실시간 메모리 동기화 |
| **`/var/log/wtmp`** | 바이너리 | **W**ho tmp (전체 로그인/로그아웃/재부팅 이력) | `last` | 누적 기록 / `-f` 옵션으로 파일 강제 지정 |
| **`/var/log/btmp`** | 바이너리 | **B**ad tmp (로그인 실패 및 침입 시도 기록) | `lastb` (우분투: `last -f`) | 보안상 `sudo`(root) 권한 필수 |
| **`/var/log/lastlog`** | 바이너리 / DB | **Last** Log (계정별 가장 최근 1회 로그인 기록) | `lastlog` (우분투: `lastlog2`) | `-u` 옵션으로 특정 사용자 지정 가능 |
| **`/var/log/auth.log`**<br>(CentOS: `secure`) | 텍스트 | 인증/보안 로그 (`sudo`, `su`, `ssh`, 비밀번호 변경) | `cat`, `tail`, `grep` | 보안 및 PAM 모듈 인증 내역 기록 |
| **`/var/log/syslog`**<br>(CentOS: `messages`) | 텍스트 | 전체 시스템 운영, 데몬 동작, 하드웨어 알림 | `cat`, `tail`, `journalctl` | 운영체제 전반의 표준 종합 상황 일지 |

---

## 주요 명령어 문법

### 1. 바이너리 로그 조회
- `last`: `/var/log/wtmp` 파일을 읽어 전체 로그인 및 시스템 재부팅 이력 출력
- `last -n [줄수]`: 가장 최근 접속 기록 중 지정한 줄 수만큼만 출력
- `sudo last -f /var/log/btmp` (또는 `sudo lastb`): 로그인 실패 이력 확인
- `lastlog -u [계정명]` (또는 `lastlog2 -u [계정명]`): 특정 계정의 마지막 로그인 일시/접속지 확인

### 2. 실시간 접속자 및 텍스트 로그 모니터링
- `w`: 현재 로그인한 사용자 정보, CPU 부하율, 실행 중인 명령어(WHAT) 상세 출력
- `who`: 현재 접속 중인 사용자의 계정명, 접속 터미널(tty/pts), 접속 일시 출력
- `sudo tail -f /var/log/auth.log`: 실시간 인증 및 보안 로그 스트리밍 감시
- `logger -t [태그] -p [우선순위] "메시지"`: 시스템 로그(syslog)에 사용자가 직접 테스트 메시지 주입

### 3. systemd 저널 로그 (`journalctl`)
- `journalctl -b`: 이번 부팅 이후에 발생한 시스템 저널 로그만 출력
- `journalctl -f`: 저널 로그 실시간 스트리밍 출력
- `journalctl -u [서비스명]`: 특정 시스템 서비스(예: ssh, cron)의 로그만 필터링

---

## 빈출 함정 지문 정리
#### 1. `/var/log/wtmp` 파일은 `cat` 명령어로 내용을 손쉽게 확인할 수 있다. (x)
- 바이너리(이진) 형식의 파일이므로 `cat`으로 열면 글자가 깨지며, 전용 명령어인 `last`를 사용해야 합니다.

#### 2. `lastb` 명령어는 전체 성공한 로그인 및 재부팅 이력을 조회하는 명령어이다. (x)
- `lastb`는 로그인에 실패한 불량 접속(Bad tmp) 기록을 조회하며, 성공/재부팅 이력은 `last`로 조회합니다.

#### 3. 현재 로그인 중인 사용자의 목록과 작업 현황은 `/var/log/lastlog`에 실시간으로 기록된다. (x)
- 현재 실시간 접속자 정보는 `/var/run/utmp`에 기록되며, `lastlog`는 각 사용자별 '가장 마지막 로그인 1회' 시점만을 기록합니다.

#### 4. CentOS의 `/var/log/secure`에 대응하는 데비안/우분투 계열의 보안 인증 로그 파일은 `/var/log/auth.log`이다. (o)
