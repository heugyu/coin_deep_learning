> 기본적인 로직은 기존과 같은 방향으로 일별 집계와 누적 데이터는 유지 
집계 데이터와 ML 데이터의 혼합으로 하이브리드 유사 상품 추천
로직 및 데이터의 간소화를 위해 검색 채널과 카테고리 채널 데이터만 사용
성능 개선을 목표

---

### 기존 이슈사항

- 누적 데이터의 문제점
    - flow_set 의  score의 차감으로만 누적 → 실질적으로 노이즈 데이터 삭제 로직 부재
    - 누적 flow_set의 업데이트 날짜 필터를 고려하지 않음
    - 균일하지 않은 scale 모든 각각의 score들이 균일한 0~1 사이의 스케일 값이 상대적임
- 채널의 문제점
    - 넓은 범위의 키워드 검색, 브랜드의 키워드, 샵 키워드 검색일 경우(노이즈 발생)
    - 페이즈를 여러개를 띄울때의 독립성 부재
    - page관리 룰이 모바일 및 PC가 다름
    - 상품의 잘못된 등록의 문제점
    - 카테고리가 좁은 범위의 상품군과 넓은 범위의 상품군
    - 연관 상품을 위한 포용적인 상품을 나오도록 여러 타입의 채널을 사용
    - 부족한 데이터를 채우기 위한 여러 타입의 채널을 사용
- Total Score의 스케일
    - 누적 데이터가 포함된 최종적 Score에 대한 스케일 부족

---

### 개선방안

- 로직 간소화
    - Score 반영에 크게 영향이 없는 부분 간소화

        → 유저의 성별, 나이 관련 로직 제거

        → 스페셜, 베스트셀러, 그룹바이, 타임세일, 데일리딜, 메인, 그밖 채널 제거

        → 수치화의 스케일화 0~1 사이의 값으로 

        → 스크립트의 간소화

