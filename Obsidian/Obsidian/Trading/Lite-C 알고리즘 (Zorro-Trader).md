


  
공유하기

![User](https://s.gravatar.com/avatar/3d0c6ae88ed7e354a187cef29c93d4fa?s=480&r=pg&d=https%3A%2F%2Fcdn.auth0.com%2Favatars%2Fki.png)

##### 나의 말:

바이낸스에서 크립토 선물 거래를 하려고 해. 선물 거래금액 기준 최근 24시간 거래량이 상위 15위 안에 든 것만 진입 신호를 발생시키고 싶어. 진입 로직은 래리 윌리엄스 변동성 돌파야. 청산은 ATR 기반 샹들리에 청산을 적용해 줘.

네, 바이낸스에서 크립토 선물 거래를 위한 전략을 Zorro lite-C로 구현하는 방법을 설명해 줄게.

## 요구사항 분석

1. **최근 24시간 거래량 기준 상위 15위 암호화폐 선물만 진입**
    
    - 바이낸스 API를 이용하여 거래량 데이터를 가져와야 함.
    - Zorro에서 거래할 자산을 동적으로 선택.
2. **진입 전략: 래리 윌리엄스 변동성 돌파 전략**
    
    - 특정 기준(예: 전일 고가 + 변동성)을 돌파하면 매수.
    - 변동성은 ATR 또는 다른 지표를 사용 가능.
3. **청산 전략: ATR 기반 샹들리에 청산**
    
    - 매수 시 ATR을 기준으로 동적 스탑로스를 설정하여 청산.

---

## Zorro lite-C 코드 예제

### 1. **바이낸스 API에서 24시간 거래량 기준 상위 15개 종목 가져오기**

Zorro는 기본적으로 바이낸스 API를 직접 호출하는 기능이 없으므로, 거래량 데이터를 별도로 가져와야 해. 이를 위해 파이썬 스크립트로 데이터를 저장한 후, Zorro에서 불러오는 방법이 일반적이야.

#### 📌 **외부 데이터 저장 예제 (Python)**

python

복사편집

```
import requests
import json

# 바이낸스 24시간 거래량 데이터 가져오기
url = "https://api.binance.com/api/v3/ticker/24hr"
response = requests.get(url)
data = response.json()

# 거래량 기준 상위 15개 선물 종목 필터링
top_assets = sorted(data, key=lambda x: float(x['quoteVolume']), reverse=True)[:15]
top_symbols = [asset["symbol"] for asset in top_assets]

# 파일로 저장
with open("top_assets.txt", "w") as f:
    f.write(",".join(top_symbols))

```



#### 📌 **Zorro lite-C에서 데이터 불러오기**

c

복사편집

```
string TopAssets[15]; // 상위 15개 종목 저장 배열

// 외부 파일에서 상위 종목 불러오기
void LoadTopAssets() {
    char Content[512];
    int handle = file_content("top_assets.txt", Content, 512);
    if (handle) {
        int n = 0;
        char* token = strtok(Content, ",");
        while (token && n < 15) {
            TopAssets[n++] = strf(token);
            token = strtok(NULL, ",");
        }
    }
}

```



---

### 2. **래리 윌리엄스 변동성 돌파 전략**

c

복사편집

```
var breakoutLevel;
function run()
{
    if(is(INITRUN)) {
        LoadTopAssets(); // 상위 종목 리스트 로드
    }

    // 현재 선택된 자산이 상위 15위에 포함되는지 확인
    int isTopAsset = 0;
    for(int i = 0; i < 15; i++) {
        if(strstr(Asset, TopAssets[i])) {
            isTopAsset = 1;
            break;
        }
    }
    
    if(!isTopAsset) return; // 제외된 자산이면 거래하지 않음

    set(PARAMETERS); // 매개변수 최적화 활성화
    BarPeriod = 1440; // 1일 봉 기준

    // ATR을 기반으로 변동성 계산
    var atr = ATR(20);

    // 래리 윌리엄스 변동성 돌파
    if(is(NEWDAY)) {
        breakoutLevel = priceHigh(1) + atr * 0.5;
    }

    if(priceClose(0) > breakoutLevel) {
        enterLong();
    }
}

```



---

### 3. **ATR 기반 샹들리에 청산**

c

복사편집
```
function trailExit() {
    var atr = ATR(20);
    Stop = priceHigh(0) - atr * 3; // ATR의 3배를 스탑로스로 설정
}

```



---

### **정리**

1. **바이낸스 API**를 통해 **최근 24시간 거래량 상위 15개 종목**을 가져옴.
2. **Zorro lite-C에서 해당 종목만 거래**하도록 필터링.
3. **래리 윌리엄스 변동성 돌파**로 진입 신호 생성.
4. **ATR 기반 샹들리에 청산**을 적용하여 스탑로스 설정.

이제 이 코드로 원하는 전략을 구현할 수 있어! 🔥 궁금한 점 있으면 질문해줘. 🚀

##### 나의 말:

zorro 에서 쓰는 거래 유니버스는 어떻게 세팅해?

###### ChatGPT의 말:

Zorro에서 **거래 유니버스(Trading Universe)**를 설정하는 방법은 크게 두 가지가 있어:

1. **자산 리스트를 미리 정의** (예: `Assets.csv` 파일 사용)
2. **스크립트 내에서 동적으로 설정** (`asset()` 함수를 이용)

---

## 1️⃣ **미리 정의된 자산 리스트 사용 (`Assets.csv`)**

Zorro는 기본적으로 `History/Assets.csv` 파일에서 거래할 수 있는 자산 정보를 읽어와.  
이 파일을 직접 수정하면 거래할 자산 리스트를 관리할 수 있어.

### **예제: `Assets.csv` 설정**

아래는 BTC, ETH, BNB 선물 거래를 위한 `Assets.csv` 예제야.

csv

복사편집

```
Name,Price,Spread,Pip,Multiplier,Margin,RollLong,RollShort,Symbol
BTCUSDT,60000,10,0.1,1,0.01,0,0,BTCUSDT
ETHUSDT,4000,5,0.1,1,0.01,0,0,ETHUSDT
BNBUSDT,500,2,0.01,1,0.01,0,0,BNBUSDT

```


`Name,Price,Spread,Pip,Multiplier,Margin,RollLong,RollShort,Symbol BTCUSDT,60000,10,0.1,1,0.01,0,0,BTCUSDT ETHUSDT,4000,5,0.1,1,0.01,0,0,ETHUSDT BNBUSDT,500,2,0.01,1,0.01,0,0,BNBUSDT`

📌 **필드 설명**

- `Name`: Zorro에서 사용할 자산 이름
- `Price`: 현재 가격 (백테스트용, 실거래 시 무시됨)
- `Spread`: 스프레드 크기
- `Pip`: 최소 변동 단위
- `Multiplier`: 계약 크기 (선물, CFD 등에서 사용)
- `Margin`: 레버리지 계산을 위한 마진 비율
- `Symbol`: 브로커에서 실제 사용하는 심볼

📌 **사용 방법**  
`Assets.csv` 파일을 설정하면 `asset("BTCUSDT")` 같은 함수 호출로 해당 자산을 쉽게 로드할 수 있어.

---

## 2️⃣ **스크립트 내에서 동적으로 거래 유니버스 설정**

`asset()` 함수를 사용하면 **스크립트 내부에서 특정 자산만 선택하여 거래**할 수 있어.

### **예제: 특정 자산만 거래**

c

복사편집
```
function run()
{
    set(PARAMETERS);
    BarPeriod = 1440; // 1일 봉

    assetList("BTCUSDT,ETHUSDT,BNBUSDT"); // 선택된 자산만 사용

    while(asset(loop(Assets))) {  // 설정한 유니버스 내에서 순회
        var atr = ATR(20);
        if(priceClose(0) > priceHigh(1) + atr * 0.5) {
            enterLong();
        }
    }
}

```



📌 **설명**

- `assetList("...")`: 특정 자산만 거래하도록 설정
- `while(asset(loop(Assets)))`: 정의한 유니버스 내 자산을 순회

---

## 3️⃣ **거래량 기준 동적 유니버스 구성**

네가 말한 **최근 24시간 거래량 기준 상위 15개 종목만 거래**하려면 외부에서 데이터를 가져와야 해.  
아래는 **바이낸스 API에서 가져온 거래량 데이터를 이용하여 Zorro에서 동적으로 유니버스를 설정하는 코드**야.

### **Python으로 상위 15개 자산 저장**

python

복사편집

```
import requests

url = "https://api.binance.com/api/v3/ticker/24hr"
response = requests.get(url)
data = response.json()

top_assets = sorted(data, key=lambda x: float(x['quoteVolume']), reverse=True)[:15]
top_symbols = [asset["symbol"] for asset in top_assets]

# 파일로 저장
with open("top_assets.txt", "w") as f:
    f.write(",".join(top_symbols))

```



### **Zorro lite-C에서 데이터 불러오기**

c

복사편집

```
string TopAssets[15];

void LoadTopAssets() {
    char Content[512];
    int handle = file_content("top_assets.txt", Content, 512);
    if (handle) {
        int n = 0;
        char* token = strtok(Content, ",");
        while (token && n < 15) {
            TopAssets[n++] = strf(token);
            token = strtok(NULL, ",");
        }
    }
}

function run()
{
    if(is(INITRUN)) {
        LoadTopAssets();
    }

    assetList(strf("%s", join(",", TopAssets, 15)));  // 동적으로 유니버스 설정

    while(asset(loop(Assets))) {
        var atr = ATR(20);
        if(priceClose(0) > priceHigh(1) + atr * 0.5) {
            enterLong();
        }
    }
}

```



📌 **설명**

1. **Python**이 바이낸스에서 **거래량 상위 15개 자산을 가져와서 `top_assets.txt`에 저장**
2. **Zorro lite-C**가 `file_content()`를 사용하여 데이터를 읽고 `assetList()`에 적용
3. **백테스트 및 실거래에서 최신 데이터를 반영하여 거래**

---

## **🚀 정리**

1. **정적 유니버스**
    - `Assets.csv`를 사용하여 거래할 자산을 사전에 정의
2. **동적 유니버스 (스크립트 내 설정)**
    - `assetList("BTCUSDT,ETHUSDT,...")`를 사용하여 특정 자산만 거래
3. **실시간 유니버스 (API 활용)**
    - 바이낸스 API를 사용하여 최근 24시간 거래량 기준 상위 종목만 동적으로 거래

이제 원하는 방식으로 거래 유니버스를 세팅할 수 있을 거야!  
추가 질문 있으면 편하게 해줘~ 🚀🔥

##### 나의 말:

Assets.csv 파일도 거래소의 정보를 통해 자동으로 만들 수 없어?

###### ChatGPT의 말:

네, 거래소의 정보를 이용해서 `Assets.csv` 파일을 자동으로 생성할 수 있어!  
**바이낸스 API를 활용하여 실시간으로 자산 정보를 가져와서 `Assets.csv`를 동적으로 업데이트하는 방법**을 설명해 줄게.

---

## 1️⃣ **Python으로 `Assets.csv` 자동 생성**

아래 Python 스크립트는 바이낸스에서 **선물 거래 가능한 모든 자산 목록을 가져와** `Assets.csv` 파일을 생성해.

### **📌 Python 코드 (자동 생성)**

python

복사편집

```
import requests
import csv

# 바이낸스 선물 마켓 심볼 가져오기
url = "https://fapi.binance.com/fapi/v1/exchangeInfo"
response = requests.get(url)
data = response.json()

# 거래 가능한 선물 심볼 목록 추출
symbols = [s["symbol"] for s in data["symbols"] if s["status"] == "TRADING"]

# 가격 및 마진 정보 가져오기
ticker_url = "https://fapi.binance.com/fapi/v1/ticker/price"
spread_url = "https://fapi.binance.com/fapi/v1/ticker/bookTicker"
ticker_data = {t["symbol"]: float(t["price"]) for t in requests.get(ticker_url).json()}
spread_data = {t["symbol"]: (float(t["askPrice"]) - float(t["bidPrice"])) for t in requests.get(spread_url).json()}

# CSV 파일 저장
with open("Assets.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["Name", "Price", "Spread", "Pip", "Multiplier", "Margin", "RollLong", "RollShort", "Symbol"])

    for symbol in symbols:
        price = ticker_data.get(symbol, 0)
        spread = spread_data.get(symbol, 0)
        pip = 0.01 if "USDT" in symbol else 0.0001  # BTC, ETH 등은 0.01, 다른 것은 0.0001
        multiplier = 1  # 기본 계약 크기
        margin = 0.01  # 기본 마진 비율 (조정 가능)
        writer.writerow([symbol, price, spread, pip, multiplier, margin, 0, 0, symbol])

print("Assets.csv 파일이 생성되었습니다!")

```



📌 **설명**

- **바이낸스 API에서 거래 가능한 선물 목록을 가져옴**
- **현재 가격과 스프레드 데이터 포함**
- **CSV 파일을 생성하여 Zorro에서 바로 사용 가능**

---

## 2️⃣ **Zorro에서 `Assets.csv` 자동 로드**

Zorro에서 `Assets.csv`를 자동으로 불러오려면 아래처럼 설정하면 돼.

### **📌 `run()` 함수에서 자동 로드**

c

복사편집
```
function run()
{
    set(PARAMETERS);
    BarPeriod = 1440; // 1일 봉

    assetList("Assets.csv"); // 최신 자산 리스트 로드

    while(asset(loop(Assets))) {
        var atr = ATR(20);
        if(priceClose(0) > priceHigh(1) + atr * 0.5) {
            enterLong();
        }
    }
}

```



📌 **설명**

- **`assetList("Assets.csv")`** → `Assets.csv`의 데이터를 자동으로 불러옴
- **`while(asset(loop(Assets)))`** → 유니버스 내 자산을 하나씩 순회하며 거래

---

## 3️⃣ **자동화 실행 (Windows 스케줄러)**

이 Python 스크립트를 매일 실행하여 `Assets.csv`를 자동으로 업데이트할 수도 있어.  
Windows 작업 스케줄러 또는 Zorro의 `system()` 함수를 사용하면 돼.

### **📌 Zorro에서 Python 스크립트 실행**

c

복사편집

```
if(is(INITRUN)) {
    system("python C:\\path\\to\\generate_assets.py"); // Python 스크립트 실행
}

```



📌 **설명**

- `system()` 함수를 이용하여 `generate_assets.py` 실행
- 최신 `Assets.csv` 파일을 가져온 후 거래 실행

---

## **🚀 최종 정리**

✅ `Python` 스크립트가 **바이낸스 API에서 거래 가능한 선물 리스트를 가져와** `Assets.csv` 생성  
✅ `Zorro`는 `Assets.csv`를 자동으로 로드하여 유니버스를 설정하고 거래  
✅ `Windows 작업 스케줄러` 또는 `Zorro system()`을 이용해 **자동 실행** 가능

이제 매일 **최신 거래 가능 자산을 반영한 자동화된 Zorro 트레이딩 시스템**을 구축할 수 있어!  
추가 질문 있으면 편하게 해줘~ 🚀🔥

##### 나의 말:

여러개의 전략을 위 구조에서 돌리려면 어떤 식으로 짜면 좋을까? 예를들면 변동성돌파 전략, 이평선크로스 전략을 바이낸스의 24시간 거래량 상위 15종목에 대해서 진입 신호가 발생하도록 하고 싶어. 청산은 진입 로직에 따라 각각 적용하는 것도 있고, 함께 사용하는 청산 전략도 있으면 좋겠어.

###### ChatGPT의 말:

여러 개의 전략(예: 변동성 돌파, 이동평균 크로스)을 **바이낸스 24시간 거래량 상위 15개 종목**에 적용하려면, **다중 전략 구조**를 설계해야 해.

## ✅ **전략 실행 구조 설계**

1. **거래 유니버스 동적 설정**
    
    - 바이낸스 API에서 **거래량 상위 15개 종목**을 가져와 `Assets.csv` 자동 생성
    - `assetList()`를 사용하여 해당 종목만 거래
2. **전략 관리 시스템 구축**
    
    - `Strategy_1()`: 변동성 돌파 전략
    - `Strategy_2()`: 이동평균 크로스 전략
    - **각 전략이 독립적으로 신호를 생성하고 트레이드 실행**
3. **청산 전략 적용**
    
    - 개별 청산 로직 적용 (`exitLong()`, `exitShort()`)
    - 공통 청산 전략 (예: ATR 기반 샹들리에 스탑)

---

# **🚀 코드 구조 설계**

c

복사편집

```
#define N_STRATEGIES 2  // 실행할 전략 개수

// 전략별 ID
#define STRATEGY_VOLATILITY 0
#define STRATEGY_MA_CROSS 1

// 거래량 상위 15개 종목 저장
string TopAssets[15];

// 바이낸스 API에서 최신 종목 리스트 불러오기
void LoadTopAssets() {
    char Content[512];
    int handle = file_content("top_assets.txt", Content, 512);
    if (handle) {
        int n = 0;
        char* token = strtok(Content, ",");
        while (token && n < 15) {
            TopAssets[n++] = strf(token);
            token = strtok(NULL, ",");
        }
    }
}

// ✅ 변동성 돌파 전략 (Larry Williams)
void Strategy_Volatility() {
    var atr = ATR(20);
    static var breakoutLevel;

    if (is(NEWDAY)) {
        breakoutLevel = priceHigh(1) + atr * 0.5;
    }

    if (priceClose(0) > breakoutLevel) {
        enterLong();
    }
}

// ✅ 이동평균선 크로스 전략
void Strategy_MA_Cross() {
    vars fastMA = series(SMA(series(priceClose(), 5), 5));  // 5일 이동평균
    vars slowMA = series(SMA(series(priceClose(), 20), 20)); // 20일 이동평균

    if (crossOver(fastMA, slowMA)) {
        enterLong();
    } else if (crossUnder(fastMA, slowMA)) {
        exitLong();
    }
}

// ✅ 공통 청산 전략 (ATR 기반 샹들리에 스탑)
void Common_ExitStrategy() {
    var atr = ATR(20);
    Stop = priceHigh(0) - atr * 3;  // ATR 3배 스탑로스 적용
}

function run() {
    if (is(INITRUN)) {
        LoadTopAssets();  // 거래량 상위 15개 종목 불러오기
    }

    assetList(strf("%s", join(",", TopAssets, 15)));  // 유니버스 설정

    while (asset(loop(Assets))) {  // 각 자산별로 전략 실행
        for (int strategy = 0; strategy < N_STRATEGIES; strategy++) {
            switch (strategy) {
                case STRATEGY_VOLATILITY:
                    Strategy_Volatility();
                    break;
                case STRATEGY_MA_CROSS:
                    Strategy_MA_Cross();
                    break;
            }
        }
        Common_ExitStrategy();  // 공통 청산 전략 실행
    }
}

```


---

# **🚀 설명 및 주요 포인트**

## **✅ 1. 거래 유니버스 자동 설정**

- `LoadTopAssets()`를 사용해 **바이낸스 API에서 24시간 거래량 상위 15개 종목을 가져옴**
- `assetList()`를 통해 **자동으로 유니버스 적용**

## **✅ 2. 다중 전략 관리**

- **각 전략을 함수로 분리 (`Strategy_Volatility()`, `Strategy_MA_Cross()`)**
- **전략 ID (`STRATEGY_VOLATILITY`, `STRATEGY_MA_CROSS`)를 활용하여 실행**
- `for` 문을 사용해 **각 자산별로 여러 전략을 순차적으로 실행**

## **✅ 3. 공통 청산 전략 & 개별 청산 전략**

- **개별 청산 전략** (`exitLong()` 등)은 각 전략 내에서 실행
- **공통 청산 전략** (`Common_ExitStrategy()`)은 모든 전략에 적용 가능 (ATR 기반 샹들리에 스탑)

---

# **🔥 확장 가능성**

이 구조를 사용하면 **손쉽게 새로운 전략을 추가**할 수 있어!  
예를 들어, **RSI 기반 전략을 추가**하고 싶다면?

c

복사편집
```
#define STRATEGY_RSI 2  // 새로운 전략 추가

void Strategy_RSI() {
    vars rsi = series(RSI(14)); // RSI 14일
    if (crossOver(rsi, 30)) enterLong();
    else if (crossUnder(rsi, 70)) exitLong();
}

```


그리고 `run()` 함수에 추가하면 돼!

c

복사편집
```
for (int strategy = 0; strategy < N_STRATEGIES; strategy++) {
    switch (strategy) {
        case STRATEGY_VOLATILITY:
            Strategy_Volatility();
            break;
        case STRATEGY_MA_CROSS:
            Strategy_MA_Cross();
            break;
        case STRATEGY_RSI:
            Strategy_RSI();
            break;
    }
}

```


---

# **🚀 최종 정리**

✅ **바이낸스 API를 활용하여 24시간 거래량 상위 15개 종목을 자동 선택**  
✅ **다중 전략 구조 적용 (변동성 돌파, 이동평균 크로스, 확장 가능)**  
✅ **개별 청산 + 공통 청산 (ATR 샹들리에 스탑) 적용**  
✅ **쉽게 새로운 전략 추가 가능 (RSI, MACD 등)**

이제 원하는 전략을 자유롭게 추가하면서 유니버스 기반 트레이딩을 구현할 수 있어!  
🔥 추가 질문 있으면 편하게 해줘! 🚀