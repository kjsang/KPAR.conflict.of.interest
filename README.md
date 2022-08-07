# Title
공직자의 이해충돌방지를 위한 정책결정과정 분석: 텍스트마이닝을 활용한 다중흐름모형의 적용   
A Study of the Policy-Making Process for Preventing Conflict of Interest among Public Servants: The Application of the Multiple Streams Framework Using Text Mining   
   
## 0. Abstract & Keywords
### 0.1. Korean
본 연구는 다중흐름모형과 텍스트마이닝 기법을 결합하여 공직자의 이해충돌방지를 위한 정책결정과정을 분석하는 데 목적이 있다. 이를 위해 본 연구는 웹 크롤링을 통해 신문기사를 수집하여 단어빈도분석, 가중로그승산비 분석, 토픽 모델링 분석 등을 수행하였다. 분석결과를 바탕으로 각 시기별로 나타난 흐름을 파악하고 정책결정과정에 영향을 미친 주요 행위자와 사건, 문제 등을 살펴보았다. 분석 결과, 먼저 초점사건으로 조성된 국가적 분위기와 대중의 인식은 부패인식지수 등 문제의 흐름에서 나타난 지표와 괴리를 보였다. 둘째, 정부 내 의제설정과정에서 정책선도가의 역할은 분명했지만 입법부 내의 의제설정 및 정책결정 과정에서의 영향력은 미미하였다. 다중흐름모형을 활용한 정책연구 중 비정형 데이터의 정량적 분석을 활용한 사례는 소수에 불과하다. 본 연구는 통계언어 R을 활용한 텍스트마이닝 기법을 다중흐름모형에 적용하여 연구의 재현가능성과 분석의 타당성을 제고하는 방법론을 제시했다는 점에서 의의가 있다.   
   
### 0.2. English
The purpose of this study is to analyze the policy-making process for preventing conflicts of interest among public officials by combining the multiple streams framework and text mining. To this end, this study collected newspaper articles through web crawling and performed word frequency analysis, weighted log odds ratio analysis and topic modeling analysis with the statistical language R. It identified the streams that appeared by each period, the major actors, events and issues that influence the policy-making process. The analysis is as follows. As a result of analysis, considering the national mood created by series of focusing events, the indicator as a sub-factor of the problem stream shows a significant gap with public perception. Second, the role of policy entrepreneurs in the process of establishing the government agendas is clear. However, the impact on the process of adopting agendas for the legislature turns out to be insignificant. Since there are only a few cases that utilize the quantitative data in the multiple stream framework, this study contributes to the reproducibility of research and validity of analysis by applying text mining analysis to the multiple stream framework with the statistical language R.   

### 0.3. Keywords (Korean)
다중흐름모형, 이해충돌방지법, 텍스트마이닝   
### 0.4. Keywords (English)
Keywords: Multiple Stream Framework, Conflict-of-Interest Prevention Act, Text Mining   
   
## 1. Author
Kim Jusang(1st Author) & Chang Hyunjoo(Corresponding Author)

## 2. Requirements
본 분석은 `MacOS Big Sur(11.5.1 version)`에서 `R(4.1.0 version)` 및 `Rstudio(1.4.1717 version)`으로 진행되었음을 알립니다. `Windows` 환경의 경우 font 오류 등의 문제로 코드상 문제가 발생할 수 있습니다. `MacOS` 환경에서 구동해주시면 감사하겠습니다. 분석결과에 대한 자세한 내용은 논문 원본을 참조해주세요.

### 2.1. 웹크롤링 코드
크롤링 코드 실행을 위해 우선적으로 `KPAR.conflict.of.interest.Rproj`를 실행한 후 `2.code//crawling.R` 을 실행해주세요.  

## 3. 본 분석
본 분석 코드를 살펴보기 위해서는 `KPAR.conflict.of.interest.Rproj`를 실행한 후 `2.code//code.R` 을 실행해주세요.  

