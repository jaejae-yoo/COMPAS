# COMPAS 2020 경진과제
## 주제: (광양시) 전기자동차 충전소 최적입지 선정
## 주최 & Data Source
* LH 한국토지주택공사 https://compas.lh.or.kr/

## 분석 흐름도 
- ### 2.1 데이터 전처리
  **1. <05.광양시_대중집합시설_야영장.csv> :arrow_right: <광양시_대중집합시설_야영장_전처리.csv>**
       &nbsp;<br>1) lon, lat가지고 geometry컬럼 만들기<br/>
       &nbsp;2) '14.광양시_소유지정보.geojson'데이터를 바탕으로 야영지 면적을 구하기 (컬럼명 : DGM_AR)
       &nbsp;<br>3) cateogory컬럼 추가<br/>
       &nbsp;4) 컬럼명 변경 (소유주체(공공/민간) --> 소유주체)
       &nbsp;<br>5) 주차면수 컬럼 추가 --> 주차장법 시행령: 부설주차장의 설치대상 시설물 종류 및 설치기준에 따라 계산<br/>

  **2. <02.광양시_주차장_공간정보.csv> :arrow_right: <광양시_주차장_공간정보_전처리.csv>**
      1) lon, lat가지고 geometry만들기
      2) 컬럼명 변경 (면적 --> DGM_AR)
      3) cateogory 컬럼 추가
      4) 소유지 컬럼 추가

  **3. <26.광양시_도시계획(교통시설).geojson> :arrow_right: 광양시_도시계획(교통시설)_전처리.csv**
      1) '주차'관련된 데이터만 추출
      2) 면적이 0인 곳은 제외
      3) geometry를 이용해 lon, lat컬럼을 만들고, geometry의 Multipoligon형태를 Point로 변경
      4) category 컬럼 추가
      5) 소유자 컬럼 추가
      6) 주차면수계산

  **4. <15.광양시_건물정보.geojson> :arrow_right: <세부용도명>을 기준으로 아래에 해당하는 데이터를 추린 후 각각 전처리하고 합침-> <항목1~4.csv>**
      항목 1)아파트, 다가구주택, 다세대주택, 공동주택, 다중주택
      항목 2)기타공공시설, 동사무소, 공공시설, 공공업무시설
      항목 3)대학교, 대학, 쇼핑센터, 관광호텔, 호텔, 주차장
      항목 4)기타문화및집회시설, 도서관, 종합병원,휴게소




- ### 2.2 학습테이블 만들기 위한 함수 생성
  -2.2 1) 완속충전기 설치가능 여부를 계산하기 위한 전처리
  -2.2 2) 인구분포 & 자동차 등록현황
  -2.2 3) 학습테이블을 리턴하는 함수 생성
  -2.2 4) 리턴된 테이블 표준화


- ### 2.3 매핑
- ### 2.4 K-Means clustering를 이용한 군집화 & 최종 위치 선정
