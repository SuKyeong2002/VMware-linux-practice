# 05. Linux Kernel Module Management Practice

## 실습 개요
리눅스 커널 모듈(LKM)을 관리하는 핵심 도구(`lsmod`, `insmod`, `rmmod`, `modprobe`, `modinfo`, `depmod`)의 동작 원리와 의존성 처리 차이를 실습하고 검증합니다.

---

## 주요 명령어 및 실습

### 1. 모듈 조회
- `lsmod`: 현재 커널에 적재된 모듈 목록 출력 (`/proc/modules` 참조)
- `modinfo [모듈명]`: 모듈 파일 경로, 라이선스, 작성자, 파라미터, 의존성(`depends`) 상세 정보 확인

### 2. `modprobe` (의존성 자동 해결 - 권장 표준)
- `sudo modprobe [모듈명]`: `/lib/modules/$(uname -r)/modules.dep`를 참조하여 의존성 모듈까지 함께 자동 적재
- `sudo modprobe -r [모듈명]`: 의존 관계를 고려하여 안전하게 모듈 언로드(제거)
- `sudo modprobe -c`: 현재 적용된 모듈 관련 설정 파일(`modprobe.d`) 내용 전체 출력

### 3. `insmod` & `rmmod` (단일 모듈 수동 제어)
- `sudo insmod [모듈경로.ko]`: 특정 경로의 모듈 파일을 직접 커널에 적재 (의존성 해결 불가)
- `sudo rmmod [모듈명]`: 적재된 모듈을 커널에서 제거 (의존성 해결 불가)

### 4. 모듈 의존성 갱신
- `sudo depmod -a`: 모듈 간 의존성 목록 파일(`modules.dep`)을 최신 상태로 갱신

---

## insmod vs modprobe 비교

| 비교 항목 | insmod | modprobe |
| :--- | :--- | :--- |
| **의존성 자동 해결** | **불가능** (의존 모듈 없으면 적재 실패) | **가능** (연관 모듈 자동 적재) |
| **인자 형태** | 모듈 **전체 파일 경로** (`.ko` 포함) | **모듈 이름**만 지정 |
| **모듈 제거 기능** | 별도 명령어(`rmmod`) 필요 | **`-r`** 옵션으로 자체 제거 가능 |
| **참조 설정 파일** | 없음 | `/etc/modprobe.d/*.conf` |

---

## 빈출 함정 지문 정리
#### 1. `insmod` 명령어는 모듈 간 의존성을 자동으로 파악하여 필요한 모듈을 함께 적재한다. (x)
- 의존성을 자동으로 해결하는 명령어는 `modprobe`이며, `insmod`는 단일 `.ko` 파일만 강제로 올리므로 의존성 해결이 불가능합니다.

#### 2. `insmod`와 `rmmod`는 모두 인자로 모듈의 파일명(`.ko`)을 지정해야 한다. (x)
- `insmod`는 파일 전체 경로(예: `dummy.ko`)가 필요하지만, `rmmod`는 커널에 등록된 **모듈 이름(예: `dummy`)**만 지정합니다.

#### 3. `lsmod` 명령어는 `/etc/modules` 파일의 내용을 화면에 출력한다. (x)
- `lsmod`는 `/proc/modules`의 실시간 커널 메모리 정보를 읽어 가공하여 출력합니다.

#### 4. `depmod` 명령어가 생성하는 의존성 파일의 경로는 `/etc/modprobe.conf`이다. (x)
- 의존성 파일은 `/lib/modules/$(uname -r)/modules.dep`에 저장됩니다.
