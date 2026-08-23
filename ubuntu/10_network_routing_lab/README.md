# 10. Linux Network Routing, DNS & Security Practice

## 실습 개요
커널 라우팅 테이블(`route`, `netstat`, `ip route`)의 구조를 이해하고, 네트워크 및 DNS 진단 명령어(`ping`, `host`, `nslookup`)와 파일 접근/속성 제어 도구(`chattr`, `lsattr`, `setfacl`, `getfacl`)의 기능을 검증합니다.

---

## 핵심 명령어 체계 및 분류

| 카테고리 | 도구/명령어 | 주요 옵션/문법 | 핵심 기능 |
| :--- | :--- | :--- | :--- |
| **라우팅 조회** | `ip route show` | - | 커널 라우팅 테이블 조회 (표준) |
| **라우팅 관리** | `ip route add / del` | `[대역] via [GW] dev [NIC]` | 정적 라우팅 경로(Static Route) 수동 추가 및 삭제 |
| **네트워크 진단** | `ping` | `-c [횟수]`, `-i [간격]`, `-q` | ICMP 패킷 왕복 통신 테스트 및 네트워크 지연율 측정 |
| **DNS 해석** | `host`, `nslookup` | `[도메인] [지정DNS_IP]` | 도메인 이름을 IP로 해석 (공용 DNS 수동 지정 가능) |
| **파일 확장 속성** | `chattr`, `lsattr` | `+a`, `-a`, `+i`, `-i` | 파일 추가 전용(`a`), 완전 불변(`i`) 속성 제어 및 조회 |
| **확장 접근 제어** | `setfacl`, `getfacl` | `-m`, `-x`, `-b` | 특정 사용자/그룹별 개별 권한(ACL) 설정 및 확인 |

---

## 주요 명령어 문법

### 1. 라우팅 테이블 및 경로 제어
- `ip route show`: 현재 등록된 커널 라우팅 테이블 전체 출력
- `sudo ip route add 192.168.100.0/24 via 127.0.0.1 dev lo`: 특정 대역으로 가는 정적 라우팅 경로 수동 추가
- `sudo ip route del 192.168.100.0/24 via 127.0.0.1 dev lo`: 등록된 정적 라우팅 경로 제거

### 2. 네트워크 및 DNS 진단
- `ping -c 3 8.8.8.8`: 구글 공용 DNS IP로 ICMP 패킷을 3회 전송하여 물리적 통신 상태 점검
- `host www.jdcdfs.com`: DNS 서버에 A 레코드(공인 IP) 질의
- `host [도메인] 8.8.8.8`: 로컬 DNS 캐시를 우회하고 구글 공용 DNS(`8.8.8.8`)에 직접 질의

### 3. 파일 확장 속성 (`chattr` / `lsattr`)
- `sudo chattr +a [파일명]`: 파일 수정/삭제를 금지하고 내용 추가(Append only)만 허용
- `sudo chattr +i [파일명]`: root를 포함한 모든 사용자의 수정/삭제/추가를 차단(Immutable 동결)
- `lsattr [파일명]`: 적용된 확장 속성 플래그 상태 확인

### 4. 확장 접근 제어 목록 (`setfacl` / `getfacl`)
- `setfacl -m u:[사용자명]:rwx [파일명]`: 특정 사용자에게 개별 읽기/쓰기/실행 권한 부여
- `setfacl -x u:[사용자명] [파일명]`: 해당 사용자의 개별 ACL 권한 규칙 삭제
- `setfacl -b [파일명]`: 파일에 적용된 모든 확장 ACL 규칙 일괄 초기화
- `getfacl [파일명]`: 파일에 설정된 상세 ACL 권한 목록 조회

---

## 빈출 함정 지문 정리
#### 1. `chattr +i` 속성이 부여된 파일은 관리자(root) 권한으로 강제 삭제(`rm -f`)가 가능하다. (x)
- `+i` (Immutable) 속성이 걸린 파일은 최고 관리자(`root`)라도 속성을 해제(`chattr -i`)하기 전까지는 수정이나 삭제가 불가능합니다.

#### 2. 파일의 수정은 막고 내용 덧붙이기(추가)만 가능하도록 설정하는 명령어는 `setfacl -m +a [파일]`이다. (x)
- `setfacl`은 사용자/그룹별 권한(ACL)을 다루는 명령어이며, 추가 전용 속성은 `chattr +a [파일]`을 사용합니다.

#### 3. `host 도메인` 실행 시 `NXDOMAIN` 에러가 발생하는 것은 대상 서버의 방화벽이 요청을 차단했기 때문이다. (x)
- `NXDOMAIN`(Non-Existent Domain)은 방화벽 차단이 아니라, DNS 서버에 해당 도메인 이름 자체가 등록되어 있지 않음을 의미합니다.

#### 4. `ping` 테스트 시 상대방의 IP 주소를 알고 있어도 내부 DNS가 없으면 통신이 불가능하다. (x)
- IP 주소를 직접 지정하여 `ping [IP]`를 칠 때는 DNS 질의 과정(이름 해석)을 거치지 않으므로 정상 통신이 가능합니다.
