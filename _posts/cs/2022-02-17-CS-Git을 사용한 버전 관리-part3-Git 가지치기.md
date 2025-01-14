---
layout: post
title:  "[Git] Part3 - Git 가지치기"
excerpt: "Elice강의 - Git을 사용한 버전 관리- Part3 정리"

categories:
  - cs
tags:
  - [Git]

toc: true
toc_sticky: true
 
date: 2022-02-17
last_modified_at: 2022-02-25
---

* 해당 블로그는 Elice에서 제공하는 'Git을 사용한 버전관리' 강의를 듣고 개인적으로 정리한 글입니다.

#### 전체 과정
Part 1 - Git 이란?
Part 2 - Git 시작하기
**Part 3 - Git 가지 치기**
Part 4 - Git 원격 저장소  

# Part 3 - Git 가지 치기
## Ch 1. Git Branch
### Git branch 란? 
-. 독립적으로 어떤 작업을 진행하기 위한 개념

-. 각각의 브랜치는 다른 브랜치의 영향을 받지 않음 (완전히 독립적)

-. 브랜치 나누는 기준은 기능단위/작업단위 등 각자 정하기 나름.

### Git branch 종류
1) 메인 브랜치 : 배포할 수 있는 수준의 안정적인 브랜치 ('master' 브랜치)

2) topic 브랜치 : 기능 추가나 버그 수정과 같은 단위 작업을 위한 브랜치. 수시로 생성 및 삭제 될 수 있음

### Git branch 관련 명령어 
* 현재 생성되어 있는 브랜치 확인: ```git branch```
* 브랜치 생성: ```git branch {생성할 branch 명}```
* 브랜치 전환 (이동)  : ```git checkout {전환할 branch 명}```
* Git Navigation: ```git checkout {snapshot hash; 16진수로 이루어진 snapshot의 핵심 코드}``` 

-. Git navigation이란? ```git log``` 로 확인한 snapshot을 넘나드는 것 (HEAD 이동).

* snapshot hash 코드 확인: ``` git log --pretty=oneline```. snapshot 의 hash 값을 이용하여 과거의 파일 내용 확인 가능
* commit graph 확인: ``` git log --graph --all ```.  (추가 옵션으로 더 깔끔하게 보기; pretty 옵션) ``` git log --pretty=oneline --graph --all
* git merge: ``` git merge {병합시킬 브랜치명} ```
* merge 된 브랜치 확인: ``` git branch --merged ```
* git branch 삭제: ``` git branch -d {branch명} ```
* git 상태 확인: ``` git status ```

#### [Remind] 사용 가능한 git log 옵션: 

``` git log --oneline ``` 

``` git log --pretty=oneline ```

``` git log --all ```

``` git log --graph ```

#### 참고:  [Linux] 터미널 상에서 파일 생성
* nano 에디터 이용 방법: ``` nano {생성/수정할 파일명; new_file.py} ``` # nano 에디터를 닫을 때는 ctrl X -> Y -> Enter
* echo 명령어 활용


## 브랜치 병합하기 (Ch 2 & 3)
## Ch 2. Git fast forward (병합 merge 방식)
-. fast forward 병합: 기존 master 브랜치에서 다른 것 변경된 것 없이 병합할 브랜치의 내용만 추가 (update)  

-. 예시로 설명. 'like_feature' Branch의 working directory 에서 새로운 정보를 넣어 commit  해보자.

step 1) like_feature 로 브랜치 전환 (이동)

``` git checkout {전환할 브랜치명; like_feature} ``` # HEAD pointer 가 'like_feature' 브랜치를 가리키게 됨.

step 2) 신규 작업 후 commit -> 새로운 check point 가 생성됨.


## Ch 3. Git Merge 
### fast-forward 방식으로 병합해보기
-. (Ch 2의 step 1, 2와 이어짐) 예시로 설명. 'master' 브랜치에 fast-forward 방법으로 'like_feature' 브랜치 병합하기 

step 3) master 브랜치로 이동 후 like_feature 브랜치 병합 (fast forward 방식으로 병합)

``` git checkout {전환할 브랜치명; master} ```

``` git merge {병합시킬 브랜치명; like_feature} ``` # like_feature 브랜치의 내용이 master 브랜치로 업데이트 (merge).  

### 갈라지는 branch
-. 각각의 branch 는 다른 branch 의 영향을 받지 않기 때문에, 여러 작업을 동시에 진행할 수 있음.

-. 각각의 branch 의 working directory 에서 같은 파일의 내용을 다르게 수정하는 것도 가능. (단, 수정한 코드 구문이 같으면, 충돌 발생)

-. (merge코드의 경우, fast-forward 방식이든 다른 방식이든 코드는 동일) ``` git checkout {전환할 브랜치명; master} ``` ``` git merge {병합시킬 브랜치명; like_feature} ``` # 중심이 될 브랜치 (ex. master) 의 내용과 병합할 브랜치 (ex. like_feature) 내용이 합쳐진 상태로 merge 됨.

### fast-forward 방식 병합 vs 갈라진 branch 병합 차이점
-. fast-forward 방식 병합 후엔, 병합의 중심이 될 브랜치 (ex. master) 와 병합할 브랜치 (ex. like_feature) 가 바라보는 체크포인트 가 ** 같아짐 **

-. 갈라진 브랜치 병합 후엔, 병합의 중심이 될 브랜치 (ex. master) 와 병합할 브랜치 (ex. like_feature) 가 바라보는 체크포인트 가 ** 다름 **

### git branch 삭제
-. 보통 topic 브랜치의 경우, 기능개발이나 일종의 작업이 끝난 후 삭제해주는 것이 일반적.l
* git branch 삭제: ``` git branch -d {branch명} ```

## Ch 4. conflict 해결
### merge conflict
-. Merge 할 두 branch에서 '같은 파일'을 동시에 변경 했을 때 충돌 발생 가능성 존재. 이런 경우, git 사용자가 "직접" 충돌을 해결하여 merge 수행 가능

step 1) ``` git status ``` 명령어로 어느 '파일'에서 충돌이 발생했는지 확인
* 양쪽의 브랜치에서 동일 파일에 동시에 접근했을 때 ``` # both modified:  {충돌난 파일명} ``` 문구가 출력됨

step 2) 충돌난 파일 수정 (수정 완료 후 , git 에서 표시한 '<<<...', '===...', '>>>...' 가 포함된 행 삭제)

step 3) 수정 완료 후 ``` git add ```, ``` git commit ``` 과정을 거쳐 다시 merge 수행 (commit 시, 자동으로 merge 가 수행된다.)

### git merge 충돌 방지 tip
-. 'master' Branch 의 변화를 지속적으로 가져와서 충돌이 발생하는 부분을 제거.

-. 제일 좋은 방법은, 'master'의 브랜치의 변경을 최소화하는 것. 이유: master 브랜치는 배포가 가능한 수준의 안정적인 부분만을 가지고 있어야 한다.