### 3.1. 사용 패키지
```{r}
if (!require("pacman")) (install.packages("pacman"))
pacman::p_load(
  tidyverse, magrittr,
  tidytext, tidyr,
  KoNLP, 
  lubridate, scales, 
  tidylo, rvest,
  tm, furrr, topicmodels,
  ggpmisc, # 시계열 시각화
  ggraph, # 네트워크 분석
  widyr, # 단어쌍 
  tidygraph,
  ggthemes
)
```
### 3.2. 사용자 사전
```{r}
user_dic <- data.frame(
  term = c('이해충돌방지법안','이해충돌방지법', "김영란법", "공직자윤리법", "이해충돌", "전수조사", "공공분야", "소급적용", "경제부총리", "사회부총리",
           "김영란", "국민권익위원회", "국민권익위원장", "행정심판위원회", "참여연대", "법제처", "부패영향평가", "국제투명성기구",
           "하태경", "이명박", "박근혜", "문재인", "이성보", "손혜원", "전현희", "이상민", "정홍원", "윤상현", "김한길", "직무관련성", "김태년", "이낙연", "박근혜", "박영선", "안철수", "주호영", "오세훈", "정세균", "추미애", "김병욱", "박덕흠", "성일종", "김기식", "김상조", "박병석", "심상정", "최인호", "이명박", "이완구", "나경원", "박원순", "홍남기", "김용태", "이상민", "박지원", "신동근", 
           "감사원", "청와대", "행정안전부", "법무부", "국무총리", "국무위원", "검찰총장",
           '부정부패','부패방지', "명문화", "직권남용", "부패인식지수",
           '국회의원', "중앙부처", "뇌물죄", "금품수수", "부정청탁", "미공개정보", "퇴직공무원", "퇴직자", "대가성", "축의금", "경조사비", "인사청문회", "포퓰리즘",
           '국민의힘', '자유한국당', '민주당', "더불어민주당", "정의당", "국민의당", "원내대표", "정무위원회", "국회법", "여의도", "임직원", "법무부", "상임위", "정무위", "정무위원회", "사립학교", "법안소위", "새정치민주연합", "반부패정책협의회", "바른미래당", "국정조사", "부동산거래분석원", "새정치연합", "헌법재판소", "국토교통위", "국토교통위원회", "법제사법위원회", "법제사법위", "국토교통부",
           "떡값", "스폰서", "사익추구", "규정",
           "추상적", "포괄적", "이해당사자", "이해관계자",
           "지방자치단체장", 
           '코로나19'), 
  tag ='ncn')
buildDictionary(ext_dic = 'NIAdic', user_dic = user_dic)
```

### 3.4. 데이터 전처리
```{r}
data %>% # 총 3,148 건 데이터 중
  mutate(date = date %>% 
           str_replace_all("(오전|오후) [0-9]{1,2}:[0-9]{1,2}", "") %>% # 날짜에 붙은 시간 제거
           lubridate::as_date(), # 날짜변수를 날짜로 정제
         press = press %>%  # 신문사를
           str_replace_all("\\..*|\n| |(.*| )ⓒ", ""),
         ref = paste0("(", press, ", ", date, ")"))%>% # . 뒷부분 모두 제거 & \n 태그 제거 & ⓒ 앞부분 모두 제거 
  filter(press %in% c("경향신문", "국민일보", "내일신문", "동아일보", "문화일보", "서울신문", "세계일보", "조선일보", "중앙일보", "한겨레", "한국일보")) -> data_prep_ver1 # 일간지만 뽑기
data_prep_ver1 %>% # 일간지만 뽑아서 1,816 건
  mutate(content = content %>%
           str_replace_all("\n\t\n\t\n\n\n\n// flash 오류를 우회하기 위한 함수 추가\nfunction _flash_removeCallback()", "") %>%
           str_replace_all("\n|\t|\\(\\)|\\{\\}", "") %>% 
           str_replace_all("\\[(동아일보|서울신문)\\]", "")) %>% 
  filter(!(year(date) == 2011 & month(date) <= 5 & day(date) <= 30) # 2011년 5월 31일 이전 기사 제외
         ) -> data_prep_ver2 # 1,784 건
data_prep_ver2 %>% write_excel_csv("data_prep_ver2.csv")

# 2.2. 데이터 정제: 기사제목 기준 필터링 (키워드) ----------------------------

data_prep_ver2 %>% 
  filter(!title %>% str_detect("북한|중국|해외|북핵|외교|국제")) %>% # 1,742 건
  filter(!title %>% str_detect("수상|회고록|In&Out|왜냐면")) %>% # 1,731 건
  filter(title %>% str_detect("이해충돌|이해관계|공무원|청탁|공직|뇌물|부패|비리|공정|LH|특권|관피아")) -> data_prep_ver3 # 584 건
data_prep_ver3 %>% write_excel_csv("data_prep_ver3.csv") # 중간 저장

# 2.3. 데이터 정제: 기사제목 기준 필터링 (내용분석) ---------------------

data_prep_ver3 %>% # 584 건 중 내용분석을 통해 걸러내기
  filter(!title %>% str_detect("특임검사|파워우먼|공직열전|보관신탁|휴직|인재개발|분쟁조정 심의|파워인터뷰")) -> data_prep_ver4
data_prep_ver4 # 573 건
data_prep_ver4 %>% write_excel_csv("data_prep_ver4.csv")

# 2.4. 데이터 정제: 본문 태그 필터링 ------------------------------

data_prep_ver4 %>% 
  mutate(
    content = content %>% 
      str_replace_all("☞.*", "") %>%
      str_replace_all("▶.*", "") %>%
      str_replace_all("★.*", "") %>%
      
      ...
      
      ) -> data_prep_ver5
```
수집한 데이터를 기사 제목을 기준으로 1차 정제과정을 거쳐 `584` 건의 신문기사를 확보하였습니다. 이후 2차 정제과정에서는 해당 기사의 본문을 내용분석하여 이해충돌 방지법안의 제정과정과 무관한 기사를 제외하였습니다. 이후 포토기사 등 본문 내 텍스트의 비중이 타 기사와 비교하여 현저히 적은 기사 등을 글자수 기준(한글 200자 이상)으로 여과하여 제외하였습니다.  
  
