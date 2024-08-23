---
title: "Rottentomatoes 사이트 크롤링"
excerpt: "프로젝트_크롤링"

categories:
  - Project
---

# RottenTomatoes 사이트 크롤링
* Top Box office 영화 데이터 불러오기
* Top Box Ofiice 영화 등급별 비율 시각화
* 관객 리뷰 데이터 시각화


* 라이브러리 설치
```python
# !pip install beautifulsoup4 
# !pip install selenium
# !pip install wordcloud
# !pip install textblob
```
colab 환경이 아닌 경우 !를 빼고 프롬프트에서 입력하기

# Rottentomatoes Top Box Office 영화 데이터 가져오기
```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from bs4 import BeautifulSoup
import pandas as pd

# 브라우저 꺼짐 방지 옵션
chrome_options = Options()
chrome_options.add_experimental_option("detach", True)

# 크롬 드라이버 생성
driver = webdriver.Chrome(options=chrome_options)

# 페이지 로딩이 완료될 때 까지 기다리는 코드
driver.implicitly_wait(3)

# 웹 페이지 열기
driver.get("https://www.rottentomatoes.com/browse/movies_in_theaters/sort:top_box_office?page=5")

# 페이지 로딩이 완료될 때 까지 기다리는 코드
driver.implicitly_wait(5)  # 동적 콘텐츠가 로드되도록 조금 더 기다립니다.

# 페이지 소스 가져오기
html_source = driver.page_source

# 뷰티풀수프를 사용하여 HTML 파싱
soup = BeautifulSoup(html_source, 'html.parser')

# 영화 카드 선택
movie_cards = soup.find_all('div', class_='js-tile-link')

# 영화 제목 추출
titles = [span.get_text(strip=True) for span in soup.find_all('span', class_='p--small')]

# 개봉 날짜 추출
release_dates = [span.get_text(strip=True) for span in soup.find_all('span', class_='smaller')]

# 비평 점수 및 관객 점수 추출
critic_scores = [card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'criticsScore'}).get_text(strip=True) if card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'criticsScore'}) else 'No Score' for card in movie_cards]
audience_scores = [card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'audienceScore'}).get_text(strip=True) if card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'audienceScore'}) else 'No Score' for card in movie_cards]

# 리스트 길이 확인
num_titles = len(titles)
num_release_dates = len(release_dates)
num_critic_scores = len(critic_scores)
num_audience_scores = len(audience_scores)

# 길이를 맞추기 위해 모든 리스트에서 가장 짧은 길이 선택
min_length = min(num_titles, num_release_dates, num_critic_scores, num_audience_scores)

# 모든 리스트에서 동일한 길이로 자르기
titles = titles[:min_length]
release_dates = release_dates[:min_length]
critic_scores = critic_scores[:min_length]
audience_scores = audience_scores[:min_length]

# DataFrame 생성
df = pd.DataFrame({
    '제목': titles,
    '개봉일': release_dates,
    '토마토 미터': critic_scores,
    '관객 점수': audience_scores
})

# 브라우저 닫기
driver.quit()
```

**데이터 프레임 확인**
```python
df
```
이러한 값이 나왔다.

