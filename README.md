# bistro

[ResDAO]

package res;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;

import notice.NoticeDTO;
import utility.DBClose;
import utility.DBOpen;

public class ResDAO {
	private DBOpen dbopen=null;
    private Connection con=null;
    private PreparedStatement pstmt=null;
    private ResultSet rs=null;
    private StringBuilder sql=null;
    
    public ResDAO() {
    	dbopen=new DBOpen();
    }//end
    
    public ResDTO res(String m_no) {
    	ResDTO dto=null;
        try {
            con=dbopen.getConnection();
            
            sql=new StringBuilder();
            sql.append(" SELECT m_name, m_price ");
            sql.append(" FROM wr_menu ");
            sql.append(" WHERE m_no= ? ");   
            
            pstmt=con.prepareStatement(sql.toString());
            pstmt.setString(1, m_no);
            rs=pstmt.executeQuery();
            if(rs.next()) {
                dto= new ResDTO();
                dto.setM_price(rs.getInt("m_price"));
                dto.setM_name(rs.getString("m_name"));
            }//end
            
        }catch (Exception e) {
            System.out.println("전체목록실패:"+e);
        }finally {
            DBClose.close(con, pstmt, rs);
        }//end
        return dto;
    	
    }//end
    
    
    
    
   
    
    //비즈니스 로직 구현
    //insert문
    public int insert(ResReadDTO dto) {
        int cnt=0;
        try {
            con=dbopen.getConnection();
            
            sql=new StringBuilder();
            sql.append(" INSERT INTO wr_res(r_no, j_id, r_people, r_date, r_time, m_no) ");
            sql.append(" VALUES (nextval(wr_res_seq), ?, ?, ?, ?, ?) ");

            pstmt=con.prepareStatement(sql.toString());
            pstmt.setString(1, dto.getJ_id());
            pstmt.setInt(2, dto.getR_people());
            pstmt.setString(3, dto.getR_date());
            pstmt.setString(4, dto.getR_time());
            pstmt.setString(5, dto.getM_no());
            
            cnt=pstmt.executeUpdate();
            
        }catch (Exception e) {
            System.out.println("추가실패:"+e);
        }finally {
            DBClose.close(con, pstmt);
        }//end
        return cnt;
    }//insert() end
    
    
    /*
    //조회하기 (?)
    public ArrayList<ResDTO> list(){
        ArrayList<ResDTO> list=null;
        try {
            con=dbopen.getConnection();
            
            sql=new StringBuilder();
            sql.append(" SELECT r_no, j_id, r_please, r_people, r_method, r_con, r_date, r_time, m_no ");
            sql.append(" FROM wr_res ");

            
            pstmt=con.prepareStatement(sql.toString());
            rs=pstmt.executeQuery();
            if(rs.next()) {
                list=new ArrayList<ResDTO>();
                do {
                    ResDTO dto=new ResDTO(); //한줄담기
                    dto.setR_no(rs.getString("r_no"));
                    dto.setJ_id(rs.getString("j_id"));
                    dto.setR_please(rs.getString("j_please"));
                    dto.setR_people(rs.getInt("r_people"));
                    dto.setR_method(rs.getString("r_method"));
                    dto.setR_con(rs.getString("r_con"));
                    dto.setR_date(rs.getString("r_date"));
                    dto.setR_time(rs.getString("r_time"));
                    dto.setM_no(rs.getString("m_no"));
                    list.add(dto); //list에 모으기
                }while(rs.next());
            }//end
            
        }catch (Exception e) {
            System.out.println("전체목록실패:"+e);
        }finally {
            DBClose.close(con, pstmt, rs);
        }//end
        return list;
    }//list() end
    s*/
    
    public ResReadDTO read(String j_id) {
    	ResReadDTO dto= null;
        try {
          con = dbopen.getConnection();
          sql = new StringBuilder();
          sql.append(" select m_name, m_price, r_date, r_people, r_time ");
          sql.append(" from wr_res WR join wr_menu WM ");
          sql.append(" on WR.m_no=WM.m_no ");
          sql.append(" where j_id = ? "); 
          
          pstmt = con.prepareStatement(sql.toString());
          pstmt.setString(1, j_id); //...
          rs = pstmt.executeQuery();
          if(rs.next()) {
            dto = new ResReadDTO();
            dto.setM_name(rs.getString("m_name"));
            dto.setM_price(rs.getString("m_price"));
            dto.setR_date(rs.getString("r_date"));
            dto.setR_people(rs.getInt("r_people"));
            dto.setR_time(rs.getString("r_time"));
            
          }//if end

        } catch (Exception e) {
            System.out.println("상세보기실패"+e);
        } finally {
            DBClose.close(con, pstmt, rs);
        }//end
        return dto;
    }//read() end
    
    
} //class end

