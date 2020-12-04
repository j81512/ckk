----------------------
### 프로젝트명 : CKK
##### [Site 바로가기](park.jh92.kro.kr/CKK)
##### [Project 상세정보](https://www.notion.so/CKK-Cooking-King-91c88bfdf1ee45a7bf71995041b7cc42)
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

> 여러 조건을 통해 다양한 검색이 가능하다.
```java
	// [Controller]
	//어떤 조건들이 있는지 확인 후,
	//map을 이용하여 조건들을 service->dao로 보내준다.
	//조건들을 포함한 url을 가진 pagebar를 생성해준다.
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

		int numPerPage = 12;
		int cPage = 1;
		
		try {
			cPage = Integer.parseInt(request.getParameter("cPage"));
		} catch (NumberFormatException e) {
			
		}
		
		String lowPriceS = request.getParameter("lowPrice");
		String highPriceS = request.getParameter("highPrice");
		String[] categoryArr = request.getParameterValues("category");
		String[] locationArr = request.getParameterValues("location");
		String keywordType = request.getParameter("keywordType");
		String keyword = request.getParameter("keyword");
		int lowPrice = lowPriceS != null && !lowPriceS.isEmpty() ? Integer.parseInt(lowPriceS) : 0;
		int highPrice = highPriceS != null && !highPriceS.isEmpty() ? Integer.parseInt(highPriceS) : 0;
		String classDateS = request.getParameter("classDate");
		Date classDate = classDateS != null && !"".equals(classDateS) ? Date.valueOf(classDateS) : null;
		Map <String, Object> param = new HashMap<>();
		param.put("categoryArr", categoryArr);
		param.put("locationArr", locationArr);
		param.put("keywordType", keywordType);
		param.put("keyword", keyword);
		param.put("lowPrice", lowPrice);
		param.put("highPrice", highPrice);
		param.put("classDate", classDate);
		
		Map<String, Object> map = new BoardService().getBoardList(numPerPage, cPage, param);
		List<Board> list = (List<Board>)map.get("list");
		int totalContents = (Integer)map.get("totalContents");
		System.out.println("list@servlet = " + list);
		System.out.println("totalContent@servlet = " + totalContents);
		
		
		String url = request.getRequestURI() + "?";
		
		if(categoryArr != null && categoryArr.length != 0) {
			for(int i = 0; i < categoryArr.length; i++){
				url += "category=" + categoryArr[i] + "&";
			}
		}
		if(locationArr != null && locationArr.length != 0) {
			for(int i = 0; i < locationArr.length; i++) {
				url += "location=" + locationArr[i] + "&";
			}
		}
		if(keyword != null && !"".equals(keyword)) {
			url += "&" + keywordType + "=" + keyword + "&";
		}
		if(lowPrice != 0) {
			url += "&lowPrice=" + lowPrice;
		}
		if(highPrice != 0) {
			url += "&highPrice=" + highPrice;
		}
		
		if(classDate != null) {
			url += "&classDate=" + classDate;
		}
		String pageBar = Utils.getPageBarHTML(cPage, numPerPage, totalContents, url);
		
		request.setAttribute("list", list);
		request.setAttribute("pageBar", pageBar);
		request.getRequestDispatcher("/WEB-INF/views/board/boardList.jsp").forward(request, response);
		
	}
```
```java
	// [DAO]
	//Controller에서 map을 통해 넘어온 조건들을 확인하여,
	//선택된 조건만을 이용하여 query를 작성 후 검색한다.
	public Map<String, Object> getBoardList(Connection conn, int numPerPage, int cPage, Map<String, Object> param) {

		Map<String, Object> map = new HashMap<>();
		List<Board> list = new ArrayList<>();
		PreparedStatement pstmt = null;
		ResultSet rset = null;
		String sql = prop.getProperty("getBoardList1");
		String[] categoryArr = (String[])param.get("categoryArr");
		String[] locationArr = (String[])param.get("locationArr");
		String keywordType = (String)param.get("keywordType");
		String keyword = (String)param.get("keyword");	
		int lowPrice = (Integer)param.get("lowPrice");
		int highPrice = (Integer)param.get("highPrice");
		Date classDate = (Date)param.get("classDate");
		
		if(categoryArr != null && categoryArr.length != 0) {
			sql += " and category in(";
			for(int i = 0; i < categoryArr.length; i++) {
				if(i != 0) sql += ",";
				sql += "'" + categoryArr[i] + "'";
			}
			sql += ")";
		}
		
		if(locationArr != null && locationArr.length != 0) {
			sql += " and class_location in(";
			for(int i = 0; i < locationArr.length; i++) {
				if(i != 0) sql += ",";
				sql += "'" + locationArr[i] + "'";
			}
			sql += ")";
		}
		
		if(keyword != null && keyword != "") {
			sql += " and " + keywordType + " like '%" + keyword + "%'";
		}
		
		if(lowPrice != 0 && highPrice != 0) {
			sql += " and price between " + lowPrice + " and " + highPrice;
		}
		else if(lowPrice != 0 && highPrice == 0) {
			sql += " and price > " + lowPrice;
		}
		else if(lowPrice == 0 && highPrice != 0) {
			sql += " and price < " + highPrice;
		}
		
		if(classDate != null) {
			sql += " and class_date = to_date('"+ classDate +"','yyyy/MM/dd')";
		}
		
		sql += prop.getProperty("getBoardList2");
		
		System.out.println(sql);
		
		Board b = null;
		int totalContents = 0;
		
		int startRnum = (cPage-1)*numPerPage+1;
		int endRnum = cPage*numPerPage;
		
		try {
			pstmt = conn.prepareStatement(sql);
			pstmt.setInt(1, startRnum);
			pstmt.setInt(2, endRnum);
			rset = pstmt.executeQuery();
			while(rset.next()) {
				b = new Board();
				b.setApplyExpireYn(rset.getString("apply_expire_yn"));
				b.setCapacity(rset.getInt("capacity"));
				b.setClassAddress(rset.getString("class_address"));
				b.setClassContent(rset.getString("class_content"));
				b.setClassDate(rset.getDate("class_date"));
				b.setClassEndYn(rset.getString("class_end_yn"));
				b.setClassLocation(rset.getString("class_location"));
				b.setClassNo(rset.getInt("class_no"));
				b.setEndTime(rset.getInt("end_time"));
				b.setLastApplyDate(rset.getDate("last_apply_date"));
				b.setPrice(rset.getInt("price"));
				b.setStartTime(rset.getInt("start_time"));
				b.setTitle(rset.getString("title"));
				b.setTutorId(rset.getString("tutor_id"));
				b.setClassPic1Org(rset.getString("class_pic1_org"));
				b.setClassPic2Org(rset.getString("class_pic2_org"));
				b.setClassPic3Org(rset.getString("class_pic3_org"));
				b.setClassPic1Ren(rset.getString("class_pic1_ren"));
				b.setClassPic2Ren(rset.getString("class_pic2_ren"));
				b.setClassPic3Ren(rset.getString("class_pic3_ren"));
				b.setTutorName(rset.getString("tutor_name"));
				b.setCategory(rset.getString("category"));
				totalContents = rset.getInt("cnt");
				
				list.add(b);
			}
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			close(rset);
			close(pstmt);
		}
		
		map.put("list", list);
		map.put("totalContents", totalContents);
		
		return map;
		
	}
```
</div>
</details>

