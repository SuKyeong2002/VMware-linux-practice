# 01. Linux Disk Quota Practice

## 📌 실습 개요
가상 디스크 이미지를 생성하여 루프백 디바이스로 마운트하고, 사용자별 디스크 용량 제한(Block Quota) 및 파일 개수 제한(Inode Quota)을 설정하여 초과 차단 동작을 검증합니다.

---

## 🛠️ 주요 명령어

### 1. 환경 구성 및 쿼터 활성화
- **가상 디스크 생성:** `dd if=/dev/zero of=./disk.img bs=1M count=100`
- **파일시스템 포맷:** `mkfs.ext4 -F ./disk.img`
- **마운트 포인트 생성 및 마운트:** `mkdir -p ./mount_point && sudo mount -o loop,usrquota ./disk.img ./mount_point`
- **쿼터 DB 생성 및 활성화:** `sudo quotacheck -cum ./mount_point && sudo quotaon -v ./mount_point`
- **일반 사용자 쓰기 권한 부여:** `sudo chmod 777 ./mount_point`

### 2. 쿼터 상태 확인
- **전체 사용자 쿼터 현황 리포트:** `sudo repquota -vs ./mount_point`
- **개인 쿼터 상세 조회:** `quota -vs -f ./mount_point`

---

## 🧪 실습 및 동작 검증

### 실습 1. 디스크 용량 제한 (Block Quota)
- **제한 설정 (Soft 3MB / Hard 5MB):**  
  `sudo setquota -u $USER 3000 5000 0 0 ./mount_point`
- **Soft Limit 초과 테스트 (4MB 파일 생성):**  
  `dd if=/dev/zero of=./mount_point/test_4m bs=1M count=4`  
  *(결과: 3MB 초과로 경고 마크 `+-` 및 7일 유예 기간 `grace: 7days` 발생)*
- **Hard Limit 초과 테스트 (추가 2MB 쓰기 시도):**  
  `dd if=/dev/zero of=./mount_point/test_extra bs=1M count=2`  
  *(결과: 총 5MB 도달 시 `Disk quota exceeded` 에러 출력과 함께 쓰기 차단)*

### 실습 2. 파일 개수 제한 (Inode Quota)
- **제한 설정 (용량 무제한 / Inode Soft 5개 / Hard 10개):**  
  `sudo setquota -u $USER 0 0 5 10 ./mount_point`
- **Inode 초과 테스트 (12개 빈 파일 생성 시도):**  
  `for i in {1..12}; do touch ./mount_point/file_$i; done`  
  *(결과: 용량이 남아있어도 10개까지만 생성되고 11번째 파일부터 `Disk quota exceeded` 발생)*

---

## 💡 핵심 요약
- **Block Quota vs Inode Quota:** 디스크 저장 용량(KB) 제한 vs 파일/디렉터리 개수 제한
- **Soft Limit:** 경고 한도. 초과 시 유예 기간(Grace Period) 내에 정리 필요
- **Hard Limit:** 절대 한도. 초과 시 즉시 파일 생성/쓰기 차단
EOF

## 💡 핵심 요약
- **Block Quota vs Inode Quota:** 디스크 저장 용량(KB) 제한 vs 파일/디렉터리 개수 제한
- **Soft Limit:** 경고 한도. 초과 시 유예 기간(Grace Period) 내에 정리 필요
- **Hard Limit:** 절대 한도. 초과 시 즉시 파일 생성/쓰기 차단
