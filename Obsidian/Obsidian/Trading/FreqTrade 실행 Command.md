



  
데이터 다운로드  
freqtrade download-data -c "user_data/config-static-tutorial.json" --days 1000 -t 5m 15m 30m 1h 4h  

freqtrade download-data -c "user_data/config-static-tutorial.json" --days 2000 -t 5m 15m 30m 1h 4h  
  
  
  
백테스트 실행  
freqtrade backtesting --config user_data/config-static-tutorial-new.json --strategy Strategy003  --timerange=20210301-  

freqtrade backtesting --config user_data/config-static-tutorial-new.json --strategy VBO_v2  --timerange=20210301-  

freqtrade backtesting --config user_data/config-static-tutorial-new2.json --strategy LarryWilliamsVolatilityBreakout  --timerange=20210301-  

freqtrade backtesting --config user_data/config-static-tutorial-new2.json --strategy CustomBitcoinStrategy --timerange=20190101-  

freqtrade backtesting --config user_data/config-static-tutorial-new2.json --strategy CustomBitcoinStrategy --timerange=20190101- --fee 0.001  
  
  
백테스트 (기간별 손익)  
freqtrade backtesting --config user_data/config-static-tutorial-new2.json --strategy CustomBitcoinStrategy --timerange=20190101- --breakdown day week month  
  
  
  
  
  
봇 실행 (+웹 UI)  
freqtrade trade --config user_data/config-static-tutorial-new2.json --strategy CustomBitcoinStrategy  
  
  
최적화 (안됨)  
freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --config user_data/config-static-tutorial-new.json --strategy CustomBitcoinStrategy