- 데이터
    - 장바구니 및 구매
        - page_no IN (106, 423, 502, 533)  AND ref_page_no IN (50, 51, 356)
        flowpath_no IN (585, 372 ,109, 480, 372)
        - 장바구니의 경우 ref_page_no가 상품 상세페이지 이고 flowpath_no가 검색 테이터만 사용하도록 함

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a96de5ef-06c8-470e-b8fd-c9a19939cbaf/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a96de5ef-06c8-470e-b8fd-c9a19939cbaf/Untitled.png)

    - 검색 채널
        - 검색 채널

            → 585, 372 ,109, 480, 372

            → page_no IN (50, 51, 356) AND reg_page_no IN (585, 372, 109) 
            AND flowpath_no IN (585, 480, 372, 109)

            → count_banner_no의 조건도 있지만 같은 검색어로 PV가 있는 데이터에 대해 
            count_banner_no가 null인 경우가 있음 모바일 쪽 데이터 

            → ref_page_value와 flowpath_value로 조인하려함 

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f67a02da-f27a-482a-aba3-6a47fe3b1ed1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f67a02da-f27a-482a-aba3-6a47fe3b1ed1/Untitled.png)

        - 상세페이지로 flowpath_no는 고루 분포 됨

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e63dd78-30e7-4810-8ca1-ffdc7e342eb3/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e63dd78-30e7-4810-8ca1-ffdc7e342eb3/Untitled.png)

    - 카테고리 채널
        - 카테고리의 경우 고객의 행동에 따라 다르지만 유사한 상품을 본다라기 보다 큰 의미로 
        비슷한 상품을 보는 경우가 많음

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85392cf3-8d35-4188-b756-feb91eddc7bf/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85392cf3-8d35-4188-b756-feb91eddc7bf/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee3bcb2b-3c73-4c2f-9428-cdcc164b7775/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee3bcb2b-3c73-4c2f-9428-cdcc164b7775/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f5f83506-a4ef-4b94-97fa-9d91cd967d87/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f5f83506-a4ef-4b94-97fa-9d91cd967d87/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/20b20a68-fd72-4f3f-87f6-08cb9e3bed12/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/20b20a68-fd72-4f3f-87f6-08cb9e3bed12/Untitled.png)

        - 실제로 위와 같은 쌍으로 만들어지는 count는 1
        카테고리의 경우에는 1이상의 중복 쌍이 있는경우만 사용하거나 
        daily 데이터와 누적 데이터를 merge 할때 필터 해도 될듯

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e024c9b7-f31c-41de-9fa7-161e3923e927/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e024c9b7-f31c-41de-9fa7-161e3923e927/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a10822f-6384-4b15-a452-d7c79db26d3b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a10822f-6384-4b15-a452-d7c79db26d3b/Untitled.png)

            - 상품의 쌍이 1이상 만들어 지더라도 유사하지 않은 쌍이 존재 
            이것은 쌍을 만드는 FULL OTER JOIN 에서 A → B→ A→C 의 경우 같음 
            같은 대카테고리 안에서 본 상품의 경우 성격은 비슷한 상품이지만 
            유사하다고는 어려움
            - 이런 문제를 피하기 위해선 필터의 강화가 필요 중, 소 카테고리가 flowpath_value인 데이터만 사용함
    - ML 학습 데이터
        - Web Log 1년 데이터
            - 학습 모델은 word2vec을 사용하였고 검색 키워드에 대한 상품의 리스트를 sentence로 보고 
            각 상품의 코드를 word로 보아 근처에 있는 상품의 코드를 학습

                → 결국 집계와 비슷한 방법이지만 집계의 FULL OUTER 조인과는 다른 확률 모델 

                → 학습된 모델은 상품에 대한 벡터값을 리턴하여 cos로 가까운 값을 유사하다 생각

            - WebLog로 집계되는  검색 상품의 상품 결과가 국가별로 제한 
            → 검색 결과가 많다는 것은 넓은 범위의 키워드(브랜드 이름, 카테고리 이름)일 경우가 많기 때문
            - click_list, category_list, keyword_list, recom_list, bestseller_list, special_list, groupbuy_list, recomkeyword_list
            → 채널 데이터 사용, 많은 데이터로 학습에 사용하기 위해

        - ML 데이터 문제점
            - 키워드로 검색하고 PV가 있는 상품들로 리스트를 구성하기 때문에 특정 브랜드, 카테고리명, 넓은 범위의 검색어의
            경우에는 넓은 범위로 상품이 묶임 ex) 샤오미, 나이키, 화장품
            - 넓은 범위의 키워드를 배제 하다보니 인기있는 상품만 추천 대상이 됨
            - 데이터를 어떻게 구성하냐에 따라 word2vec, fasttext 사용가능 어떻게 구성..
            - 결과적으로 유사 상품이 추천되지만 인기가 낮은 상품 다수

                ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c50db3e-bd2a-44c6-9620-815271d1a98b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c50db3e-bd2a-44c6-9620-815271d1a98b/Untitled.png)

                ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/12c60ffa-3c39-4f67-bcd7-40b6b4dfb93e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/12c60ffa-3c39-4f67-bcd7-40b6b4dfb93e/Untitled.png)

        - ML 데이터 개선
            - 현재는 매일 국가별로 1년치 WebLog 데이터로 W2V 모델 학습

                → Daily 데이터만 사전 추가 후 업데이트? ↔ 결과 나빠질지 좋아질지 모름

                ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/169f3f0b-c736-449a-9b55-37c6419690ac/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/169f3f0b-c736-449a-9b55-37c6419690ac/Untitled.png)

            - 기준 - 상품 리스트 → 상품 리스트를 학습 상품 A가 B옆에 있을 확률
            - 검색 후 상품 개수 필터 제거
            - 검색어 → 브랜드 제거
            - 검색어를 기준으로 상품을 학습 ↔ 검색어 및 제목(노이즈를 제거)을 학습 후 검색어 와 유사 검색어의 상품도 함께 학습
- Score
    - 기본은 MINMAX Scale
    - ML 데이터의 경우 유사 확률이 0~1 사이값으로 scale됨, 결국 집계 데이터의 score 값이 0~1 사이값으로 scale 필요
    그러기 위해서는 채널별 count 값을 누적하며 매번 MINMAX SCALE 하여 계산
    - 가중치를 위해서는 (가중치) * score + (1-가중치) * score로 가중치를 반영한 0~1값으로 계산

---

### 개선 .ver 개발