### 3.5. 형태소 분석
```{r}
data_prep_ver6 %>% 
  unnest_tokens(input = content,
                output = words,
                token = extractNoun,
                drop = F) %>% 
  select(-content) -> data_word
data_word %>%  write_excel_csv("data_word.csv")
```
이해충돌 방지법의 정책결정과정을 분석하기 위해 국내 신문기사를 대상으로 텍스트 마이닝 기법을 통한 텍스트의 최소 단위인 단어를 추출하였습니다. 이를 위해 `KoNLP 패키지(Jeon & Kim, 2016)`를 활용한 한글 토큰화(tokenizing) 작업과 형태소 분석 및 전처리를 수행하였습니다. 데이터 셋 `544`건의 기사를 형태소 분석하여 명사만을 추출해 총 `106,365` 건의 단어 데이터 셋을 구성하였습니다. 숫자 및 불용어를 제거하고 빈도수를 기준(빈도수 5 이하 단어 제거)으로 전처리를 수행하여 해당 데이터를 `96,580` 개의 단어로 전환할 수 있었습니다. 이후 한 글자 단어를 제외하고 유사 단어군을 통합하는 과정을 거치는 2차 정제과정을 통해 `69,864` 개의 단어로 이루어진 최종 데이터 셋을 구성하였습니다.

### 3.6. 전처리 과정 도식화
전처리과정을 도식화한 결과는 다음과 같습니다.  

