# 12. Linux Mail Service & Alias Practice Lab

리눅스 메일 전송 데몬(MTA: Postfix)과 메일 유틸리티(MUA: Mailutils)를 구축하고, 메일 별칭(`/etc/aliases`)을 통한 계정 포워딩 및 1:N 그룹 메일 발송 메커니즘을 실습하는 랩입니다.

---

## 1. 실습 환경 개요
* **핵심 패키지:**
  * **MTA (Mail Transfer Agent):** Postfix (`Local only` 모드)
  * **MUA (Mail User Agent):** GNU Mailutils (`mail`)
* **핵심 파일:** `/etc/aliases`, `/etc/aliases.db`

---

## 2. 실습 명령어 모음
### 2-1. 메일 패키지 설치 및 환경 설정
#### 2-1-1. 패키지 업데이트 및 메일 서버/도구 설치
- sudo apt update
- sudo apt install -y postfix mailutils

#### 2-1-2. 설치 대화창(GUI) 선택 옵션:
- General type of mail configuration: [Local only]
- System mail name: 기본 호스트명 유지

### 2-2. 메일 별칭(/etc/aliases) 설정
#### 2-2-1. 별칭 설정 파일 열기
- sudo nano /etc/aliases
- 파일 하단에 아래 규칙을 추가하고 저장합니다 (Ctrl + O -> Enter -> Ctrl + X)

```
postmaster:    root

# 1:1 전달 (admin으로 오는 메일을 sukyeong 계정으로 포워딩)
admin: sukyeong

# 1:N 그룹 메일 (devteam으로 오는 메일을 sukyeong과 root 둘 다에게 동시 배달)
devteam: sukyeong, root
```

### 2-3. 해시 DB 갱신 및 서비스 재기동 
#### 2-3-1. 텍스트 파일을 바이너리 해시 DB(/etc/aliases.db)로 컴파일
- sudo newaliases

#### 2-3-2. 변경된 설정을 적용하기 위해 메일 데몬 재시작
- sudo systemctl restart postfix

### 2-4. 4단계: 메일 발송 및 포워딩 수신 검증
#### 2-4-1. [테스트 1] 1:1 별칭 포워딩 검증 (admin -> sukyeong)

```
# admin 별칭으로 메일 전송
echo "mail aliase test content" | mail -s "mail test title" admin

# sukyeong 계정 수신함 확인
mail
# -> 목록에서 메일 번호(1) 입력 후 확인, 'q'를 눌러 종료 (mbox로 자동 보관)
```

#### 2-4-2. [테스트 2] 1:N 그룹 메일 다중 수신 검증 (devteam -> root, sukyeong)

``` 
# devteam 그룹 별칭으로 메일 전송
echo "Group mail test" | mail -s "DevTeam Notice" devteam

# 1) root 계정 메일함 확인 (관리자 권한)
sudo mail -u root

# 2) 본인(sukyeong) 계정 메일함 확인
mail
```

## 3. 핵심 메커니즘 및 개념 정리
### 3-1. sudo newaliases를 실행해야 하는 이유
- 검색 성능 (O(1)): /etc/aliases는 텍스트 파일이므로 매 메일 수신 시마다 순차 탐색(Full Scan)하면 속도가 저하됩니다. 이를 바이너리 해시 DB인 /etc/aliases.db로 변환하여 고속 색인을 가능하게 합니다.
- 데이터 무결성: 관리자가 텍스트 파일을 수정하는 도중 불완전한 설정 파일을 메일 데몬이 읽어 발생하는 충돌 및 오류를 방지합니다.

### 3-2. q vs x 종료 및 mbox 보관 원리
- q (Quit): 읽은 메일을 공용 수신함(/var/mail/사용자명)에서 개인 보관함(~/mbox)으로 자동 이동합니다. (시스템 파티션 /var의 용량 고갈 방지 및 새 메일/보관 메일 분리)
- x (Exit): 상태 변경 없이 받은 편지함에 그대로 둔 채 빠져나옵니다.
- 보관함 메일 열기: mail -f ~/mbox

### 3-3. 메일 저장 포맷 비교
- mbox: 단일 파일에 모든 메일을 텍스트로 이어 붙이는 전통적인 방식 (CLI 표준)
- Maildir: ~/Maildir/{new,cur,tmp} 구조로 메일 1통당 1개의 독립 파일로 저장하는 현대 표준 방식 (동시 접근 안전, 파일 락킹 불필요)