- Row Data 구성
    1. 불필요한 채널 및 성별 나이 제거 후 필요한 데이터로만 daily_web_log 구성 
        - 전체 웹로그 중에서 상품 상세 페이지와 장바구니 웹 로그만 집계
        - tracking_session_id를 사용하여 고객별로 합침

            DROP TABLE recommend_goods.rg_v3_temp_daily_page_access_log_info;
            CREATE TABLE recommend_goods.rg_v3_temp_daily_page_access_log_info
            (
            	cust_no string,
            	ref_page_no string, 
              	ref_page_value string, 
              	flowpath_no string, 
              	flowpath_value string,
            	flowpath_type string,
            	count_banner_no string,
            	page_no string, 
            	page_value string,
            	group_code string,
            	gd_no string, 
            	gdlc_cd string, 
            	gdmc_cd string, 
            	gdsc_cd string, 
            	add_info4 string,
            	log_dt string	
            )
            PARTITIONED BY (svc_nation_cd string)
            ROW FORMAT DELIMITED
            FIELDS TERMINATED BY '\001'
            STORED AS ORC TBLPROPERTIES("orc.compress" = "SNAPPY")

            WITH daily_web_logs AS
            (
            	SELECT
            		tracking_session_id,      
            		ref_page_no, 
            		trim(ref_page_value) ref_page_value, 
            		flowpath_no, 
            		trim(flowpath_value) flowpath_value,
            		flowpath_type,
            		page_no, 
            		trim(page_value) page_value, 
            		count_banner_no,
            		group_code,
            		regexp_replace(trim(gd_no), '!', "") gd_no, 
            		gdlc_cd, 
            		gdmc_cd, 
            		gdsc_cd, 
            		add_info4,
            		log_dt,
            		CASE WHEN length(cust_no) <> 9 THEN NULL ELSE cust_no END cust_no,
            		svc_nation_cd
            	FROM web_logs.page_access_log_2day
            	WHERE
            		-- 집계 하루 전 02:00 ~ 집계 당일 02:00
            		log_dt >= CONCAT(DATE_SUB(CURRENT_DATE,1),' 02:00:00.000') 
            		AND log_dt <  CONCAT(DATE_SUB(CURRENT_DATE,0),' 02:00:00.000')
            		AND client_side_access_yn = 'Y'	
            		-- page_no는 장바구니와 상세페이지로
            		AND page_no IN ('106', '423', '502', '533', '50', '51', '356')
            		-- 회사 내부 IP 제거 
            		AND user_ip not like '%218.237.90.%'   -- NTower 유선(개발)
            		AND user_ip not like '%175.125.134.%'  -- NTower 유선(기술)
            		AND user_ip not in ('110.13.128.163') -- Ntower 무선
            		AND user_ip not like '%211.115.100.%'  -- KRIDC
            		AND user_ip not like '%172.30.%' -- B클래스 대역 (VM)
            		AND user_ip not in ('175.114.172.107') -- 웹 취약점 스캔 -> 보안팀 확인 후 별도 전달
            		AND user_ip not in ('113.43.45.114', '113.43.45.29', '118.238.237.169', '118.238.237.29') -- JP DPC
            		-- SG -- 
            		AND user_ip not in ('203.126.153.98', '203.126.153.28') -- Active 유선
            		AND user_ip not in ('203.126.153.100') -- Active 무선
            		AND user_ip not in ('124.155.210.34', '124.155.210.28') --  Backup 유선
            		AND user_ip not in ('124.155.210.35') -- Backup 무선
            		-- ID -- 
            		AND user_ip not in ('103.85.67.178', '103.85.67.29') -- Active 유선
            		AND user_ip not in ('103.85.67.179') -- Active 무선
            		AND user_ip not in ('36.37.107.58', '36.37.107.29') -- Backup 유선
            		AND user_ip not in ('36.37.107.59') -- Backup 무선
            		-- MY -- 
            		AND user_ip not in ('58.71.200.226', '58.71.200.29') -- Active 유선
            		AND user_ip not in ('58.71.200.227') -- Active 무선
            		AND user_ip not in ('211.24.109.163') -- Backup 유선
            		AND user_ip not in ('211.24.109.162', '211.24.109.29') -- Backup 무선
            		-- CN-ED --
            		AND user_ip not in ('58.247.249.130', '58.247.249.29') -- Active 유선
            		AND user_ip not in ('58.247.249.131') -- Active 무선
            		AND user_ip not in ('101.231.47.138', '101.231.47.29') -- Backup 유선
            		AND user_ip not in ('101.231.47.139') -- Backup 무선
            		-- Qxpress -- 
            		AND user_ip not in ('61.84.105.33', '61.84.105.28') -- KR 유선&무선
            		AND user_ip not in ('36.67.139.87', '36.67.139.24') -- IDDPC 유선
            		AND user_ip not in ('119.73.140.50', '119.73.140.28') -- SG DPC&FSC 유선&무선 Active
            		AND user_ip not in ('129.126.217.2', '129.126.217.28') -- SG DPC&FSC Backup
            		AND user_ip not in ('14.23.101.234') -- CNDPC(광저우) 유선
            		AND user_ip not in ('112.64.177.10') -- CNDPC(상해) 유선
            )
            
            INSERT OVERWRITE TABLE recommend_goods.rg_v3_temp_daily_page_access_log_info
            PARTITION(svc_nation_cd)
            SELECT
            	COALESCE(CASE WHEN A.cust_no IS NOT NULL THEN A.cust_no ELSE B.cust_no END, A.tracking_session_id) cust_no,
            	A.ref_page_no, 
            	A.ref_page_value, 
            	A.flowpath_no, 
            	A.flowpath_value,
            	A.flowpath_type,
            	A.count_banner_no,
            	A.page_no, 
            	A.page_value,
            	A.group_code,
            	A.gd_no, 
            	A.gdlc_cd, 
            	A.gdmc_cd, 
            	A.gdsc_cd, 
            	A.add_info4,
            	A.log_dt,
            	A.svc_nation_cd
            FROM daily_web_logs A
            JOIN
            	(
            	SELECT 
            		tracking_session_id, 
            		cust_no, 
            		svc_nation_cd
            	FROM default.cust_tracking_info
            	WHERE
            		partition_dt IN 
            		(
            		DATE_SUB(CURRENT_DATE,1),
            		DATE_SUB(CURRENT_DATE,2),
            		DATE_SUB(CURRENT_DATE,3),
            		DATE_SUB(CURRENT_DATE,4),
            		DATE_SUB(CURRENT_DATE,5),
            		DATE_SUB(CURRENT_DATE,6),
            		DATE_SUB(CURRENT_DATE,7)
            		)
            	) B
            ON
            	A.svc_nation_cd = B.svc_nation_cd
            	AND A.tracking_session_id = B.tracking_session_id
            WHERE
            	page_no IN ('50', '51', '356', '106', '423', '502', '533')

    2. daily_web_log 데이터 
        - insert_cart 데이터와 search, category pv 데이터 분리 및 flow_set 데이터 생성 준비

            DROP TABLE recommend_goods.rg_v3_temp_daily_preprocessed_page_access_log_info;
            CREATE TABLE recommend_goods.rg_v3_temp_daily_preprocessed_page_access_log_info
            (
            	cust_no string,
            	flowpath_value string,
            	gd_no string,
            	new_group_cd string,
            	group_cd string,
            	gdlc_cd string,
            	gdmc_cd string,
            	gdsc_cd string,
            	reg_dt string,
            	svc_nation_cd string
            )
            PARTITIONED BY (channel_type string)
            ROW FORMAT DELIMITED
            FIELDS TERMINATED BY '\001'
            STORED AS ORC TBLPROPERTIES("orc.compress" = "SNAPPY")

            WITH goods_info AS
            (
            	SELECT
            		B.new_group_cd,
              		A.group_code group_cd,
            		A.gd_no,
            		A.gdlc_cd,
            		A.gdmc_cd,
            		A.gdsc_cd,
            		A.svc_nation_cd
            	FROM
            		(
            		SELECT
            			group_code,
            			gd_no,
            			gdlc_cd,
            			gdmc_cd,
            			gdsc_cd,
            			svc_nation_cd 
            		FROM
            			(
            			SELECT
            				row_number() over (partition by svc_nation_cd, gd_no order by log_dt desc) row_no, 
            				group_code,
            				gd_no, 
            				gdlc_cd, 
            				gdmc_cd, 
            				gdsc_cd, 
            				svc_nation_cd
            			FROM recommend_goods.rg_v3_temp_daily_page_access_log_info
            			WHERE
            				page_no IN ('50', '51', '356')
            			) R
            		WHERE
            			row_no = 1
            		) A
            	LEFT OUTER JOIN
            		(	
            		SELECT
            			*
            		FROM recommend_goods.rg_corr_group_map
            		WHERE
            			 group_type = 'L'
            		) B
            	ON
            		A.svc_nation_cd = B.svc_nation_cd
            		AND A.gdlc_cd = B.cate_cd
            ), 
            preprocessed_insert_page_access_log AS
            (
            	SELECT
            		A.cust_no,
              		A.flowpath_value,
              		A.gd_no,
              		B.new_group_cd,
              		B.group_cd,
              		B.gdlc_cd,
              		B.gdmc_cd,
              		B.gdsc_cd,
              		A.reg_dt,
              		A.svc_nation_cd,
            		'insert' channel_type
            	FROM
            		(
            		SELECT 
            			DISTINCT
            			cust_no,
            			flowpath_value,
            			ref_page_value gd_no,
            			TO_DATE(log_dt) reg_dt,
            			svc_nation_cd
            		FROM rg_v3_temp_daily_page_access_log_info
            		WHERE
            			page_no IN ('106', '423', '502', '533')
            			AND ref_page_no IN ('50', '51', '356')
            			AND flowpath_no IN ('585', '372', '109', '480')
            		) A
            	JOIN goods_info B
            	ON
            		A.svc_nation_cd = B.svc_nation_cd
            		AND A.gd_no = B.gd_no
            ),
            preprocessed_cate_page_access_log AS
            (
            	SELECT
            		A.cust_no,
              		A.flowpath_value,
              		A.gd_no,
              		B.new_group_cd,
              		B.group_cd,
              		B.gdlc_cd,
              		B.gdmc_cd,
              		B.gdsc_cd,
              		A.reg_dt,
              		A.svc_nation_cd,
            		'cate' channel_type
            	FROM
            		(
            		SELECT 
            			DISTINCT
            			cust_no,
            			gd_no,
            			flowpath_value,
            			TO_DATE(log_dt) reg_dt,
            			svc_nation_cd
            		FROM rg_v3_temp_daily_page_access_log_info
            		WHERE
            			page_no IN ('50', '51', '356')
            			AND flowpath_no IN ('12', '355', '582', '583', '584', '592', '593', '594')
            			AND ref_page_no = flowpath_no
            			AND ref_page_value = flowpath_value
            			AND SUBSTR(flowpath_value, 1, 1) IN ('2', '3')
            		) A
            	JOIN goods_info B
            	ON
            		A.svc_nation_cd = B.svc_nation_cd
            		AND A.gd_no = B.gd_no
            ),
            preprocessed_search_page_access_log AS
            (
            	SELECT
            		A.cust_no,
            		A.flowpath_value,
            		A.gd_no,
            		B.new_group_cd,
            		B.group_cd,
            		B.gdlc_cd,
            		B.gdmc_cd,
            		B.gdsc_cd,
            		A.reg_dt,
            		A.svc_nation_cd,
            		'search' channel_type
            	FROM
            		(
            		SELECT
            			DISTINCT
            			cust_no,
            			gd_no,
            			flowpath_value,
            			TO_DATE(log_dt) reg_dt,
            			svc_nation_cd
            		FROM rg_v3_temp_daily_page_access_log_info
            		WHERE
            			page_no IN ('50', '51', '356')
            			AND ref_page_no IN ('585', '372', '109')
            			AND flowpath_no IN ('585', '480', '372', '109')
            			AND flowpath_value = ref_page_value
            		) A
            	JOIN goods_info B
            	ON
            		A.svc_nation_cd = b.svc_nation_cd
            		AND A.gd_no = B.gd_no
            )
            
            INSERT OVERWRITE TABLE recommend_goods.rg_v3_temp_daily_preprocessed_page_access_log_info
            PARTITION(channel_type)
            SELECT * FROM preprocessed_insert_page_access_log
            UNION ALL
            SELECT * FROM preprocessed_cate_page_access_log
            UNION ALL
            SELECT * FROM preprocessed_search_page_access_log

    3. Target Goods Select 추천 대상이 될 상품 선정
        - 혹시 모르기 때문에 goods_summary_info 사용하여 타겟 대상 선정

            DROP TABLE recommend_goods.rg_v3_temp_daily_goods_info;
            CREATE TABLE recommend_goods.rg_v3_temp_daily_goods_info
            (
            	gd_no string,
            	daily_pv_cnt int,
            	daily_contr_cnt int,
            	weekly_pv_cnt int,
            	weekly_contr_cnt int,
            	svc_nation_cd string
            )
            ROW FORMAT DELIMITED
            FIELDS TERMINATED BY '\001'
            STORED AS ORC TBLPROPERTIES("orc.compress" = "SNAPPY")

            INSERT OVERWRITE TABLE recommend_goods.rg_v3_temp_daily_goods_info
            SELECT
            	gd_no,
            	daily_pv_cnt,
            	daily_contr_cnt,
            	weekly_pv_cnt,
            	weekly_contr_cnt,
            	svc_nation_cd
            FROM
            	(
            	SELECT
            		ROW_NUMBER() OVER(PARTITION BY svc_nation_cd, gd_no ORDER BY log_dt desc) row_no,
            		gd_no,
            		cust_pv_feature['total_pv_cnt'] daily_pv_cnt,
            		cust_contr_feature['total_contr_cnt'] daily_contr_cnt,
            		cust_pv_feature_week['total_pv_cnt'] weekly_pv_cnt,
            		cust_contr_feature_week['total_contr_cnt'] weekly_contr_cnt,
            		svc_nation_cd
            	FROM summary_info.goods_summary_info
            	WHERE
            		log_dt IN 
            				( 
            					DATE_SUB(current_date,1),
            					DATE_SUB(current_date,2)
            				)
            	) R
            WHERE
            	row_no = 1

    4. flow_set 생성 

            DROP TABLE recommend_goods.rg_v3_daily_flow_set_info;
            CREATE TABLE recommend_goods.rg_v3_daily_flow_set_info
            (
            	count int,
            	group_cd string,
            	gd_no string,
            	gdlc_cd string,
            	gdmc_cd string,
            	gdsc_cd string,
            	next_group_cd string,
            	next_gd_no string,
            	next_gdlc_cd string,
            	next_gdmc_cd string, 
            	next_gdsc_cd string,
            	reg_dt string,
            	svc_nation_cd string
            )
            PARTITIONED BY (channel_type string)
            ROW FORMAT DELIMITED
            FIELDS TERMINATED BY '\001'
            STORED AS ORC TBLPROPERTIES("orc.compress" = "SNAPPY")

        - insert

            WITH temp_page_access_log_data_type_insert AS
            (
            	SELECT 
            		*
            	FROM recommend_goods.rg_v3_temp_daily_preprocessed_page_access_log_info
            	WHERE
            		channel_type = 'insert'
            )
            INSERT OVERWRITE TABLE recommend_goods.rg_v3_daily_flow_set_info
            PARTITION(channel_type)
            SELECT
            	count(*) insert_cnt,
            	A.group_cd,
            	A.gd_no,
            	A.gdlc_cd,
            	A.gdmc_cd,
            	A.gdsc_cd,
            	B.group_cd next_group_cd,
            	B.gd_no next_gd_no,
            	B.gdlc_cd next_gdlc_cd,
            	B.gdmc_cd next_gdmc_cd,
            	B.gdsc_cd next_gdsc_cd,
            	A.reg_dt,
            	A.svc_nation_cd,
            	'insert' channel_type
            FROM temp_page_access_log_data_type_insert A
            JOIN temp_page_access_log_data_type_insert B
            ON
            	A.svc_nation_cd = B.svc_nation_cd
            	AND A.cust_no = B.cust_no
            	AND A.reg_dt = B.reg_dt
            WHERE
            	A.gd_no <> B.gd_no
            GROUP BY
            	A.group_cd,
            	A.gd_no,
            	A.gdlc_cd,
            	A.gdmc_cd,
            	A.gdsc_cd,
            	B.group_cd,
            	B.gd_no,
            	B.gdlc_cd,
            	B.gdmc_cd,
            	B.gdsc_cd,
            	A.reg_dt,
            	A.svc_nation_cd

        - search

            WITH temp_page_access_log_data_type_search AS
            (
            	SELECT 
            		*
            	FROM recommend_goods.rg_v3_temp_daily_preprocessed_page_access_log_info
            	WHERE
            		channel_type = 'search'
            )
            
            INSERT OVERWRITE TABLE recommend_goods.rg_v3_daily_flow_set_info
            PARTITION(channel_type)
            SELECT
            	count(*) search_cnt,
            	A.group_cd,
            	A.gd_no,
            	A.gdlc_cd,
            	A.gdmc_cd,
            	A.gdsc_cd,
            	B.group_cd next_group_cd,
            	B.gd_no next_gd_no,
            	B.gdlc_cd next_gdlc_cd,
            	B.gdmc_cd next_gdmc_cd,
            	B.gdsc_cd next_gdsc_cd,
            	A.reg_dt,
            	A.svc_nation_cd,
            	'search' channel_type
            FROM temp_page_access_log_data_type_search A
            JOIN temp_page_access_log_data_type_search B
            ON
            	A.svc_nation_cd = B.svc_nation_cd
            	AND A.reg_dt = B.reg_dt
            	AND A.cust_no = B.cust_no
            	AND A.flowpath_value = B.flowpath_value
            WHERE
            	A.gd_no <> B.gd_no
            GROUP BY
            	A.group_cd,
            	A.gd_no,
            	A.gdlc_cd,
            	A.gdmc_cd,
            	A.gdsc_cd,
            	B.group_cd,
            	B.gd_no,
            	B.gdlc_cd,
            	B.gdmc_cd,
            	B.gdsc_cd,
            	A.reg_dt,
            	A.svc_nation_cd

        - cate

            WITH temp_page_access_log_data_type_cate AS
            (
            	SELECT 
            		*
            	FROM recommend_goods.rg_v3_temp_daily_preprocessed_page_access_log_info
            	WHERE
            		channel_type = 'cate'
            )
            INSERT OVERWRITE TABLE recommend_goods.rg_v3_daily_flow_set_info
            PARTITION(channel_type)
            SELECT
            	count(*) cate_cnt,
            	A.group_cd,
            	A.gd_no,
            	A.gdlc_cd,
            	A.gdmc_cd,
            	A.gdsc_cd,
            	B.group_cd next_group_cd,
            	B.gd_no next_gd_no,
            	B.gdlc_cd next_gdlc_cd,
            	B.gdmc_cd next_gdmc_cd,
            	B.gdsc_cd next_gdsc_cd,
            	A.reg_dt,
            	A.svc_nation_cd,
            	'cate' channel_type
            FROM temp_page_access_log_data_type_cate A
            JOIN temp_page_access_log_data_type_cate B
            ON
            	A.svc_nation_cd = B.svc_nation_cd
            	AND A.reg_dt = B.reg_dt
            	AND A.cust_no = B.cust_no
                AND A.flowpath_value = B.flowpath_value
            WHERE
            	A.gd_no <> B.gd_no
            GROUP BY
            	A.group_cd,
            	A.gd_no,
            	A.gdlc_cd,
            	A.gdmc_cd,
            	A.gdsc_cd,
            	B.group_cd,
            	B.gd_no,
            	B.gdlc_cd,
            	B.gdmc_cd,
            	B.gdsc_cd,
            	A.reg_dt,
            	A.svc_nation_cd