**데이터 정제 과정**
![_flow](https://user-images.githubusercontent.com/75797388/147895880-8e54d83d-3687-4543-8d5b-1ec5d66964fa.jpg)

## 4. 분석
### 4.1. 빈도분석 및 가중로그승산비 분석
```{r}
data_tf_idf %>% 
  bind_log_odds(set = period, 
                feature = words, 
                n = n) %>%
  rename(log_odds = "log_odds_weighted") -> data_tidylo
```
가중로그승산비를 `tidylo::bind_log_odds`를 통해 계산하였습니다.

```{r}
data_tidylo %>% 
  filter(period == "first") %>% 
  group_by(words) %>%
  summarise(
    n = sum(n, na.rm = TRUE),
    tf_idf = sum(tf_idf, na.rm = TRUE),
    log_odds = sum(log_odds, na.rm = TRUE)
  ) %>%
  arrange(desc(n)) %>%
  ungroup() -> data_final_first
```

### 4.1.1. LDA 분석(토픽모델링)
```{r}
data_final %>% 
  mutate(id = paste0(id, "_", date)) %>% 
  count(id, words, sort = T) %>% 
  bind_tf_idf(words, id, n) %>% 
  cast_dtm(document = id,
           term = words,
           value = n) -> data_dtm
str(data_dtm)
topics <- c(2:15)
topics %>% 
  future_map(
    LDA, x = data_dtm, control = list(seed = 486)
    ) -> data_lda
    
tibble(
  k = topics,
  perplex = map_dbl(data_lda, perplexity)
) -> data_lda_prep

data_lda_prep %>% 
  ggplot(mapping = aes(x = k, 
                       y = perplex)) +
  geom_point() +
  geom_line() +
  ggplot2::geom_vline(xintercept = 9, size = 1, color = 'red', alpha = 0.7, linetype = 2) -> data_lda_엘보우
data_lda_엘보우

data_lda <- LDA(data_dtm, k=9, control=list(seed=486))
data_lda %>% 
  tidy(matrix = "beta") -> data_topics
data_topics %>% write_excel_csv("data_topics.csv")
read_csv("data_topics.csv") -> data_topics
data_topics %>% 
  group_by(topic) %>% 
  slice_max(beta, n = 30) %>%
  ungroup() %>%
  arrange(topic, -beta) -> data_topic_terms
data_topic_terms %>% write_excel_csv("topic.csv")
```
```{r}
List of 6
 $ i       : int [1:45385] 1 6 17 24 38 43 45 53 83 91 ...
 $ j       : int [1:45385] 1 1 1 1 1 1 1 1 1 1 ...
 $ v       : num [1:45385] 32 1 1 1 7 1 2 1 1 3 ...
 $ nrow    : int 544
 $ ncol    : int 2201
 $ dimnames:List of 2
  ..$ Docs : chr [1:544] "2310_2020-05-07" "1425_2016-05-31" "2074_2019-03-08" "2192_2019-08-22" ...
  ..$ Terms: chr [1:2201] "특권" "관리" "공동주택" "채용" ...
 - attr(*, "class")= chr [1:2] "DocumentTermMatrix" "simple_triplet_matrix"
 - attr(*, "weighting")= chr [1:2] "term frequency" "tf"
  
A LDA_VEM topic model with 9 topics.  
  
A tibble: 270 x 3
   topic term       beta
   <int> <chr>     <dbl>
 1     1 공직자   0.0358
 2     1 직무     0.0226
 3     1 이해충돌 0.0216
 4     1 공무원   0.0186
 5     1 업무     0.0146
 6     1 관련     0.0144
 7     1 금지     0.0123
 8     1 규정     0.0120
 9     1 주식     0.0116
10     1 심사     0.0111
… with 260 more rows
```

토픽모델링 분석을 통해 9개의 토픽을 추출하였습니다.  
  
### 4.1.2. 시기별 누적 토픽수 계산
```{r}
data_gamma <- tidy(data_lda, matrix = "gamma")
data_gamma %>%
  separate(document, c("document", "date"), sep = "_", convert = T) %>% 
  mutate(
    year = year(date),
    month = month(date),
    quarter = lubridate::quarter(date),
    yq = paste0(year, ": ", quarter, "Q"),
    quarterly = lubridate::yq(yq),
    topic = as.factor(topic)
  ) %>%
  group_by(topic, year) %>%
  summarise(sum = sum(gamma)) %>%
  ungroup() -> data_gamma_visualization
```
### 4.2. 토픽모델링 시각화
토픽모델링 분석을 시각화한 결과는 다음과 같습니다.   
![그림3_토픽모델링 분석 결과](https://user-images.githubusercontent.com/75797388/178692334-9b2e2597-3955-4f38-b19c-b171c14b79d2.jpeg)
   
![그림4_시기별 토픽 비중](https://user-images.githubusercontent.com/75797388/178692349-3dfebd6d-6401-4ce6-8076-727109bb99f5.jpeg)
   
![그림5_시기별 토픽 적재확률](https://user-images.githubusercontent.com/75797388/178692361-333a98d6-b151-459a-8c0c-36f362d2a328.jpeg)

### 5. 분석결과 해석
논문을 참조해주세요.

