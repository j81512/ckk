----------------------
### 프로젝트명 : CKK
### [Site 바로가기](park.jh92.kro.kr/CKK)
### [Project 상세정보](https://www.notion.so/CKK-Cooking-King-91c88bfdf1ee45a7bf71995041b7cc42)
----------------------

### Sample ID 및 Password
> 구매자 : honggd / qwer1234 <br>
> 판매자 : jhpark / qwer1234 <br>
> 관리자 : admin / qwer1234 <br>

--- 

### 개발 환경
**O/S :** Windows 10<br>
**Server :** Apache Tomcat 8.5<br>
**DB :** Oracle<br>
**Language :** Java, SQL, HTML5, CSS3, JavaScript<br>
**IDE :** Eclipse, SQL Developer

---

### 핵심 기능 (작성자가 구현한 기능만 코드 설명)
<details><summary>클래스 게시판의 다양한 조회</summary><div markdown="1">

> 

</div>
</details>

<details><summary>캘린더를 통한 스케줄 조회 및 관리</summary><div markdown="1">
	
> 

</div>
</details>

<details><summary>클래스 종료 후 강사에 대한 리뷰 및 평점 작성</summary><div markdown="1">

> 
</div>
</details>

<details><summary>포인트 시스템을 통한 학생의 충전 및 결제와 튜터의 환급 기능</summary><div markdown="1">
> 
</div>
</details>

<details><summary>튜터의 클래스 횟수와 평점에 따른 네가지 등급 자동 변경 기능</summary><div markdown="1">
> 
</div>
</details>

<details><summary>ERP게시판을 통한 지점간의 상품 교환 기능</summary><div markdown="1">
</div>
</details>

<details><summary>유저의 각종 문의 및 건의, 관리자의 공지사항 및 답변을 위한 고객센터</summary><div markdown="1">
</div>
</details>

<details><summary>포인트 시스템을 통한 학생의 충전 및 결제와 튜터의 환급 기능 </summary><div markdown="1">
</div>
</details>
 
<details><summary>이력서 검증을 통한 튜터 권한 부여 기능 </summary><div markdown="1">
</div>
</details>
---

### 주요 테이블  
  
+ ALL_USER ( 회원 관리용 테이블 )
  + 시스템에 가입되어있는 회원의 정보를 저장하는 테이블
  + USER_ID COLUMN 을 Primary key로 사용
  + RESUME_YNP COLUMN 의 상태값을 사용하여 일반회원, 튜터회원의 상태를 구분
  
+ TUTOR_RESUME ( 튜터 이력서 관리 테이블)
  + 제출 이력서의 정보를 저장하는 테이블
  + 일반 회원에서 튜터 회원으로 전환을 원할 경우, 이력서를 작성하면 해당 테이블에 정보값이 입력
  + TUTOR_ID 값을 ALL_USER테이블에서 참조(Foreign key)하여 Primary key로 사용
  
+ POINT_LOG ( 포인트 입출력 기록 테이블 )
  + 시스템 내에서 사용될 포인트를 충전 및 환전 시, 그 정보가 기록되는 테이블
  + 해당 테이블에 정보가 기록되면 ALL_USER의 포인트 총량 컬럼에 update가 되는 트리거가 대입되어있음
  + POINT_LOG의 고유 시퀀스값인 LOG_NO를 Primary key로 사용
  
+ CS_BOARD ( 고객센터 게시판 관리 테이블 )
  + 고객센터 작성 글의 내용을 저장하는 테이블
  + 비밀글, 답변여부를 해당 컬럼으로 확인 가능
  + CS_BOARD의 고유 시퀀스값인 CS_NO를 Primary Key로 사용
  
+ CLASS ( 등록된 수업 관리 테이블 )
  + 수업(클래스) 게시판에 등록된 수업의 정보를 저장하는 테이블
  + 해당 테이블에서 입력받은 수강 종료 날짜, 지원 종료 날짜는 Oracle schduler를 통해 종료 여부가 체크됨
  + CLASS의 고유 시퀀스값인 CLASS_NO를 Primary Key로 사용

  
--- 

