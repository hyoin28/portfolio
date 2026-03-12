# 🗒️ 박효인의 portfolio 입니다.
# 📌 Intro
Python, SQL을 활용한 다양한 프로젝트 경험을 기반으로 데이터 엔지니어링과 데이터 분석 역량을 길러왔습니다. 데이터 수집 및 전처리, 구조화하는 데이터파이프라인을 기반으로 분석에 활용 가능한 데이터를 마련하여 의사결정에 도움을 주는 일에 관심이 있습니다. 또한 데이터 기반 의사결정을 통해 기업 성장에 기여하는 것에 흥미를 가지고 있습니다. 이 GitHub에는 제가 지금까지 수행한 데이터처리, 분석, 모델링 등과 관련된 프로젝트 코드들이 정리되어 있습니다.

# 📌 My Skill
✔️ **Programming Language** : Python, SQL

✔️ **RDBMS** : MySQL

✔️ **Library** : pandas, numpy, sklearn, folium, pytorch, selenium 등

✔️ **커뮤니케이션 및 협업 능력** : 다양한 팀프로젝트에서 항상 책임감있고 열려있는 태도로 프로젝트를 이끄는 리더의 면모를 성장시켜왔습니다.

# 💡 Project 1
- **프로젝트명** : 제주시 전기차 충전소 최적입지 선정모델
- **프로젝트 목적** : 전기차 수요가 많은 제주시의 충전인프라 부족문제를 해결하기 위해 추가 충전소는 어디에 설치하는 것이 가장 적합할지에 대한 답을 내놓기 위함
- **기간** : 2024.10 ~ 2024.12
- **참여인원** : 4명
- **핵심역할** : 데이터 수집 및 전처리, 모델 구현
- **기여도** : 90%
- **프로그래밍 언어** : Python
- **주요 패키지 및 라이브러리** : pandas, folium, haversine, geopandas, matplotlib, StandardScaler, scipy, numpy, pulp
- [**사용 데이터**](https://github.com/hyoin28/project1/tree/main/data)
- [**전체코드 링크**](https://github.com/hyoin28/project1/blob/main/%EC%A0%9C%EC%A3%BC%EC%8B%9C%20%EC%A0%84%EA%B8%B0%EC%B0%A8%EC%B6%A9%EC%A0%84%EC%86%8C%20%EC%B5%9C%EC%A0%81%EC%9E%85%EC%A7%80%20%EC%84%A0%EC%A0%95%20%EB%AA%A8%EB%8D%B8.ipynb)

## (1) 데이터 가공 및 정제
- 필요한 데이터만 추출하기 위해 **pandas**의 여러 메서드 사용

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
<br>

- 데이터 가공을 위해 **haversine 공식** 사용

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
<br>

- 가공한 데이터를 **StandardScaler()**로 표준화
```python 
scaler = StandardScaler()
df_total_scaled = scaler.fit_transform(df_total)
df_total_finall = pd.DataFrame(df_total_scaled, index=df_total.index, columns=df_total.columns)
```
## (2) 데이터 유효성 검증을 위한 시각화
- **folium** 라이브러리를 사용해 여러 데이터를 지도에 레이어링하여 표시함

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
<시각화 출력결과 예시> - 전기차 충전소의 위치(검은색 점)와 교통량(히트맵)이 레이어링돼 표시된 모습

<img width="500" height="250" alt="Image" src="https://github.com/user-attachments/assets/b5a6180a-d7e1-41eb-8714-23d4cd5ac6e4" />

## (3) 최적입지 선정 모델 구현 - **MCLP(Maximal Covering Location Problem) 알고리즘** 사용 : 제한된 시설물의 개수로 수요량을 최대화하는 입지를 선정하는 알고리즘
<br>

<코드 일부예시>
```python
def mclp_with_pulp(points,P,S,sites,demand_weight):
    print('----- Configurations -----')
    print('  I %g' % points.shape[0])
    print('  P %g' % P)
    print('  S %g' % S)

    J = sites.shape[0] #후보지 배열의 행의 개수를 불러옴 = 후보지의 개수
    I = points.shape[0] #수요지 배열의 행의 개수를 불러옴 = 수요지의 개수

    coverage_matrix = np.zeros((I,J))
    for i in range(I):
        for j in range(J):
            coverage_matrix[i,j] = geodesic(points[i],sites[j]).km #수요지와 후보지 간의 거리 계산

    coverage_matrix = (coverage_matrix <= S).astype(int)

    #모델 정의 : LpProblem을 사용해 최대화 문제를 정의, 목표는 커버된 수요지의 가중치 합을 최대화 하는 것
    model = LpProblem(name="MCLP", sense=LpMaximize)

    # Add variables
    x = [LpVariable(f"x{j}", cat="Binary") for j in range(J)] #후보지 j가 선택되었는지 여부를 나타내는 이진변수
    y = [LpVariable(f"y{i}", cat="Binary") for i in range(I)] #수요지 i가 커버되었는지 여부를 나타내는 이진변수

    # 목적함수
    model += lpSum(demand_weight[i] * y[i] for i in range(I))

    # 제약조건
    model += lpSum(x) == P # K개의 후보지를 선택하는 제약

    model.solve()
```
- 수요를 최대화하는 20개의 최적입지가 위/경도로 출력됨

## (4) 모델검증 및 분석
- 독립변수에 따른 모델 출력결과를 분석하여, 본 프로젝트 상에서 최대한 많은 변수를 사용하여 모델을 실행하는 것이 가장 좋은 결과를 냄을 검증함
<img width="600" height="500" alt="Image" src="https://github.com/user-attachments/assets/7d9c823a-bad9-403f-8839-0e9e7e54d8fd" />
<br>

- 가장 효율적인 최적입지 선정개수를 알기 위해 입지개수에 따른 모델 결과수치를 분석함
<img width="600" height="450" alt="Image" src="https://github.com/user-attachments/assets/69ab5c29-aab8-4716-be6f-cbe0ce6b898f" />

# 💡 Project 2
- **프로젝트명** : 애니메이션 OTT사이트(가상) DB구현
- **프로젝트 목적** : 기업의 요구사항을 이해하여 알맞은 구조로 DB를 생성하고 데이터를 적재하는 것을 목표로 함
- **기간** : 2024.05 ~ 2024.06
- **참여인원** : 4명
- **핵심역할** : ER-Diagram 및 Relational DB schema 도식화, DB 생성(테이블 생성, 데이터 삽입), 프로젝트 보고서 작성
- **기여도** : 70%
- **프로그래밍 언어** : Python, SQL
- **주요 패키지 및 라이브러리** : mysql.connector, pandas, csv, StringIO
- [**전체코드 링크**](https://github.com/hyoin28/project2/blob/main/%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98ott%EC%82%AC%EC%9D%B4%ED%8A%B8%20DB%EA%B5%AC%ED%98%84.ipynb)

## (1) 프로젝트에서 주어진 기업의 요구사항을 만족시키는 DB 구조 도식화
- 테이블의 속성, 테이블간의 관계, 제약조건 등을 고려

<**ER Diagram**>

<img width="500" height="370" alt="Image" src="https://github.com/user-attachments/assets/adafd384-2c5a-4d27-9b10-fdf9e126d3a4" />
<br>

<**Relational DB schema**>

<img width="372" height="480" alt="Image" src="https://github.com/user-attachments/assets/a4922a97-85dc-4a2b-944b-e70828497b60" />

## (2) mysql.connector로 Python과 MySQL을 연결하여 DataBase 생성 - CREATE DATABASE
```python
def requirement1(host, user, password):
    cnx = mysql.connector.connect(host=host, user=user, password=password)
    cursor = cnx.cursor()
    cursor.execute('SET GLOBAL innodb_buffer_pool_size=2*1024*1024*1024;')
    
    cursor.execute('DROP DATABASE IF EXISTS DMA_team02;')
    cursor.execute('CREATE DATABASE IF NOT EXISTS DMA_team02;')
    cursor.execute('USE DMA_team02;')
    cnx.commit()
```
## (3) DB에 Table 생성 - CREATE TABLE

<코드 일부예시>
```python
cursor.execute('''
    CREATE TABLE USER(
    id INT(11) NOT NULL,
    Name VARCHAR(255),
    PRIMARY KEY (id)
    );
    ''')
```
## (4) 데이터 전처리 및 삽입 - INSERT
- csv파일의 데이터를 불러와 삽입
- 콤마(,) 기준 데이터이므로 split() 사용해 분리
- 문자열 'null' 을 실제 null로 변환하기 위해 replace() 사용

<코드 일부예시>
```python
filepath = directory + '/' + 'user.csv'
    
    with open(filepath, 'r', encoding='utf-8') as csv_data:
        for row in csv_data.readlines()[1:]:
            row = row.strip().split(',')

            for idx, data in enumerate(row):
                if data == '':
                    row[idx] = 'null'
                if idx == 0:
                    row[idx] = int(data)
            row = tuple(row)
            sql = 'INSERT INTO USER VALUES {};'.format(row)
            sql = sql.replace('\'null\'', 'null')
            cursor.execute(sql)
```

- 속성값이 'a, b, c' 이러한 형태로 저장돼있을 경우, split(',')로 인해 값 하나가 콤마를 기준으로 분리되므로 csv, StringIO 라이브러리를 사용해 데이터를 삽입함
- 필요없는 속성은 pop() 함수로 삭제
```python
 with open(filepath, 'r', encoding='utf-8') as csv_data:
        for row in csv_data.readlines()[1:]:
            import csv
            from io import StringIO
            f = StringIO(row)
            reader = csv.reader(f)
            for r in reader:
                r.pop(3) #'Primiered' 컬럼은 파생정보로서 관리돼야하기 때문에 삭제
                for idx, data in enumerate(r):
                    if data == '':
                        r[idx] = 'null'
                    if idx in [0,5]:
                        r[idx] = int(data)
                r = tuple(r)
                sql = 'INSERT INTO ANIME(id, Name, Aired, Source, studio_id) VALUES ({},"{}","{}","{}",{});'.format(r[0],r[1],r[2],r[3],r[4])
                sql = sql.replace('\'null\'', 'null')
                cursor.execute(sql)     
```

## (5) View 생성을 위한 query 작성
- 사용자가 해당 사이트에서 알고싶어 할만한 view를 구상함
- view를 통해 향후 데이터분석에 사용가능함

View 1 <**사용자 n명 이상이 시청하고, 시청한 사용자가 매긴 평균평점이 m점 이상인 장르별 애니메이션**>
```python
query = '''
    CREATE VIEW AnimeGenre AS
    SELECT
        G.Name AS GenreName,
        A.id AS AnimeId,
        A.Name AS AnimeName,
        AVG(AUR.rating) AS AverageRating,
        COUNT(AUR.user_id) AS viewers
    FROM 
        ANIME AS A
    JOIN 
        ANIME_GENRE AS AG ON A.id = AG.anime_id
    JOIN 
        ANIME_USER_RATING AS AUR ON A.id = AUR.anime_id
    JOIN 
        GENRE AS G ON AG.genre_id = G.id
    GROUP BY 
        AG.genre_id, A.id, A.Name
    HAVING
        viewers >= %(n)s AND AverageRating >= %(m)s;
    '''
```

View 2 <**평점이 8점 이상인 애니를 제작한 스튜디오**>
```python
query = '''
    CREATE VIEW HighRateAnime AS
    SELECT
        S.Name AS StudioName,
        A.Name AS AnimeName,
        R.Rating AS Rating
    FROM STUDIO AS S
    JOIN ANIME AS A ON S.id = A.studio_id
    JOIN ANIME_USER_RATING AS R ON A.id = R.anime_id
    WHERE R.rating >= 8;
    '''
```
# 💡 Project 3
- **프로젝트명** : CGV 현재상영작 정보 DB 구현
- **프로젝트 목적** : 2025년 애니메이션 영화의 흥행으로 다시 부흥했던 영화산업을 보며 사람들이 어떤 영화를 선호하는지 데이터기반으로 정확히 알아보기 위함
- **기간** : 2025.01.04 ~ 2025.01.10
- **참여인원** : 1명(개인프로젝트)
- **프로그래밍 언어** : Python, SQL
- **주요 패키지 및 라이브러리** : selenium, pandas, sqlalchemy
- [**전체코드 링크**](https://github.com/hyoin28/project3/blob/main/cgv_%ED%81%AC%EB%A1%A4%EB%A7%81.ipynb)

## (1) CGV 홈페이지상의 현재상영작 세부정보들을 동적 웹크롤링하여 데이터 수집 - Selenium
- 추출데이터 : 영화 제목, 예매순위, 예매율, 누적관객수, 에그지수
- 총 46개의 영화 정보가 수집됨

<코드 일부예시>
```python
#각 영화의 데이터 추출
data = [] #데이터들을 담을 빈 리스트 생성

for i in range(m_count) :
    boxes = driver.find_elements(By.CLASS_NAME, 'bestChartList_chartItem__bJ0PK')  #각 영화의 정보를 담고있는 박스들(35개의 박스들)
    box = boxes[i] #각 영화 박스
    btn = box.find_element(By.CSS_SELECTOR, 'button.btn.btn-md.line-gray') #box안의 '상세보기' 버튼 찾기
    driver.execute_script(
        "arguments[0].scrollIntoView({block: 'center'});", btn
    ) #버튼을 화면 중앙으로 스크롤하여 위치시킴(클릭실패 예방)
    time.sleep(0.5) #스크롤 후 대기
    old_url = driver.current_url #클릭 전 현재 url저장
    driver.execute_script("arguments[0].click();", btn) #JavaScript로 강제클릭
    wait.until(EC.url_changes(old_url)) #현재 url과 다른 url이 될때 까지 기다림

    #영화제목
    title = t(By.CLASS_NAME, 'cnms01020_movieName__XiCWf')
    #영화 장르
    genre = movie_genre()
    #예매율 순위
    rank = list(movie_info().keys())[0]
    #예매율
    ratio = list(movie_info().values())[0]
    #누적관객수
    aud = movie_info()['누적관객수']
    #에그지수
    egg = movie_info()['에그지수']
    
    data.append([title, genre, rank, ratio, aud, egg])
```

## (2) 추출한 데이터 리스트를 데이터프레임으로 변환후 전처리
<전처리 과정>
- 예매 순위에서 숫자만 추출하기
- 예매율/에그지수의 '%' 단위 없애기
- 누적관객수에서 숫자만 남기고 '만' 단위로 통일하기

<코드 일부예시>
```python
#예매율, 에그지수 '%' 단위 없애기
movie_df['reservation_rate'] = list((movie_df['reservation_rate'].str.replace('%', '')).astype(float))

lst_egg =[]
for e in movie_df['egg']:
    if e == '?' :
        lst_egg.append(0)
    else:
        lst_egg.append(int(e.replace('%', '')))
movie_df['egg'] = lst_egg
```

## (3) MySQL DB 생성 및 데이터 적재
- SQLAlchemy 방식 사용
```python
#MySQL 서버 연결
engine = create_engine(
    f"mysql+pymysql://{user}:{password}@{host}?charset=utf8mb4",
    echo=True
)
#DB 생성
with engine.connect() as conn:
    conn.execute(text('DROP DATABASE IF EXISTS CGV'))
    conn.execute(text('CREATE DATABASE CGV'))
    conn.commit()

#CGV DB로 다시 연결
engine = create_engine(
    f"mysql+pymysql://{user}:{password}@{host}/{db_name}?charset=utf8mb4"
)
#테이블 생성
create_table_sql = """
CREATE TABLE IF NOT EXISTS CURRENT_MOVIE(
    title VARCHAR(100),
    genre VARCHAR(100),
    reservation_rank INT,
    reservation_rate FLOAT,
    aud_count VARCHAR(100),
    egg INT
    );
"""

with engine.connect() as conn:
    conn.execute(text(create_table_sql))
    conn.commit()

#데이터 삽입
df = pd.read_csv('current_movie_data.csv', encoding='utf-8-sig')
df.to_sql(name='current_movie',
          con=engine, #DB연결객체, 내부적으로 mysql에 접속
          if_exists='append', #테이블이 이미 존재할 경우 기존데이터 유지 후 뒤에 추가
          index=False
          )
```

## (4) 데이터분석 - 장르 선호도 알아보기
<query 1>

```sql
SELECT genre, COUNT(*) as count, SUM(aud_count) as total_audience
FROM cgv.current_movie
GROUP BY genre
ORDER BY total_audience DESC;
```
<img width="384" height="174" alt="Image" src="https://github.com/user-attachments/assets/38890756-2e76-4bd9-af97-3f776385d12f" />

- 누적관객수 기준 장르의 개수를 세는 쿼리를 작성
- 출력결과 실제로 '애니메이션' 장르의 누적관객수가 가장 많은 것을 확인할 수 있음
- '에니메이션' 장르의 영화개수가 많다는 것을 감안하더라도 '드라마' 장르와 비교하면 훨씬 선호도가 높다는 것을 알 수 있음
- 2위 'SF, 액션, 어드벤처' 장르는 1개임에도 높은 관객수치를 기록했는데, 이는 특정 유명영화라는 예외적인 경우임

<query 2>

```sql
SELECT genre, COUNT(*) as genre_count
FROM (SELECT title, genre, reservation_rank
FROM current_movie
ORDER BY reservation_rank
LIMIT 10) A
GROUP BY genre
ORDER BY genre_count DESC;
```
<img width="264" height="133" alt="Image" src="https://github.com/user-attachments/assets/8641fa5a-7e18-4356-944c-70f0b9fbffcc" />

- '애니메이션' 장르 선호도가 높다는 것을 더 정확히 입증하기 위해 현재상영작 top10에서 장르별 개수를 출력하는 쿼리를 작성함
- '애니메이션' 장르가 4개로 가장 많은 것을 확인할 수 있음



