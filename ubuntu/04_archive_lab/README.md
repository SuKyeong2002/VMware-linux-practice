# 04. Linux Archive & Compression Practice

## 실습 개요
리눅스의 표준 단일 압축 도구(`gzip`, `bzip2`, `xz`)와 아카이브 도구인 `tar`, Windows 호환 포맷(`zip`)을 활용하여 파일 묶기, 압축, 해제, 수정 및 압축 파일 실시간 조회 명령어(`zcat` 등)를 검증합니다.

---

## 주요 명령어 및 실습

### 1. 단일 압축 및 해제 실습 (`-k` 옵션으로 원본 유지)
- `gzip -k file_a.txt` (gzip 압축 -> file_a.txt.gz)
- `gzip -d file_a.txt.gz` 또는 `gunzip file_a.txt.gz` (gzip 해제)
- `bzip2 -k file_b.txt` (bzip2 압축 -> file_b.txt.bz2)
- `bzip2 -d file_b.txt.bz2` 또는 `bunzip2 file_b.txt.bz2` (bzip2 해제, 충돌 시 -f 옵션 사용)
- `xz -k file_c.txt` (xz 압축 -> file_c.txt.xz)
- `xz -d file_c.txt.xz` 또는 `unxz file_c.txt.xz` (xz 해제)

### 2. `tar` 아카이브 및 동시 압축
- `tar -cvf backup.tar file_a.txt file_b.txt file_c.txt` (단순 묶기)
- `tar -czvf backup.tar.gz file_*.txt` (gzip 동시 압축)
- `tar -cjvf backup.tar.bz2 file_*.txt` (bzip2 동시 압축)
- `tar -cJvf backup.tar.xz file_*.txt` (xz 동시 압축)
- `tar -tvf backup.tar.gz` (압축 해제 없이 내부 파일 목록 확인)
- `tar -xzvf backup.tar.gz -C ./target_folder` (-C 옵션으로 특정 디렉터리에 해제)

### 3. `tar` 고급 관리 실습
- `tar -czvf backup.tar.gz --exclude='exclude_folder' .` (특정 폴더 제외 후 압축)
- `tar -rvf backup.tar new_file.txt` (기존 .tar 아카이브에 새 파일 추가)
- `tar -uvf backup.tar new_file.txt` (기존 .tar 내 수정된 최신 파일만 갱신/추가)

### 4. 압축 파일 내용 실시간 조회 및 검색 (압축 해제 불필요)
- `zcat test.log.gz` / `zgrep "pattern" test.log.gz` (gzip 텍스트 조회/검색)
- `bzcat bz_test.log.bz2` / `bzgrep "pattern" bz_test.log.bz2` (bzip2 텍스트 조회/검색)
- `xzcat xz_test.log.xz` / `xzgrep "pattern" xz_test.log.xz` (xz 텍스트 조회/검색)

---

## 빈출 함정 지문 정리
#### 1. gzip, bzip2, xz는 별도 옵션 없이 실행해도 원본 파일이 유지된다. (x)
- 기본 실행 시 원본 파일이 삭제되고 압축본만 남으므로 원본 유지를 위해 `-k`(`--keep`) 옵션이 필요합니다.

#### 2. `tar.xz` 형식으로 묶으면서 압축할 때 사용하는 옵션은 소문자 `-j`이다. (x)
- 소문자 `-j`는 `bzip2`이며, `xz`는 대문자 **`-J`**를 사용해야 합니다.

#### 3. 이미 압축된 `backup.tar.gz` 파일에 `-r` 옵션으로 새 파일을 덧붙일 수 있다. (x)
- `-r`(추가) 및 `-u`(갱신) 옵션은 압축되지 않은 순수 `.tar` 아카이브 파일에서만 동작합니다.

#### 4. `tar -cvfz backup.tar.gz [대상]` 명령어로 정상 압축할 수 있다. (x)
- `-f` 옵션 바로 뒤에는 반드시 생성할 파일명이 와야 하므로 `-f`는 옵션 문자열의 맨 마지막에 위치해야 합니다 (`-czvf`).
