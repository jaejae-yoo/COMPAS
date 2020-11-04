# COMPAS 2020 경진과제
## 주제: (광양시) 전기자동차 충전소 최적입지 선정
## 주최 & Data Source
* LH 한국토지주택공사 https://compas.lh.or.kr/

## 분석 흐름도 
- ### 2.2 데이터 전처리
      ✔️2.1.1 <05.광양시_대중집합시설_야영장.csv> ➡️ <광양시_대중집합시설_야영장_전처리.csv>
            1) (lon, lat) -> geometry컬럼 추가
            2) '14.광양시_소유지정보.geojson'데이터 바탕으로 야영지 면적 컬럼 생성 (컬럼명 : DGM_AR)
            3) cateogory 컬럼 추가<br/>
            4) 컬럼명 변경 (소유주체(공공/민간) --> 소유주체)
            5) 주차장법 시행령: 부설주차장의 설치대상 시설물 종류 및 설치기준에 따라 계산 --> 주차면수 컬럼 추가 

      ✔️2.2.2 <02.광양시_주차장_공간정보.csv> ➡️  <광양시_주차장_공간정보_전처리.csv>
            1) (lon, lat) -> geometry컬럼 추가
            2) 컬럼명 변경 (면적 --> DGM_AR)
            3) cateogory 컬럼 추가
            4) 소유주체 컬럼 추가

      ✔️2.2.3 <26.광양시_도시계획(교통시설).geojson> ➡️  광양시_도시계획(교통시설)_전처리.csv
            1) '주차'가 포함된 데이터만 추출
            2) 면적이 0인 데이터 제외
            3) geometry 컬럼을 활용해 lon, lat로 만들고, geometry의 Multipoligon형태를 Point로 변경
            4) category 컬럼 추가
            5) 소유주체 컬럼 추가
            6) 주차면수계산 --> 주차면수 컬럼 추가

      ✔️.2.4 <15.광양시_건물정보.geojson> ➡️ <세부용도명>을 기준으로 아래에 해당하는 데이터를 추린 후 각각 전처리하여 통합
            <전라남도 광양시 공동주택 현황_20200929.csv> 데이터를 활용하여 각 주택에 대한 세대수를 통해 주차면수를 계산-><항목1~4.csv>
            항목 1)아파트, 다가구주택, 다세대주택, 공동주택, 다중주택
            항목 2)기타공공시설, 동사무소, 공공시설, 공공업무시설
            항목 3)대학교, 대학, 쇼핑센터, 관광호텔, 호텔, 주차장
            항목 4)기타문화및집회시설, 도서관, 종합병원,휴게소

- ### 2.3 학습테이블 만들기 위한 함수 생성
      ✔️2.3.1) 완속충전기 설치가능 여부를 계산하기 위한 전처리
            1) '01.광양시_충전기설치현황.csv'에서 완속충전기_설치가능여부 컬럼을 생성하여 완속 충전기일 경우 1, 급속 충전기일 경우 0 삽입
            2) '06.광양시_전기차보급현황(연도별,읍면동별).csv'를 활용하여 각 행정구역별 3년간 신규 전기차 등록대수를 계산하여 supply 컬럼 추가
            (환경부 2020년 전기자동차 보급 및 충전인프라 구축사업 충전인프라 설치 운영지침에 따르면, 해당 장소에 설치된 충전기가 없는 경우 해당 장소의 주차단위구획 수× (0.01 + 해당 시‧도의 최근 3년간 신규 차량등록대수 분의 신규 전기차 등록대수)까지 설치가 가능하다는 지침을 활용하기 위함)

      ✔️2.3.2) 인구분포 & 자동차 등록현황
            1) '08.광양시_격자별인구현황(100X100).geojson'와 '03.광양시_자동차등록현황_격자(100X100).geojson'에서 NAN, O값 제거

      ✔️2.3.3) 학습테이블을 리턴하는 함수 생성
            1) 인구분포계산 & 인구분포컬럼 추가
            2) 등록되어있는 자동차분포 계산 & 자동차등록현황 컬럼 추가
            3) 소유주체구분 --> 0 : 개인, 1 : 개인아님, 2 : None & 법인
            4) 위에서 계산한 전기차증가량 컬럼 추가

      ✔️2.2 4) 리턴된 테이블 표준화
            1) 인구분포, 자동차등록현황, 전기차증가량 표준화

- ### 2.4 매핑
        1) 각 행정구역별 3년간 증가한 전기차 수 choropleth 표현
        2) 등록되어있는 자동차 현황 heatmap 표현
        3) 인구분포 heatmap 표현
<div>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img width=280 src="https://user-images.githubusercontent.com/61091307/98155278-51af0480-1f19-11eb-80bf-8e5c66d54a59.PNG">
<img width=300 src="https://user-images.githubusercontent.com/61091307/98155374-7b682b80-1f19-11eb-866d-8ba76ae0ea7c.PNG">
<img width=300 src="https://user-images.githubusercontent.com/61091307/98155430-920e8280-1f19-11eb-8f9a-552f35387804.PNG">
</div> 
&nbsp;&nbsp;&nbsp;&nbsp;➡️(사진 순서대로) 3년간 전기차 수 증가량, 자동차 현황, 인구분포가 광양읍, 중마동에 집중되어 있음을 확인       
        
- ### 2.5 K-Means clustering를 이용한 군집화 & 최종 위치 선정
       ✔️2.5.1)전처리한 테이블을 급속 / 완속으로 나눔
            1) 소유주체 --> 0 : 개인, 1 : 개인아님, 2 : None&법인
                  소유주체의 2에 해당하는 None과 법인은 소유주체를 판단하기 어려워서 개인(0)과 함께 완속충전기만 설치하기로 함.
                  완속충전기 --> 소유주체 : 0,1,2 (소유주체 구분없이 모두)
                  급속충전기 --> 소유주체 : 1(급속충전기는 사유지에 설치할 수 없기 때문에 소유자가 개인이 아닌 경우에만 설치함)
            2) 앞에서 전처리한 데이터를 통합한 테이블에서 '23.광양시_개발행위제한구역.geojson'과 '28.광양시_도시계획(환경기초시설).geojson'에 포함되는 위치는 모두 제거
 
       🔥2.5.2) K-means clustering 이용하여 충전기 설치 할 위치들 군집화 후 위치선정🔥
            1) 각 데이터들 상대적으로 인구분포, 자동차등록현황, 전기차증가량을 고려한 인구분포_자동차등록_전기차 컬럼 추가
            2) kmeans군집화에 '인구분포','자동차등록현황','전기차증가량' 세 개의 컬럼을 넣고 20개의 군집으로 나타내기 위해 n_clusters를 20으로 설정 -> 완속충전기, 급속충전기 각각 진행

- ### 🔸 급속충전기 빨간색으로 표현 / 🔹 완속충전기 파란색으로 표현  
<div>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img width=500 src="https://user-images.githubusercontent.com/61091307/98154673-79519d00-1f18-11eb-8417-25bb2730b55f.PNG">
</div>
