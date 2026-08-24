# 11. Linux NFS Network File Sharing Practice

## 실습 개요
네트워크 파일 공유 시스템인 NFS(Network File System) 서버 및 클라이언트 환경을 구축하고, 공유 설정 파일(`/etc/exports`), 제어 명령어(`exportfs`), RPC 포트 매퍼(`rpcinfo`), 상태 통계(`nfsstat`) 및 관련 도구들의 실무 운용과 자격 검정 핵심 이론을 검증합니다.

---

## NFS 핵심 설정 및 옵션 체계 (`/etc/exports`)

| 설정 항목 / 옵션 | 기능 설명 | 기본값 여부 | 비고 |
| :--- | :--- | :--- | :--- |
| **`rw` / `ro`** | 읽기/쓰기(`rw`) 또는 읽기 전용(`ro`) 권한 부여 | `ro` | 기본값은 읽기 전용 |
| **`sync` / `async`** | 디스크 쓰기 완료 후 응답(`sync`) vs 버퍼 처리 후 즉시 응답(`async`) | `sync` | 데이터 무결성 보장 |
| **`root_squash`** | 클라이언트의 `root`를 서버의 `nobody`(`nfsnobody`) 계정으로 매핑 | **기본값** | root 권한 탈취 방지 |
| **`no_root_squash`** | 클라이언트의 `root` 권한을 서버에서도 그대로 인정 | - | 보안상 매우 위험 |
| **`all_squash`** | `root`를 포함한 **모든 접속자를 익명(`nobody`) 계정으로 매핑** | - | 다중 사용자 공용 공유 시 사용 |
| **`no_all_squash`** | 일반 사용자의 UID/GID를 그대로 유지 | **기본값** | 동일 UID 매핑 필요 |
| **`no_subtree_check`** | 하위 디렉터리 권한 검사를 생략하여 전송 속도 및 안정성 향상 | - | 실무 권장 옵션 |

---

## 주요 명령어 문법

### 1. NFS 서버 공유 관리 (`exportfs`)
- `sudo exportfs -ra`: `/etc/exports` 설정 파일을 다시 읽어 변경 사항을 즉시 반영 (새로고침)
- `sudo exportfs -v`: 현재 NFS로 서비스 중인 디렉터리와 세부 적용 옵션을 상세히 출력
- `sudo exportfs -u [호스트:경로]`: 지정한 디렉터리의 NFS 공유를 즉시 중단
- `sudo exportfs -ua`: 서비스 중인 모든 NFS 공유 디렉터리를 일괄 내보내기 해제

### 2. 마운트 목록 및 RPC 상태 진단 (`showmount`, `rpcinfo`)
- `showmount -e [서버IP]`: NFS 서버가 외부에 공개(내보내기)하고 있는 디렉터리 목록 조회
- `showmount -a [서버IP]`: 현재 서버의 공유 디렉터리를 마운트 중인 클라이언트 IP/호스트 목록 출력
- `rpcinfo -p [서버IP]`: RPC 포트매퍼(포트 111)에 등록된 서비스(`nfs(2049)`, `mountd` 등) 및 포트 번호 조회
- `rpcinfo -t [서버IP] [프로그램명]`: TCP 프로토콜을 통해 특정 RPC 서비스의 응답 대기 상태 점검

### 3. 클라이언트 마운트 및 통계 분석 (`mount`, `nfsstat`)
- `sudo mount -t nfs [서버IP]:[서버경로] [로컬마운트포인트]`: 원격 NFS 디렉터리를 로컬 파일시스템에 마운트
- `nfsstat`: NFS 서버 및 클라이언트의 전체 RPC 통신 통계 출력
- `nfsstat -s`: NFS 서버 관점의 요청 수(`calls`) 및 오류 통계만 필터링 출력
- `nfsstat -c`: NFS 클라이언트 관점의 전송 통계만 필터링 출력

---

## 빈출 함정 지문 정리
#### 1. `/etc/exports` 설정 파일에서 호스트 대상(`*`)과 옵션 괄호`(` 사이에 공백을 두어야 정상 인식된다. (x)
- 호스트 명칭과 괄호 사이에 공백이 들어가면 문법 오류(`syntax error: bad option list`)가 발생하므로 반드시 붙여서 `*(rw,...)` 형태로 작성해야 합니다.

#### 2. `all_squash` 옵션은 root 사용자를 제외한 일반 사용자의 권한만 nobody로 매핑하는 옵션이다. (x)
- `all_squash`는 root 사용자와 일반 사용자 모두(전체)를 nobody 계정으로 강등시킵니다. (root 사용자만 매핑하는 옵션은 `root_squash`입니다.)

#### 3. `/etc/exports` 파일을 수정한 후에는 반드시 NFS 데몬(`systemctl restart nfs-server`)을 재시작해야만 적용된다. (x)
- 데몬을 재시작하지 않고 `exportfs -ra` 명령어를 실행하면 서비스 중단 없이 즉시 변경 사항을 반영할 수 있습니다.

#### 4. NFS는 기본적으로 고정 포트 2049(NFS)와 함께 보조 프로세스 관리를 위해 RPC 포트매퍼(111번 포트)를 사용한다. (o)
