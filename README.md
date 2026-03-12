# 박효인의 portfolio 입니다.
# Intro
Python, SQL을 활용한 다양한 프로젝트 경험을 기반으로 데이터 엔지니어링과 데이터 분석 역량을 길러왔습니다. 데이터 수집 및 전처리, 구조화하는 데이터파이프라인을 기반으로 분석에 활용 가능한 데이터를 마련하여 의사결정에 도움을 주는 일에 관심이 있습니다. 데이터 기반으로 직접 의사결정을 해 기업의 성장에 기여하고 싶은 마음 또한 가지고 있습니다. 이 GitHub에는 제가 지금까지 수행한 데이터처리, 분석, 모델링 등과 관련된 코드들을 정리하고 있습니다.

# Project 1
- 프로젝트명 : 제주시 전기차 충전소 최적입지 선정모델
- 프로젝트 목적 : 전기차 수요가 많은 제주시의 충전인프라 부족문제를 해결하기 위해 추가 충전소는 어디에 설치하는 것이 가장 적합할지에 대한 답을 내놓기 위함
- 기간 : 2024.10 ~ 2024.12
- 참여인원 : 4명
- 핵심역할 : 데이터 수집 및 전처리, 모델 구현
- 기여도 : 90%
- 프로그래밍 언어 : Python
- 주요 패키지 및 라이브러리 : pandas, folium, haversine, geopandas, matplotlib, StandardScaler, scipy, numpy, pulp
- [전체코드 링크](https://github.com/hyoin28/project1/blob/main/%EC%A0%9C%EC%A3%BC%EC%8B%9C%20%EC%A0%84%EA%B8%B0%EC%B0%A8%EC%B6%A9%EC%A0%84%EC%86%8C%20%EC%B5%9C%EC%A0%81%EC%9E%85%EC%A7%80%20%EC%84%A0%EC%A0%95%20%EB%AA%A8%EB%8D%B8.ipynb)

## (1) 데이터 가공 및 정제
- 필요한 데이터만 추출하기 위해 pandas의 여러 메서드 사용
<br>

<코드 일부예시>
```python
##종속변수-각 충전소의 충전량
#충전량 데이터 불러오기
df_y = pd.read_csv('한국환경공단_환경부 전기차 충전기 충전량 상세정보_20231030.csv', encoding='cp949')
df_y = df_y[df_y['충전소 주소'].str.contains('제주시')] #제주시 데이터만 뽑아오기
#구글 스프레드시트를 사용해 충전소의 주소를 위도, 경도로 변환한 csv파일 불러오기
df_l = pd.read_csv('제주시 충전소 주소(이용률데이터).csv')
df_l = df_l.drop(columns=['충전소 주소'])

#기존 충전량 데이터에 위도, 경도 추가
df_y = pd.concat([df_y, df_l], axis=1)

#충전소명이 겹치는 건 그룹화해서 충전횟수, 충전량 값 합치기
df_y_grouped = df_y.groupby('충전소명')[['2023년 01월 충전횟수', '2023년 01월 충전량', '2023년 02월 충전횟수', '2023년 02월 충전량', '2023년 03월 충전횟수', '2023년 03월 충전량', '2023년 04월 충전횟수', '2023년 04월 충전량', '2023년 05월 충전횟수', '2023년 05월 충전량', '2023년 06월 충전횟수', '2023년 06월 충전량', '2023년 07월 충전횟수', '2023년 07월 충전량', '2023년 08월 충전횟수', '2023년 08월 충전량', '2023년 09월 충전횟수', '2023년 09월 충전량']].sum()
```
- 데이터 가공을 위해 haversine 공식 사용
<br>

<코드 일부예시>
```python
#하버사인 공식을 사용하여 두 좌표 사이의 거리를 계산하는 함수
def haversine(coord1, coord2):
    R = 6371.0  # 지구 반지름 (단위: km)

    lat1, lon1 = coord1
    lat2, lon2 = coord2

    #위도와 경도를 라디안 단위로 변환
    lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])

    #거리 차이 계산
    dlat = lat2 - lat1
    dlon = lon2 - lon1

    #하버사인 공식 적용
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    c = 2 * atan2(sqrt(a), sqrt(1-a))

    #거리 계산(km를 m로 변환)
    distance = R * c * 1000
    return distance

#격자를 기준으로 근방의 충전량 좌표를 뽑아냄
close_coordinates = []
for coord1 in coordinates_1:
    for coord2 in coordinates_2:
        if haversine(coord1, coord2) <= threshold: #두 좌표의 거리가 5km 이하이면 좌표 추가
            close_coordinates.append((coord1, coord2))
```
- 가공한 데이터를 StandardScaler()로 표준화
```python 
scaler = StandardScaler()
df_total_scaled = scaler.fit_transform(df_total)
df_total_finall = pd.DataFrame(df_total_scaled, index=df_total.index, columns=df_total.columns)
```
## (2) 데이터 유효성 검증을 위한 시각화
- folium 라이브러리를 사용해 여러 데이터를 지도에 레이어링하여 표시함
<br>

<코드 일부예시>
```python
#교통량 좌표 찍기
# 지도 초기화 (중앙점으로 위도, 경도를 설정)
map_center = [df_jeju['centroid_lat'].mean(), df_jeju['centroid_lon'].mean()]  # 평균 위도, 경도로 중앙 설정
mymap = fl.Map(location=map_center, zoom_start=10)

for i, j in zip(df_total['centroid_lat'], df_total['centroid_lon']):
  fl.Circle(location=[i, j], radius=1, color='blue', fill=True, fill_color='blue', fill_opacity=0.7).add_to(mymap)

for i, j in zip(df_t['XCODE'], df_t['YCODE']):
    fl.Circle(location=[i,j], radius=300, color='red').add_to(mymap)

mymap
```
<시각화 출력결과 예시>
![image](img/충전소위치&교통량 데이터 레이어링.png)