![img](https://lh7-rt.googleusercontent.com/docsz/AD_4nXe9raeIIZLoSB6lh3y6UHwqj58k2MVEjtyRUxYGfeypmm2zQOCmiI-G6SbTEfiVCKOk9Mwq-GmTM3jAUkkbC21ITyeiwRMZCw_BcWTxglpgXLe2HLRMVUbrLbm6tYAvzKyk_JSKERk1eqoLPgTFXT6w7Mr0?key=RugJmL-fnO8f1qLziYUytQ)

# 등급 데이터 가져오기
```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from bs4 import BeautifulSoup
import pandas as pd

# 브라우저 꺼짐 방지 옵션
chrome_options = Options()
chrome_options.add_experimental_option("detach", True)

# 크롬 드라이버 생성
driver = webdriver.Chrome(options=chrome_options)

def get_movie_ratings(url):
    driver.get(url)
    driver.implicitly_wait(5)  # 동적 콘텐츠가 로드되도록 조금 더 기다립니다.
    html_source = driver.page_source
    soup = BeautifulSoup(html_source, 'html.parser')
    
    # 영화 카드 선택
    movie_cards = soup.find_all('div', class_='js-tile-link')
    
    # 영화 제목과 등급 추출
    movie_data = []
    for card in movie_cards:
        title = card.find('span', class_='p--small').get_text(strip=True)
        # 등급 정보를 직접적으로 찾는 것이 어려울 수 있으므로 여기서는 가정합니다.
        # 등급이 별도의 태그로 존재하지 않거나, 등급 정보가 없는 경우 'Not Rated'로 처리합니다.
        rating = card.find('span', class_='rating').get_text(strip=True) if card.find('span', class_='rating') else 'Not Rated'
        movie_data.append({'title': title, 'rating': rating})
    
    return movie_data

# 등급별 URL 리스트
urls = {
    'G': "https://www.rottentomatoes.com/browse/movies_in_theaters/ratings:g~sort:top_box_office",
    'PG': "https://www.rottentomatoes.com/browse/movies_in_theaters/ratings:pg~sort:top_box_office",
    'PG-13': "https://www.rottentomatoes.com/browse/movies_in_theaters/ratings:pg_13~sort:top_box_office",
    'R': "https://www.rottentomatoes.com/browse/movies_in_theaters/ratings:r~sort:top_box_office",
    'NC-17': "https://www.rottentomatoes.com/browse/movies_in_theaters/ratings:nc_17~sort:top_box_office",
    'NR': "https://www.rottentomatoes.com/browse/movies_in_theaters/ratings:nr~sort:top_box_office?page=3"
}

# 영화 제목과 등급 정보 딕셔너리 생성
rating_dict = {}
for rating, url in urls.items():
    movie_data = get_movie_ratings(url)
    for movie in movie_data:
        rating_dict[movie['title']] = rating

# 원본 영화 데이터프레임 가져오기
driver.get("https://www.rottentomatoes.com/browse/movies_in_theaters/sort:top_box_office?page=5")
driver.implicitly_wait(5)
html_source = driver.page_source
soup = BeautifulSoup(html_source, 'html.parser')

movie_cards = soup.find_all('div', class_='js-tile-link')
titles = [span.get_text(strip=True) for span in soup.find_all('span', class_='p--small')]
release_dates = [span.get_text(strip=True) for span in soup.find_all('span', class_='smaller')]
critic_scores = [card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'criticsScore'}).get_text(strip=True) if card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'criticsScore'}) else 'No Score' for card in movie_cards]
audience_scores = [card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'audienceScore'}).get_text(strip=True) if card.find('rt-text', {'context': 'label', 'size': '1', 'slot': 'audienceScore'}) else 'No Score' for card in movie_cards]

# 리스트 길이 확인
num_titles = len(titles)
num_release_dates = len(release_dates)
num_critic_scores = len(critic_scores)
num_audience_scores = len(audience_scores)

# 길이를 맞추기 위해 모든 리스트에서 가장 짧은 길이 선택
min_length = min(num_titles, num_release_dates, num_critic_scores, num_audience_scores)

# 모든 리스트에서 동일한 길이로 자르기
titles = titles[:min_length]
release_dates = release_dates[:min_length]
critic_scores = critic_scores[:min_length]
audience_scores = audience_scores[:min_length]

# DataFrame 생성
df = pd.DataFrame({
    '제목': titles,
    '개봉일': release_dates,
    '토마토 미터': critic_scores,
    '관객 점수': audience_scores
})

# 영화 제목과 등급 정보를 비교하여 등급 추가
df['등급'] = df['제목'].map(rating_dict).fillna('NR')

# 브라우저 닫기
driver.quit()
```

**데이터 프레임 확인**
```python
df.head()
```
등급 부분이 추가됨

![img](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeA3ELgt9JFklFhnGOhcHcaEng3ZEDWpyT4DtiLPCDpGDNo7EXxS7h-QQ9ukyFQ8dh1SYT08-OykiHUPEdq6nsLx15Ier3fsShGfrCCteTLH1_KRMgei0DP_l5LsK12C4LSQfU3IjvaSknrgs2_HHlK_qiA?key=RugJmL-fnO8f1qLziYUytQ)

* 날짜 형식 바꾸기
현재 날짜 데이터는 object로 되어있다. <br>
이 데이터 타입을 변경해보기

```python
def parse_release_date(date_str):
    # "Opened" 또는 "Opens" 부분 제거
    if date_str.startswith("Opened "):
        date_str = date_str.replace("Opened ", "")
    elif date_str.startswith("Opens "):
        date_str = date_str.replace("Opens ", "")
    # 문자열을 datetime 형식으로 변환
    return pd.to_datetime(date_str, format='%b %d, %Y', errors='coerce')

# DataFrame의 'Release Date' 열을 변환
df['개봉일'] = df['개봉일'].apply(parse_release_date)

# 결과 확인
df
```
![img](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeRfTXKW7dNnmqp1KaR1i2RXQaI80EMyrevghSlmhs7A-1JGoaj4MpUfdcAR8_dYCK3BflA3t8-e3RnBSx-LGIDNYFh4iGOdY--MOMyJU2scsqAJMmjAOqXajrLSqIaob7nYOwUFeT19ve1KWI1iQLmAAk?key=RugJmL-fnO8f1qLziYUytQ)

* 중복 영화 제목 항목 제거
```python
df = df.drop_duplicates(subset='제목')
df
```
![img](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcOkXeF52COY7YMjrU7o6G5Uqbf4vpEhl5ltwxYOlXgHWWCiTtPzNgaoLUpH2dR7bFhnkk14HVW9GbIzD-x15sDdcKyyfg6MWGzo3ZXAa1RYuO4g2fbYfhXbcIx3igKEW_0mFFZtEcHdBiwSvfXGP0VEy03?key=RugJmL-fnO8f1qLziYUytQ)

* 토마토 미터 or 관객 점수 데이터가 없는 항목 제거
```python
filtered_df = df[~((df['토마토 미터'] == 'No Score') | (df['관객 점수'] == 'No Score'))]
# DataFrame 출력
filtered_df
```
![img](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfQZs7d7KsMQm8T1YMofgQM5y3O7R6Av2Ze4Ahu8mUzRL_zY9nJBj7I22y02-EWNFTtRHwN65itHjOqHi_Kw2IytEuzgZe7iW1lMPuN5xuI_QvykckT7Npv65Z6mXDiWnWlvxj2OlTaklYiv6xegJ18lOM2?key=RugJmL-fnO8f1qLziYUytQ)

* 영화 등급 카운트
```python
filtered_df['등급'].value_counts()
```
