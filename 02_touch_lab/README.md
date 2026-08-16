# 02. Linux `touch` & Timestamp Practice

## 실습 개요
`touch` 명령어를 활용하여 빈 파일 생성 및 파일의 타임스탬프(atime, mtime, ctime) 특성을 이해하고, 옵션별 시간 변경 동작을 검증합니다.

---

## 주요 실습 내용

### 1. 빈 파일 생성

#### 단일 빈 파일 생성 (0바이트)
- touch touch_file_single.txt

#### 다중 파일 생성
- touch touch_file_a.txt touch_file_b.txt touch_file_c.txt

#### 중괄호 확장(Brace Expansion)을 이용한 연속 파일 생성
- touch touch_file_log_{1..5}.txt

### 2. 타임스탬프 상세 조회 (atime, mtime, ctime etc)
- stat touch_file_single.txt

### 3. 시간 갱신 옵션

#### 수정 시간(mtime)만 현재 시각으로 갱신
- touch -m touch_file_single.txt

#### 접근 시간(atime)만 현재 시각으로 갱신
- touch -a touch_file_single.txt

#### 특정 날짜/시간으로 강제 지정 (형식: YYYYMMDDhhmm.ss)
- touch -t 202608151200.00 touch_file_single.txt

#### 파일이 없을 경우 새로 생성 방지
touch -c nonexist_file.txt