-------------------------------------------------------------------------------------

package kr.co.bistrowarak;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;

import notice.NoticeDTO;
import res.ResDAO;
import res.ResDTO;
import res.ResReadDTO;

@Controller
public class ResController {
	private ResDAO dao = null;
	
	  public ResController() {
		  dao = new ResDAO();
	      System.out.println("----------ResController() 객체 생성됨");
	   }

//--------------------------------------------------------------------------------	
	 [ResController]
   
	  @RequestMapping(value = "reserve/res.do", method = RequestMethod.GET)
	  public ModelAndView wr_res(String m_no) {
		  ModelAndView mav=new ModelAndView();
		  mav.setViewName("reserve/wr_res");
		  mav.addObject("dto", dao.res(m_no));
				  
		  
	     return mav;
	  }
	  
	  /*
	  //res.do 페이지에 insert?
	  @RequestMapping(value = "reserve/res.do", method = RequestMethod.GET)
	  public ModelAndView insert(@ModelAttribute ResDTO dto) {
		  ModelAndView mav=new ModelAndView();
			mav.setViewName("reserve/wr_res");
			
			//예약테이블에 추가하기 (시간,날짜,인원수 등)
			int cnt=dao.insert(dto); 
			if(cnt==0){
				String messub="예약에 실패했습니다.";
				String mesbu1="<p><a href='javascript:history.back()'>[다시시도]</a></p>";
				
				mav.addObject("messub", messub);
				mav.addObject("mesbu1", mesbu1);
				
			}else{
				String messub="<script>    alert('예약을 완료했습니다.');  ";
				String mesbu1="    location.href='메인페이지 or 주문내역페이지'   ";
				String mesbu2="</script>";
				
				mav.addObject("messub", messub);
				mav.addObject("mesbu1", mesbu1);
				mav.addObject("mesbu2", mesbu2);
			}//if end
			
			return mav;
	  }//insert end
	  */
	  
	  @RequestMapping("reserve/resread.do")
	    public ModelAndView read(HttpServletRequest req, HttpSession session) {
	        ResReadDTO dto = new ResReadDTO();
	        dto.setJ_id((String)session.getAttribute("s_id"));
	        dto.setM_no(req.getParameter("m_no"));
	        dto.setR_people(Integer.parseInt(req.getParameter("r_people")));
	        dto.setR_date(req.getParameter("r_date"));
	        dto.setR_time(req.getParameter("r_time"));
	        
	        ModelAndView mav = new ModelAndView();
	        int cnt=dao.insert(dto); 
			if(cnt==0){
				String messub="예약에 실패했습니다.";
				String mesbu1="<p><a href='javascript:history.back()'>[다시시도]</a></p>";
				
				mav.addObject("messub", messub);
				mav.addObject("mesbu1", mesbu1);
				
			}else{
				mav.setViewName("reserve/resRead");
				mav.addObject("dto", dao.read(dto.getJ_id()));
			}//if end
	        return mav;
	  }
	  
	  
	  
}//class end

//--------------------------------------------------------------------------------	

[Res.jsp]

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<%@ include file="../wr_header.jsp" %>

<html lang='ko'>
    <head>    
        <meta charset="UTF-8">                          
        <link rel="stylesheet" href="//code.jquery.com/ui/1.13.0/themes/base/jquery-ui.css">
        <link rel="stylesheet" href="/resources/demos/style.css">
        <link rel="stylesheet" href="../css/wr_res.css">
        <script src="https://code.jquery.com/jquery-3.6.0.js"></script>
        <script src="https://code.jquery.com/ui/1.13.0/jquery-ui.js"></script>
        <script>
        $( function() {
          $( "#datepicker" ).datepicker();
        } );
        </script>
       
    </head>