- 중간검증
    - 장바구니 flow_set
        - 같은 검색을 통해 상품 상세에서 장바구니로 담은 상품의 flow_set 구성
        - 전반적으로 유사한 상품이 나오나 간혹 유사하지 않은 상품이 나오게됨
        - page_no = 502 ref_page_no = 51 flowpath_no = 480 flowpath_value 사용시 
        여러 페이지 사용시 문제 생김 또는 검색어 자체가 브랜드인 경우
        - 해결 방법은 찾지 못함 (이런식의 데이터는 많지 않기 때문에 노이즈로 무시?)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c4b52485-3e33-4c97-a369-1f85c3ed4a10/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c4b52485-3e33-4c97-a369-1f85c3ed4a10/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6834c96f-5470-48b6-a9d4-907c950febbf/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6834c96f-5470-48b6-a9d4-907c950febbf/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/abd8afcb-5569-48e1-aa07-78171164ab87/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/abd8afcb-5569-48e1-aa07-78171164ab87/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/036bc8cf-3ddc-476a-951f-6ca3df1fa071/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/036bc8cf-3ddc-476a-951f-6ca3df1fa071/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/90f1f347-3023-44a8-8d83-4916feb6e244/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/90f1f347-3023-44a8-8d83-4916feb6e244/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b09c3467-22e1-41c1-b622-e0679ec56c9b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b09c3467-22e1-41c1-b622-e0679ec56c9b/Untitled.png)

    - 검색 flow_set
        - 대체적으로 괜찮은 성능 but, 적은 양의 flow_set 누적 데이터 활용이 필요
        - 검색으로 상품을 본 경우 모바일 같은 경우에는 count_banner_no가 null값이 들어옴
        PC 모바일 둘다 같은 banner_no로 들어오는줄 암
        - PV가 검색인 경우의 ref_page_value 와 flowpath_value가 같은 조건

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0bcad59a-846f-4aaa-a5dd-e5c70f0bb83d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0bcad59a-846f-4aaa-a5dd-e5c70f0bb83d/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e7c62dc-5f66-4ab6-a6eb-9b294bda9e56/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e7c62dc-5f66-4ab6-a6eb-9b294bda9e56/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a127ee2-a8b8-4a19-864e-658b8a60c62b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a127ee2-a8b8-4a19-864e-658b8a60c62b/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6f2ee3cb-a9b8-4bb2-a182-91a4480fa816/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6f2ee3cb-a9b8-4bb2-a182-91a4480fa816/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3701bfb9-8c5f-48d7-9aea-3bc0565ddf00/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3701bfb9-8c5f-48d7-9aea-3bc0565ddf00/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d833211d-31fc-43a8-8f51-6c010c87f274/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d833211d-31fc-43a8-8f51-6c010c87f274/Untitled.png)

    - 카테고리 flow_set
        - 카테고리 검색으로 상품을 본 경우는 고객의 니즈에 따라 큰 범위의 유사와 
        작은 범위의 유사로 나눌 수 있음
        - 대카테고리의 경우는 유사한 상품보다 비슷한 종류의 상품을 보는 경우가 많음
        - flow_set 구성시 flowpath_value가 중, 소카테고리인 데이터만 사용
        - 연관 상품의 개념이라고 생각하고 사용 vs 노이즈라 생각하고 제거
        아니면 깐깐한 필터 사용 ?

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5c232611-2f74-463e-bfa2-1d2b4c9f1ab0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5c232611-2f74-463e-bfa2-1d2b4c9f1ab0/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ba9af49-3642-483a-bb9c-2f02926284c2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ba9af49-3642-483a-bb9c-2f02926284c2/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da35e450-39e5-4709-8c1a-a76a52244a0e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da35e450-39e5-4709-8c1a-a76a52244a0e/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b3e2ebb5-4669-409c-a5ad-c5e2dad2fe32/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b3e2ebb5-4669-409c-a5ad-c5e2dad2fe32/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db8617f3-262d-4273-8824-3805efc55537/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db8617f3-262d-4273-8824-3805efc55537/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d83fbd0e-f328-4fda-be99-d44bb40576ba/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d83fbd0e-f328-4fda-be99-d44bb40576ba/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0f80ec3e-59a9-4734-8ce8-3fac5f29671d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0f80ec3e-59a9-4734-8ce8-3fac5f29671d/Untitled.png)

            ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bfc7a967-d375-4659-83d8-0840dffbf787/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bfc7a967-d375-4659-83d8-0840dffbf787/Untitled.png)

