# 07. Linux LVM (Logical Volume Manager) Practice

## 실습 개요
LVM의 3계층 구조(물리 볼륨 $\rightarrow$ 볼륨 그룹 $\rightarrow$ 논리 볼륨)와 단계별 관리 명령어(`*create`, `*scan`, `*display`, `*s`, `*extend`, `*remove`)의 역할을 검증합니다.

---

## LVM 3계층 구조 및 명령어 체계

| 계층 (Layer) | 스캔(Scan) | 간략 조회(Summary) | 상세 조회(Display) | 생성(Create) | 삭제(Remove) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **PV (물리 볼륨)** | `pvscan` | `pvs` | `pvdisplay` | `pvcreate` | `pvremove` |
| **VG (볼륨 그룹)** | `vgscan` | `vgs` | `vgdisplay` | `vgcreate` | `vgremove` |
| **LV (논리 볼륨)** | `lvscan` | `lvs` | `lvdisplay` | `lvcreate` | `lvremove` |

---

## 주요 명령어 문법

### 1. 물리 볼륨 (PV)
- `sudo pvcreate [장치명...]`: 디스크 파티션을 PV로 초기화
- `sudo pvscan`: 시스템의 모든 물리 볼륨 장치를 스캔

### 2. 볼륨 그룹 (VG)
- `sudo vgcreate [VG이름] [PV장치...]`: PV들을 묶어 단일 볼륨 그룹 풀(Pool) 생성
- `sudo vgscan`: 시스템 내의 볼륨 그룹 목록 스캔 및 메타데이터 색인 갱신
- `sudo vgextend [VG이름] [새PV장치]`: 기존 볼륨 그룹에 새 디스크(PV)를 추가하여 총 용량 증설

### 3. 논리 볼륨 (LV)
- `sudo lvcreate -L [용량] -n [LV이름] [VG이름]`: 지정한 용량으로 논리 볼륨 분할
- `sudo lvextend -L +[추가용량] /dev/[VG이름]/[LV이름]`: 서비스 중단 없이 논리 볼륨 용량 확장
- `sudo lvscan`: 활성화/비활성화된 논리 볼륨 상태 출력

---

## 빈출 함정 지문 정리
#### 1. `vgscan` 명령어는 논리 볼륨(LV)의 크기를 동적으로 확장하는 명령어이다. (x)
- 볼륨 그룹을 검색/스캔하는 명령어이며, 용량 확장은 `lvextend` 또는 `vgextend`를 사용합니다.

#### 2. 볼륨 그룹(VG)을 삭제할 때 논리 볼륨(LV)이 남아있어도 `vgremove`가 즉시 수행된다. (x)
- LVM 자원은 역순(LV 삭제 $\rightarrow$ VG 삭제 $\rightarrow$ PV 삭제)으로 안전하게 제거해야 합니다.

#### 3. `pvcreate` 명령을 실행하기 전 해당 디스크 파티션의 Hex Type은 반드시 `83`이어야 한다. (x)
- MBR 기준 LVM 전용 파티션 타입 코드는 **`8e` (Linux LVM)**입니다.
