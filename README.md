## 빅데이터 - 부동산 데이터 분석 프로젝트!

---

### ✅ 데이터 크롤링

- requests 모듈로 json 처리하는 방식으로 네이버 웹 사이트를 크롤링한다.

<img width="1512" alt="1" src="https://user-images.githubusercontent.com/62542933/206828793-cb5759a7-0946-4ec0-beb5-0a24efb13e52.png">

```
  import requests
  import json
  down_url = 'https://new.land.naver.com/api/complexes/19127'
  r = requests.get(down_url,data={"sameAddressGroup":"false"},headers={
      "Accept-Encoding": "gzip, deflate, br",
      "authorization":"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IlJFQUxFU1RBVEUiLCJpYXQiOjE2NjM5MzE0MzksImV4cCI6MTY2Mzk0MjIzOX0.QF-Frm_t9I5yFolbTtDfl-kx4EQaBa3i57syd8dt-LU",
      "Host": "new.land.naver.com",
      "Referer": "https://new.land.naver.com/complexes/22627?ms=37.513603,127.0842597,16&a=APT:ABYG:JGC&e=RETAIL",
      "sec-ch-ua":"Chromium\";v=\"104\", \" Not A;Brand\";v=\"99\", \"Google Chrome\";v=\"104",
      "Sec-Fetch-Dest": "empty",
      "sec-ch-ua-mobile": "?0",
      "sec-ch-ua-platform": "macOS",
      "Sec-Fetch-Dest": "empty",
      "Sec-Fetch-Mode": "cors",
      "Sec-Fetch-Site": "same-origin",
      "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36"
  })
  r.encoding = "utf-8-sig"
  # temp에 F12 -> Network -> Preview 에 보이는 정보를 넣는다.
  temp=json.loads(r.text)
  print(temp["complexDetail"])
```

---

### ✅ 데이터 분석

<img width="757" alt="image" src="https://user-images.githubusercontent.com/62542933/206828997-167ea0f8-7d19-4c24-bdd0-9632df0b92ff.png">

#### 데이터 전처리

- 추출 : Pandas

```
  %reset -f
  import pandas as pd

  # 서울시 송파구.csv파일명을 sonpa_apt로 바꾸어 작업해야 한다.
  df = pd.read_csv('/content/songpa_apt.csv',encoding="CP949")

  # 최근 가격 변동이 존재하는 아파트
  df_test_price = df[:]
  df_test = df_test_price.dropna(how='any', subset=['최근가격변동'], axis=0)
  # print(df_test.head())

  # 매매호가가 NaN인 데이터들 제거
  df_change_price = df_test_price.dropna(how='any', subset=['매매호가'], axis=0)


  # 가격이 NaN인 데이터 제거
  df_price_nan = df_change_price.dropna(how='any', subset=['가격'], axis=0)

  df_final = df_price_nan[:]
```

#### 데이터 시각화

- folium

<img width="1089" alt="24" src="https://user-images.githubusercontent.com/62542933/206899386-7b35e22a-3378-4218-a097-b7521e9ae427.png">

```
import pandas as pd
import folium

# 지도 만들기
smap = folium.Map(location=[37.5144533,127.1059047], tiles='Stamen Terrain', zoom_start=15)

# 아파트 이름과 가격 정보를 넣어 핀을 찍는다
for name,price , lat, lng in zip(df_final.아파트명,df_final.매매호가, df_final.latitude, df_final.longitude):
  folium.Marker([lat,lng], popup=[name,price]).add_to(smap)

smap
```

<img width="1090" alt="41" src="https://user-images.githubusercontent.com/62542933/206899440-aa8bddda-3fa1-47dd-8e26-a37a1684a849.png">

```
import folium
# 송파구청을 시작 위치로 설정한다.
smap = folium.Map(location=[37.5144533,127.1059047],  zoom_start=15)

for name,rate , lat, lng in zip(df_final.아파트명,df_final.songpa_change_rate, df_final.latitude, df_final.longitude):
  folium.CircleMarker([lat,lng],
                      radius=10,
                      color='brown',
                      fill=True,
                      fill_color='coral',
                      fill_opacity=0.7,
                      popup=[name,rate]).add_to(smap)
smap
```

- seaborn

<img width="1099" alt="32" src="https://user-images.githubusercontent.com/62542933/206899543-f0f9081a-aeec-4619-aadc-e5c0eff140c3.png">

```
import seaborn as sns
# 결측치 확인
print(df['초등학교_학군정보'].isna().sum())

# 학군 정보에 해당하는 아파트 개수 그래프
df_school = df.dropna(how='any', subset=['초등학교_학군정보'], axis=0)
sns.set(rc={'figure.figsize':(60,8)},font="NanumBarunGothic")
sns.countplot(data=df_school, x='초등학교_학군정보')
```

<img width="1089" alt="35" src="https://user-images.githubusercontent.com/62542933/206899558-b7d4fec7-d518-4744-a29c-3c29049a90a8.png">

```
import seaborn as sns
# lineplot 그래프
sns.set_style('darkgrid')
sns.set(rc={'figure.figsize':(18,8)},font="NanumBarunGothic")
sns.lineplot(data=grouped_school_establish_price_mean, x='초등학교_설립정보',y='가격', marker='o', color='r', linestyle=':')
```

<img width="1102" alt="45" src="https://user-images.githubusercontent.com/62542933/206899572-e148bb54-39ab-480b-94fd-e39ef2f7a9a9.png">

```
import seaborn as sns

# 학군에 따른 가격 분포
df_school_zone=df_price_nan[:]

# 학군이 없는 데이터들을 제거
df_school_zone = df_school_zone.dropna(how='any', subset=['최근가격변동'], axis=0)


# group을 이용하여 학군으로 그루핑
grouped_school = df_school_zone.groupby(['초등학교_학군정보'])

# 학군에 해당하는 아파트들의 평균 가격
grouped_school_price_mean = grouped_school.mean()

# lineplot 그래프
sns.set_style('darkgrid')
sns.set(rc={'figure.figsize':(70,8)},font="NanumBarunGothic")
sns.lineplot(data=grouped_school_price_mean, x='초등학교_학군정보',y='가격', marker='o', color='r', linestyle=':')
```

- mataplotlib

<img width="502" alt="43" src="https://user-images.githubusercontent.com/62542933/206899612-f5c1c10e-7b39-4935-bba5-c9007ca17ee6.png">

```
import matplotlib.pyplot as plt

# 가격이 하락하는 아파트의 개수
price_change_minus=sum(df_price_nan.최근가격변동 < 0)
# 가격 변동이 없는 아파트의 개수
price_change_zero=sum(df_price_nan.최근가격변동 == 0)
# 가격이 상승하는 아파트의 개수
price_change_plus=sum(df_price_nan.최근가격변동 > 0)

ratio=[price_change_minus, price_change_zero, price_change_plus]
labels=['price drop', 'no price change', 'price rice']
# labels=['가격 하락', '가격변동없음', '가격 상승']
explode = [0, 0.1, 0]
colors=['lightsalmon','tomato','oldlace']
plt.pie(ratio, labels=labels,colors=colors, explode=explode, autopct='%.1f%%')

plt.show()
```

---