- ML Row Data 구성
    - 최신성을 고려하여 3개월 데이터만? 속도 상승, 정확도?
    - 키워드 검색으로 검색한 후 고객의 click tracking에서  브랜드 키워드를 제외한 키워드로 학습 데이터 구성
    - 키워드 - 상품, 상품 형태로 학습
    - 키워드 - 상품의 search_tag_detail, 검색어, 제목 ← 브랜드 키워드를 제외한  단어로 학습
    - target 상품의  유사 상품 추측
    - 유사 상품의 키워드 추출 → 키워드로 유사 키워드 추측
    - 유사 키워드로 유사 상품 추출

---

### 수정사항 (2020-02-13 리뷰)

- 카테고리 full outer 조인 말고 A → B → C  (A→B), (B→C) 형태로 데이터 확인 후
중, 소 카테고리 제한 또는 카테고리 정보 사용고려
- 장바구니에서 여러 페이지 사용시 생기는 노이즈 데이터는 무시(양이 많지 않음으로)
- 검색에서 PC 버전은 count_banner_no 사용 모바일은 사용 x (ref_page_value == flowpath_value)

---

### 임시 수식

- 상품기반 집계
    - Flow Set에 대해 gd_no를 기준으로 각 채널에 대해서 MINMAX Scale 취한 후 각 채널의 가중치를 반영하여 0 ~ 1 사이의 값으로 도출

        (accum_insert_flow_set - min) / (max - min) * 0.2 → 최고 0.2 반영

        (accum_search_flow_set - min) / (max - min) * 0.6 →  최고 0.6 반영

        (accum_cate_flow_set - min) / (max - min) * 0.2 →  최고 0.2 반영 

        전체 값 더하여 0 ~ 1 사이의 값으로 나오도록  

