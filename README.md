# Coin Deep Learning
- Tradingview에서 거래량 관련 지표 개발을 하다가 이를 바탕으로 back test의 필요성
- 그동안 미루고 미뤘던 코인데이터 딥러닝 테스트의 필요성 
- 기본적인 open, close, hihg, low의 RNN(LSTM), 강화학습은 문제가 있음(피쳐값의 분산), 가격이 오르고 내리고에 별 영향이 없음 
- 보다 확실한 피쳐 개발이 우선되어 했고 이를 거래량을 통한 지표를 개발 하였음 
- 거래량을 통안 지표의 기반은 Wyckoff 패턴을 주요 골자로 생각 
- 이를 통해 총 3가지의 지표를 개발 하였음 

---

# Indicate
1. ADV (Accumulation/Distribution Volume)
- BCT의 경우 거래소 마다 거래량, 가격, 유동량의 차이가 있기 때문에 전체를 아우르는 많은 거래소의 볼륨을 함께 볼 필요가 있음 
- 예를 들어 거래소 별 프리미엄이 생길수도, 덤핑이 일어날 수 있음 
- 그리고 거래량 보다는 가격이 포함된 거래대금을 보는 것이 시장 자금의 유동성을 파학할 수 있음
- 이를 고려하여 상위 거래소의 voluem, price 를 사용하여 A/D로 수요/공급의 누적 대금을 파악 
- 실제 볼륨과 달리 누적된 거래대금은 다이버전스를 찾기 쉽게 함 
- https://github.com/heugyu/coin_deep_learning/issues/1#issue-573900101

2. PF S/R (Pivots Fibonacci S/R)
- 가격의 pivot을 시작과 끝으로 한 피보나치 레벨의 지지
- volume의 price를 평균화한 VWAP 사용 length는 의미있는 34 사용
- Bollinger를 기반으로 한 지지, 저장 레벨 
- https://github.com/heugyu/coin_deep_learning/issues/2#issue-573936057
  
3. ALL
- 가격의 pivot시 거래량을 표현 (Wyckoff 패턴에서 중요)
- 차트와 보조지표간 수렴 및 발산을 캐치
- 멀티 EMA
- multiplier를 활용한 채널
- https://github.com/heugyu/coin_deep_learning/issues/3#issue-573938577
  
> 기본적인 지표와 더불어 가격의 상승과 하락에 주요한 영향이 있는 지표를 피쳐 값으로 사용할 때 RNN, 강화 학습에서는 어떻게 될지?
 
코인 및 주식 데이터로 RNN, 강화학습, 퀀트, 알고 매매 테스트 
