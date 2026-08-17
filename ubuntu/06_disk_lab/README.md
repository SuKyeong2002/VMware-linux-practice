# 06. Linux Disk & Partition Management Practice

## 실습 개요
리눅스 디스크 파티셔닝 핵심 도구(`fdisk`, `gdisk`, `parted`) 및 블록 장치 조회 도구(`lsblk`, `blkid`)의 특성을 비교하고, `dd` 가상 디스크 이미지를 생성하여 MBR/GPT 파티션 테이블 생성 및 분할 실습을 검증합니다.

---

## 주요 명령어 및 실습

### 1. 블록 장치 및 파일시스템 정보 조회
- `lsblk`: 블록 장치(디스크, 파티션, loop 장치) 트리 구조 출력
- `lsblk -f`: 파일시스템 유형(`ext4`, `squashfs`), UUID, 마운트포인트 상세 출력
- `sudo blkid`: 모든 파티션의 고유 식별자(UUID) 및 포맷 유형(`TYPE`) 확인
- `sudo fdisk -l /dev/sda`: 특정 디스크의 섹터 수, 섹터 크기(512B), 파티션 라벨(`gpt`/`dos`) 상세 조회

### 2. 가상 디스크 생성 및 파티셔닝 실습 (안전 격리 환경)

#### ① 100MB 가상 디스크 생성
- `dd if=/dev/zero of=test_disk.img bs=1M count=100`

#### ② `fdisk`를 활용한 MBR 파티션 생성
- `fdisk test_disk.img`
  - `p`: 현재 파티션 테이블 확인 (Print)
  - `n`: 새 파티션 생성 (New) $\rightarrow$ `p` (Primary) $\rightarrow$ `1` (번호) $\rightarrow$ Enter 2회 (기본 섹터)
  - `w`: 변경 사항 저장 후 종료 (Write)

#### ③ `parted`를 활용한 GPT 라벨 변환 및 일괄 파티션 생성
- `parted -s test_disk.img mklabel gpt`: 디스크 라벨을 GPT 형식으로 초기화
- `parted test_disk.img print`: 파티션 테이블 및 라벨(`Partition Table: gpt`) 조회
- `parted -s test_disk.img mkpart primary ext4 1MB 50MB`: 1MB~50MB 구간에 ext4용 파티션 즉시 생성

---

## 파티셔닝 도구 비교

| 도구명 | 파티션 형식 | 최대 용량 | 인터페이스 특징 |
| :--- | :--- | :--- | :--- |
| **`fdisk`** | MBR (기본) / GPT | MBR 기준 2TB 제한 | 전통적인 대화형 CLI 도구 |
| **`gdisk`** | **GPT 전용** | 2TB 이상 지원 | GPT 파티션 전용 대화형 도구 |
| **`parted`** | MBR, GPT 모두 지원 | 2TB 이상 지원 | CLI 비대화형 스크립트 및 일괄 생성 최적화 |

---

## 블록 장치 주요 명명 규칙 및 개념

- **`sd[a-z]`**: SATA, SCSI, SSD, USB 외장 디스크 (예: `/dev/sda`)
- **`loop[0-9]`**: Snap 패키지나 디스크 이미지를 가상 블록 장치로 마운트한 루프 장치
- **`sr[0-9]`**: CD/DVD-ROM 광학 드라이브
- **`fd[0-9]`**: 레거시 플로피 디스크 드라이브
- **`MAJ:MIN`**: 주번호(장치 드라이버 종류 식별) : 부번호(개별 장치/파티션 식별)

---

## 빈출 함정 지문 정리
#### 1. MBR 파티션 방식의 `fdisk`로 4TB 단일 디스크를 온전히 하나의 파티션으로 할당할 수 있다. (x)
- MBR(Master Boot Record) 방식은 32비트 주소 체계를 사용하여 최대 2TB까지만 인식합니다. 2TB 초과 디스크는 GPT(`gdisk` 또는 `parted`)를 사용해야 합니다.

#### 2. `fdisk` 대화형 메뉴에서 작업을 취소하고 저장하지 않고 나가려면 `w`를 입력한다. (x)
- `w`(Write)는 변경 사항을 디스크에 저장하고 종료하는 명령이며, 저장하지 않고 취소하려면 `q`(Quit)를 입력해야 합니다.

#### 3. `parted` 명령어는 스크립트에서 자동화할 수 없으며 오직 대화형으로만 실행된다. (x)
- `parted -s` 옵션을 사용하면 스크립트나 명령줄에서 비대화형(non-interactive)으로 파티션을 한 줄로 일괄 생성/수정할 수 있습니다.

#### 4. `gdisk`는 MBR 파티션 테이블만을 전문적으로 다루기 위한 유틸리티이다. (x)
- `gdisk`의 'g'는 GPT를 의미하며, GPT 파티션 전용 관리 유틸리티입니다.
