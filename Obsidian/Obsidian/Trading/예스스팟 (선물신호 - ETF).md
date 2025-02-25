










```
var 거래수량 = 1
var ChoicingMarketdata; // MarketData가 다중일때 사용
var BuyMulti, MD1_주문수량, MD2_주문수량;
// ----- 주문로직 변수 끝


function Main_OnStart()   
{ 
	var d = new Date();
	var YYYYMMDD = d.getFullYear()*10000 + (d.getMonth() + 1)*100 + d.getDate();
	var HHMMSS = d.getHours()*10000+d.getMinutes()*100+d.getSeconds();


	if(MarketData1.current < MarketData2.current)
	{
		BuyMulti = MarketData1.current / MarketData2.current;
		MD1_주문수량 = Math.floor(거래수량 / BuyMulti)
		MD2_주문수량 = 거래수량
	}
	else if (MarketData1.current > MarketData2.current)
	{
		BuyMulti = MarketData2.current / MarketData1.current;
		MD2_주문수량 = Math.floor(거래수량 / BuyMulti)
		MD1_주문수량 = 거래수량			
	}
	else
	{
		MD1_주문수량 = 거래수량
		MD2_주문수량 = 거래수량
	}

}  

function Chart1_OnRiseSignal(Signal)
{
	var d = new Date();
	var YYYYMMDD = d.getFullYear()*10000 + (d.getMonth() + 1)*100 + d.getDate();
	var HHMMSS = d.getHours()*10000+d.getMinutes()*100+d.getSeconds();

    if (Signal.signalKind == 1) // buy : KODEX150
      {
	  		ChoicingMarketdata = MarketData1;
			OrderBuy(ChoicingMarketdata.code, Signal.signalKind, MD1_주문수량);

      }
    else if (Signal.signalKind == 2) // ExitLong : KODEX150
      {
			ChoicingMarketdata = MarketData1;
			OrderSell(ChoicingMarketdata.code, Signal.signalKind, MD1_주문수량);
		
      }
    else if (Signal.signalKind == 3) // Sell : KODEX150 선물인버스
      {
	  		ChoicingMarketdata = MarketData2;
			OrderBuy(ChoicingMarketdata.code, 1, MD2_주문수량);

      }
    else if (Signal.signalKind == 4) // ExitShort : KODEX150 선물인버스
      {
			ChoicingMarketdata = MarketData2;
			OrderSell(ChoicingMarketdata.code, 2, MD2_주문수량);

      }
}

```


그냥 제가 쓰는 코드중 간단한 주문코드만 추출해서 보냅니다. 거래수량이 1일때 비율에 따라서 가격이 싼 주문을 그 비율에 맞게 주문하는 코드입니다. MarketData1과 MarketData2는 ETF와 인버스ETF 넣고, chart1은 예스랭귀지의 차트와 연동하시면 될듯 합니다. 좋은 하루 되셔요.



```
var 거래수량 = 1
var ChoicingMarketdata; // MarketData가 다중일때 사용
var BuyMulti, MD1_주문수량, MD2_주문수량;
// ----- 주문로직 변수 끝


function Main_OnStart()   
{ 
	var d = new Date();
	var YYYYMMDD = d.getFullYear()*10000 + (d.getMonth() + 1)*100 + d.getDate();
	var HHMMSS = d.getHours()*10000+d.getMinutes()*100+d.getSeconds();


	if(MarketData1.current < MarketData2.current)
	{
		BuyMulti = MarketData1.current / MarketData2.current;
		MD1_주문수량 = Math.floor(거래수량 / BuyMulti)
		MD2_주문수량 = 거래수량
	}
	else if (MarketData1.current > MarketData2.current)
	{
		BuyMulti = MarketData2.current / MarketData1.current;
		MD2_주문수량 = Math.floor(거래수량 / BuyMulti)
		MD1_주문수량 = 거래수량			
	}
	else
	{
		MD1_주문수량 = 거래수량
		MD2_주문수량 = 거래수량
	}

}  

function Chart1_OnRiseSignal(Signal)
{
	var d = new Date();
	var YYYYMMDD = d.getFullYear()*10000 + (d.getMonth() + 1)*100 + d.getDate();
	var HHMMSS = d.getHours()*10000+d.getMinutes()*100+d.getSeconds();

    if (Signal.signalKind == 1) // buy : KODEX150
      {
	  		ChoicingMarketdata = MarketData1;
			Account1.OrderBuy(ChoicingMarketdata.code,  MD1_주문수량, 0, 1);

      }
    else if (Signal.signalKind == 2) // ExitLong : KODEX150
      {
			ChoicingMarketdata = MarketData1;
			Account1.OrderSell(ChoicingMarketdata.code,  MD1_주문수량, 0, 1);
		
      }
    else if (Signal.signalKind == 3) // Sell : KODEX150 선물인버스
      {
	  		ChoicingMarketdata = MarketData2;
			Account1.OrderBuy(ChoicingMarketdata.code,  MD1_주문수량, 0, 1);

      }
    else if (Signal.signalKind == 4) // ExitShort : KODEX150 선물인버스
      {
			ChoicingMarketdata = MarketData2;
			Account1.OrderSell(ChoicingMarketdata.code,  MD1_주문수량, 0, 1);

      }
}
```


[카레] [오후 5:59] 계좌를 지우면서 Account1을 삭제했나보네요. 아마 이렇게 써야지 시장가로 주문이 나갈거에요. ETF라서 슬리피지는 그리 크지 않을것 같긴합니다. 한번 테스트 해보고 쓰셔요.
[카레] [오후 5:59] 다시보니 뭔가 이상해서.. 다시 확인해보고 보내드립니다.






| KODEX 코스닥150레버리지          | 233740 | 코스닥150 (x2) | 8760  |
| ------------------------- | ------ | ----------- | ----- |
| 삼성 인버스 2X 코스닥150 선물 ETN   | 530107 | 코스닥150 (x2) |       |
| KODEX 레버리지                | 122630 | 코스피200 (x2) | 16390 |
| KODEX 200선물 인버스 2X        | 252670 | 코스피200 (x2) | 2160  |
|                           |        |             |       |
| KODEX 미국달러선물인버스2X         | 261260 | 달러 (x2)     | 5340  |
| KODEX 미국달러선물 레버리지         | 261250 | 달러 (x2)     | 16305 |
| KODEX 코스닥150              | 229200 | 코스닥150      | 13110 |
| KODEX 코스닥150선물인버스         | 251340 | 코스닥150      | 3575  |
|                           |        |             |       |
| 미래에셋 인버스 2X 코스닥150 선물 ETN | 520057 | 코스닥150 (x2) | 5695  |













