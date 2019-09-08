# text_analysis

# 음영 지역(지하 주차장) 측위 개선

## Repository Contents
* Quick Summary
* Data
* Steps for Analytics
* Codes instructions

### Quick Summary 
#### **음영지역 측위 개선 필요**
[T Map](https://www.tmap.co.kr/tmap/about_service/tmap_info.do)에서 제공되고 있는 위치 기반 Navigation Service 는 GPS Signal Outage가 발생하는 음영 지역(지하 주차장, 터널 등) 에서는 제한적이며, 효율적인 정보 제공이 불가하다. GPS 외 WiFi Mac Address, RSSI (Received Signal Strength Index), SSID등의 주변 전파 환경, 그리고 기압, 조도 등의 센서값들을 활용하여 음영지역내 위치 정보를 제공하는 방법을 고안하고 이를 실적용하여 위치 기반 서비스 품질을 제고한다.

주변 전파환경 정보를 생성하고 이를 기반으로 서비스 제공시, 단말에서 스캔된 WiFi, RSSI등과 유사도 계산하여 가장 가까운 전파환경 정보를 찾고, 이에 매핑된 GPS를 제공함

### Data
1. **출차 로그 :**  음영지역에서 출구인접 지역을 거쳐서 출구로 나아가서 최초 GPS 수신 후 10초까지 수집
2. **입차 로그 :** 입구 진입 200미터 전 부터 지하주차장 입구를 거쳐 음영지역 주차시까지 로그 수집
3. **수집 시작 Event** 
   출차 - 음영 지역에서 T Map 실행 후 목적지 입력
   입차 - Near Destination 호출(T Map 제공) 이후 데이터 수집 시작
4. **수집 종료 Event**
  출차 - 최초 GPS 관측 후 10개 관측시 종료 혹은 수집시작 이후 5분 경과시 종료
  입차
			  Near Destination 호출 이후 5분 경과시까지 At Destination 호출되지 않으면 종료
			  At Destination 호출 이후 30초 경과시까지 음영지역 진입하지 않으면 종료
			  음영지역 진입 후 run, walk 등의 이벤트 호출
	  
![undp](https://lh3.googleusercontent.com/XjUReQIWT2F1TwBoA_vqGkAzr1Mny6p8hp0Fxe43rXsU_u9lA4KFFP3UU5rNxUuGLESsh3XhmwZh)

### Steps for Analytics
* **Step 1 Data Preparation**
	- Two variables, *illuminance*, *pressure*, are used to detect the border area between indoor and outdoor.
	 - Both *MAC AP* and *RSSI* (the unit of signal strength - the higher the stronger) are significantly used in calculating the degree of similarity.
	 - Every invalid *SSID* and its corresponding *MAC AP* and *RSSI* should be eliminated in the step of data compilation.
	 - The *MAC AP*s which values fall on "NaN" should be removed, too.
* **Step 2 Out-episodes Analyses**
	- Make groups of logs, **the starting positions of which would be the same** 
	 - Make groups of logs which **ending position would be the same** by the groups in the above.
	That is, let's classify logs, the starting positions of which would be the same, into the ending position.
* **Step 3 How to make clusters within one log**
	* The sequences of WiFi Mac Addresses in one log (or one episode) are clustered (or grouped) into a few ones based on the degree of similarity. The method to make clusters (or groups) are designed in a way that all the pairwise similarities between two groups in one log are less than 0.2
* **Step 4 Compare clusters <u>between logs</u> and compile information for the path from a underground parking lot where GPS outages through a outdoor area.** 
* **Step 5 In-episodes Analyses**
	- Link in-episodes to out ones using the underground WiFi environments. In-episodes may have more accurate POI and GPS information and these could be merged to the pre-built information in step 4.
* **Step 6 Select GPS from Out-episodes**

### Data & Codes Instructions
* **음영정보 생성**
	 * **makeInfo_20190903.py** -> hiveDF/outepi_hive.csv를 입력으로 음영 지역 정보 및 대표 GPS 계산 
	 * **makeInfo_20190826.ipynb** -> hiveDF/outepi_hive.csv를 입력으로 음영 지역 정보 및 대표 GPS 계산 
	 * **main_20190821_sampleLog77.ipynb** -> samplelog 내 출차 로그(현대백화점 판교점, 송파 헬리오시티, 수내동 3개 빌딩 지하주차장)로 부터 음영지역 생성 단계 설명 및 음영 정보 생성 Scenario Demo
	 * **outloganal_20190722.py** -> 음영정보 생성시 필요한 클래스 및 사용자 함수 정의
	 * **FinalInfo_20190903.csv** -> hiveDF/outepi_hive.csv (7개 지하 주차장 338개 Session Keys)를 기반으로 생성된 음영지역 정보
	 
 * **유사도 평가**
	 * sampleInfo/simEval_20190902.py -> 단말에서 수행될 유사도 평가 프로그램 (WiFi, RSSI를 입력 받은 후 Jaccard, Sorenson-Dice, Modified Sorenson Index 계산하고 대표 GPS 제공)
	 * sampleInfo/devMac00 -> 입력 샘플 - 현대백화점 판교점
	 * sampleInfo/devMac01 -> 입력 샘플 - 롯데백화점 분당점
	* sampleInfo/devMac02 -> 입력 샘플 - 송파 헬리오시티
	 * sampleInfo/devMac03 -> 입력 샘플 - SKT 타워
	 * sampleInfo/devMac04 -> 입력 샘플 - 타임스퀘어(B Gate)
	 * sampleInfo/devMac05 -> 입력 샘플 - SKT 분당사옥
	 * sampleInfo/devMac06 -> 입력 샘플 - 셀리지온
 
 * **Data**
	 *	hiveDF/outepi_hive.csv  -> 7개 지하 주차장에서 수집된 출차 로그 (338개 session keys)
	 *	hiveDF/inepi_hive.csv -> 7개 지하 주차장에서 수집된 입차 로그
	* samplelog/out_epi_0121.csv -> 현대 백화점 판교점 지하 주차장에서 수집된 출차 로그
	 * samplelog/out_epi_0412.csv -> 송파 헬리오시티 지하 주차장에서 수집된 출차 로그
	 * samplelog/out_epi_0502.csv -> 분당사옥 인근 3개 빌딩 지하 주차장에서 수집된 출차로그
	 * samplelog/in_epi_0121.csv -> 현대 백화점 판교점 지하 주차장에서 수집된 입차 로그
	 * samplelog/in_epi_0412.csv -> 송파 헬리오 시티 지하 주차장에서 수집된 입차 로그
	 * samplelog/in_epi_0502.csv -> 분당사옥 인근 3개 빌딩 지하 주차장에서 수집된 입차로그

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEyOTI4MDUxMV19
-->