<div id="tour" class="bg-1">
    <div class="container" >
       
        <div class="row">
        
        <!-- 본문 시작 -->
        
     <form action="resread.do"> 
        	<div class="res" style="border:3px solid #414141;   width: 600px;  overflow:auto; margin: 0 auto;">
        	
        	<div class="res_header" style="color: white ">BistroWarak 예약하기</div>
        
          <div class="position" style="margin: 90px; ">
           
                
			<input type="hidden" name="m_no" value="${dto.m_no}">
                <div class="res11" style="font-size: 12px;  position: relative; ">
                    <p>메뉴이름: ${dto.m_name}</p>
                    <p>가격: ${dto.m_price}</p>

                </div>
        

        
            <div style="width:100%; border-bottom:1px solid grey; margin-bottom:30px;"> </div>
                
                

                
        
    
<!-- 인원수 -->
<div class="inwon" style=" 
				width: 100%;
                height: 30px;
                padding-top: 30px;
                padding-left: 40px;
                padding-bottom: 50px;
                margin: 0 auto;
                margin-top: 30px;
				border-top: 1px solid #ccc;
                border-bottom: 1px solid #ccc; 
                background-color: black;">
    <div class="people" style="height:650px;">
        인원수 
        <p class="num">
        <input type="number"  class="-+"  name="r_people">
        </p>

    </div>
</div>



<!--날짜 선택하기 https://jqueryui.com/datepicker/ -->
<div class="date" style="
				width: 100%;
                height: 30px;
                padding-top: 30px;
                padding-left: 40px;
                padding-bottom: 50px;
                margin: 0 auto;
                margin-top: 30px ;
                text-align: left;
                border-top: 1px solid #ccc;
                border-bottom: 1px solid #ccc;
                background-color: black;">
    날짜 선택
    <p class="calender">Date: <input type="text" id="datepicker" name="r_date"></p> 

</div>



<!--시간 선택하기-->
<div class="time" style="width: 100%;
                height: 30px;
                margin: 0 auto;
                margin-top: 50px ;
                text-align: center;
                border-top: 1px solid #ccc;
                border-bottom: 1px solid #ccc;
                background-color: black;">
    시간 선택

    <p class="AM" style=" width: 100%;
                height: 20px;
                font-size: 12px;
                text-align: center;
                margin: 0 auto;
                background-color: seashell;">
        오전
        <div class="AM_1">
         
            <input type="radio" name="r_time" value="10:30" style="margin: 10px;"> 10:30
            <input type="radio" name="r_time" value="11:00" style="margin: 10px;"> 11:00
            <input type="radio" name="r_time" value="11:30" style="margin: 10px;"> 11:30
        </div>
    </p>

    <p class="PM" style="width: 100%;
                height: 30px;
                font-size: 12px;
                text-align: center;
                margin: 0 auto; 
                background-color: seashell;
                border-top: 1px solid #ccc;">
        오후
        <div class="PM_1">
            <input type="radio" name="r_time" value="12:00" style="margin: 10px;"> 12:00
            <input type="radio" name="r_time" value="12:30" style="margin: 10px;"> 12:30
            <input type="radio" name="r_time" value="13:00" style="margin: 10px;"> 13:00
            <input type="radio" name="r_time" value="13:30" style="margin: 10px;"> 13:30
            <input type="radio" name="r_time" value="17:30" style="margin: 10px;"> 17:30
            <input type="radio" name="r_time" value="18:00" style="margin: 10px;"> 18:00
            <input type="radio" name="r_time" value="18:30" style="margin: 10px;"> 18:30
            <input type="radio" name="r_time" value="19:00" style="margin: 10px;"> 19:00
            <input type="radio" name="r_time" value="19:30" style="margin: 10px;"> 19:30
            <input type="radio" name="r_time" value="20:00" style="margin: 10px;"> 20:00
            <input type="radio" name="r_time" value="20:30" style="margin: 10px;"> 20:30

        </div>
    </p>
    

</div>

<div style="width:100% ;height:50px; "></div>

<!--예약하기 버튼-->
<div style=" " >
    <input type="submit" value="예약하기" style=" width: 100%; height:50px; background-color: #feed8d; color: black; position:relative; top:270px; ">
</div>
</form> 

</div> <!--res 닫기--> <!-- 본문 끝 -->
        	    
   </div>  
        </div>
    </div>
</div>

<%@ include file="../wr_footer.jsp" %>
