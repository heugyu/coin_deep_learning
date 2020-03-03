# Git은 코드를 관리하고 배포, Docker는 환경을 관리하고 배포
- Container는 우리가 사용하려고 하는 OS와 Tool, 의존성 컨테이너 안에 개발 환경을 셋업하는 것
- Container는 Image로 부터 만들어지는데, 이는 마치 Class로 부터 Instance가 생성되는 개념과 흡사 
- Docker Image에 OS, 환셩설정을 명시하거나 셋업한 뒤 Host 위에 Container로 올려서 실행

# Docker Setting
1. Pytorch Docker Image Pull
2. ngrok.io를 통한 외부 원격 설정  
3. Coin Historical Data Crawling
> 이왕이면 개발한 지표값이 포함된 형태의 session, opne, close, high, close, develop_indicate 데이터
> 아니면 기본 데이터로 지표값 생성해야할 듯 

# Coin Historical Data Crawling
- 크롤링 이용하여 데이터 추출
- 데이터는 다음과 같은 형태
  ![image](https://user-images.githubusercontent.com/60495142/75801094-ee833600-5dbd-11ea-8bcb-ca3f8cb6555e.png)
  ![image](https://user-images.githubusercontent.com/60495142/75801225-1e323e00-5dbe-11ea-8aca-7e9d99f58d76.png)

# Pandas를 이요한 전처리 및 데이터 분석

# 기본 데이터로 RNN

# RNN 및 강화학습