[상품 추천 비교.xlsx](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c6400d22-2aa6-41cd-a8d4-d6bc46fbaf24/__.xlsx)

- 이슈 사항
    - scale 사용시 MAX 값이 작은 경우 변별력이 떨어진다 → 최소한의 count에 대한 filter
    ex) count가 최소 5이상의 값만 사용?
    - 검색어가 브랜드일때는 본 상품이 균일하지 않는 경우가 많음

        이 경우에는 집계에서 제거하기가 쉽지 않음, 브랜드 명을 제거 하다간 데이터가 줄어들 듯 

        ex) 샤오미

    - 검색어가 브랜드 + 특정 상품인 경우 ML 데이터가 더 유사한 경우가 있음
    - 누적 데이터를 쌓아서 업데이트 안되는 flow_set이 있는지 확인 해야함

        생각보다 하루만 flow_set 되고 업데이트 안되는 데이터가 많은 것 같음

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a9db78cc-1b80-4de4-be17-31e42f2c7f1c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a9db78cc-1b80-4de4-be17-31e42f2c7f1c/Untitled.png)

    - 가중치를 위한 featrue 중 daily_pv, weekly_pv가 있는데 이거보다는 contr_cnt 데이터가 더 좋을듯

---

### 수정사항(2020-02-26 리뷰)