<details><summary>클래스 종료 후 강사에 대한 리뷰 및 평점 작성</summary><div markdown="1">
> sql의 스케쥴러를 통해 하루 한번씩 클래스 종료 여부를 확인하는 프로시져를 실행하고,
> 종료된 클래스의 종료 여부 컬럼을 변경, 클래스를 수강한 학생에게 리뷰작성 메시지를 전송한다.
```sql
--수업 종료 여부 확인 프로시져
CREATE OR REPLACE PROCEDURE PROC_CLASS_END_CK 
IS
    V_CNO NUMBER;
    V_EDATE DATE;
    V_CNT NUMBER;
    v_applied number;
    v_user_id all_user.user_id%type;
    v_title class.title%type;
BEGIN
    SELECT COUNT(*) INTO V_CNT FROM CLASS;
    
    FOR i IN 1 .. V_CNT LOOP
        SELECT CLASS_NO, CLASS_DATE, title INTO V_CNO, V_EDATE, v_title
        FROM (
            SELECT CLASS_NO, CLASS_DATE, title, ROWNUM RNUM
            FROM CLASS)
        WHERE i = RNUM;
    IF(sysdate > V_EDATE) THEN
        UPDATE CLASS SET CLASS_END_YN = 'Y' WHERE CLASS_NO = V_CNO;
        select count(*) into v_applied from class_schedule where class_no = v_cno;
        for i in 1 .. v_applied loop
            select user_id
            into v_user_id
            from (select user_id, rownum rnum from class_schedule)
            where rnum = i;
            insert into message values(seq_msg_no.nextval, v_user_id, '수업 종료 관련 메시지 입니다.', '<' || v_title || '>\n위 수업이 종료되었습니다.\마이페이지 - 리뷰 작성을 통해 리뷰를 작성해주세요.', default, default);     
        end loop;
        
    END IF;
    END LOOP;
END;
/

--프로그램 + 스케줄러 연동 : 클래스 강의 날짜 : JOB_CK_END_DAILY
BEGIN
    DBMS_SCHEDULER.CREATE_JOB(
    JOB_NAME => 'JOB_CK_CLASS_END_DAILY',
    PROGRAM_NAME => 'CLASS_END_CK_PROC',
    SCHEDULE_NAME => 'SCHEDULE_DAILY_AM_00',
    COMMENTS => '수업종료 여부 확인 프로시저 등록',
    ENABLED => TRUE
    );
END;
/

--스케줄러 생성 : SCHEDULE_DAILY_AM_00 : 매일 자정, 1번 실행되는 스케줄러
BEGIN 
    DBMS_SCHEDULER.CREATE_SCHEDULE(
    SCHEDULE_NAME => 'SCHEDULE_DAILY_AM_00',
    START_DATE => SYSDATE,
    END_DATE => NULL,
    REPEAT_INTERVAL  => 'FREQ=DAILY;INTERVAL=1;BYHOUR=00;BYMINUTE=0;BYSECOND=0;',
    COMMENTS => '매일 자정 실행'
    );
END;
/
```
</div>
</details>

<details><summary>튜터의 클래스 횟수와 평점에 따른 네가지 등급 자동 변경 기능</summary><div markdown="1">
> 
</div>
</details>

<details><summary>포인트 시스템을 통한 학생의 충전 및 결제와 튜터의 환급 기능</summary><div markdown="1">
> 
</div>
</details>

<details><summary>캘린더를 통한 스케줄 조회 및 관리</summary><div markdown="1">
	
> 

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

