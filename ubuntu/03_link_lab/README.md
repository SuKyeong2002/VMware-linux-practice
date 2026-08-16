# 03. Linux `ln` Command & Link File Practice

## 실습 개요
`ln` 명령어를 활용하여 하드 링크(Hard Link)와 심볼릭 링크(Symbolic Link)를 생성하고, Inode 번호, 링크 카운트, 원본 삭제 시의 동작 차이를 검증합니다.

---

## 주요 명령어 및 실습

### 1. 링크 생성
- echo "Hello, Linux Master!" > origin.txt
- cat origin.txt

### 1-1. 하드 링크 생성
- ln origin.txt hard_link.txt

### 1-2. 심볼릭 링크 생성
- ln -s origin.txt sym_link.txt

### 2. Inode 및 링크 정보 확인
- ls -li
- origin Inode number = hard Inode number =! sym Inode number

## 빈출 함정 지문 정리
#### 1. 하드 링크는 다른 파일시스템(파티션)에 걸쳐 생성할 수 있다. (x) 
#### 2. 심볼릭 링크를 삭제하면 원본 파일도 함께 삭제된다. (x)
#### 3. 하드 링크는 디렉터리에 대해 생성할 수 있다. (x)