- 누적 Count가 높은 상품중에 과거 이상했던 상품 비교
- 누적 Count에 따른 ML 가중치 집계 가중치 다르게
어떤 경우에 ML 가중치가 주어지는게 좋고 어떤 경우에 집계 가중치를 주어지는게 좋은지
- ML 키워드 데이터 확인

    키워드 → 상품

    상품 → 키워드 

    두가지 데이터 모두 사용해야 높은 정확도 

- 누적된 Flow set 텍스트 연관성을 비교를 기준으로 삭제 로직 생각

---

ML 데이터

- 현재 사용하는 데이터 타입
    - 1년치 집계 사용
    - 검색의 경우 리턴되는 검색결과 수 add_info1이 국가별로 다르게 설정 그 이상이 되는 상품은 제외
    - 'click_list', 'cate_list', 'recom_list', 'recomkeyword_list', 'target_list'
    - click_list

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6915a07-f824-4681-b8d4-c2161ad35594/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6915a07-f824-4681-b8d4-c2161ad35594/Untitled.png)

    - cate_list

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/611104ab-c81a-4295-81d7-21193762e312/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/611104ab-c81a-4295-81d7-21193762e312/Untitled.png)

    - recom_list

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/917055e7-6e64-40e3-8e0a-f1d6e34c6476/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/917055e7-6e64-40e3-8e0a-f1d6e34c6476/Untitled.png)

    - recomkeyword_list

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1bd561f6-6cce-4e72-bb11-d8dab953139b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1bd561f6-6cce-4e72-bb11-d8dab953139b/Untitled.png)

- 웹 로그 데이터 집계시 add_info1의 값( 검색 상품의 총 개수)이 특정 개수 이상인 상품 제거

    → 넓은 범위의 검색어로 검색했을 시, 브랜드 이름으로 검색했을 시 데이터  분산을 고려한 로직 

    → 집계 데이터와 ML 데이터 두가지 모두 사용하기 때문에 후보 상품들이 많이 나오도록 

- ML 데이터 또한 상품 검색과 관련된 데이터만 사용하도록 함

---

- 기존 데이터와 비교 검증 → 성능 평가
- Score 구성
    - 전체적인 수식 정의 및 가중치 정의
- 최종검증
