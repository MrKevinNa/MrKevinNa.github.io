R로 웹데이터 수집하기
---------------------

먼저 우리가 인터넷 페이지를 검색하는 방식을 떠올려봅시다![1]

![웹페이지 접속하기](https://ruslanspivak.com/lsbaws-part1/LSBAWS_HTTP_request_response.png)

위와 같이 웹브라우저에 url을 입력하는 행위를 통해 **HTTP Request**를 던지면 웹서버로부터 **HTTP Response**를 받아 웹브라우저에 **HTML을 Rendering** 해줍니다.

Web Crawler의 기본 과정은 원하는 사이트에 요청을 보내고 응답을 받은 후, html로부터 필요한 데이터를 정리하는 것입니다.

웹페이지를 "요청(Request)"하는 방법으로는 GET, POST 함수 등을 사용할 수 있습니다. 이번에는 GET 함수 사용법을 알아보겠습니다.

``` r
# 패키지 설치하기
install.packages(c("httr", "rvest", "dplyr", "magrittr"))
```

``` r
# 패키지 불러오기
library(httr)
library(rvest)
library(dplyr)
library(magrittr)
```

### html을 가져오는 순서

네이버에서 제공되고 있는 실시간 검색어를 가져오는 간단한 예로 시작을 해보겠습니다. 1. 네이버에 접속해서 url을 확인합니다. 1. httr 패키지의 GET 함수 인자로 url을 할당해줍니다. 1. 위 과정에서 가져온 response의 상태와 html의 구조를 텍스트로 확인합니다.
- 상태를 확인하려면 status\_code 함수를 사용합니다. (200이면 정상!) - html의 구조를 출력하려면 content 함수를 사용합니다.

``` r
url <- "https://www.naver.com"
resp <- GET(url)
status_code(resp)
```

    ## [1] 200

``` r
# html 출력해서 육안으로 확인하기 (하지만 어지러움 ^^;)
content(x = resp, as = 'text')
```

상태코드가 200이라 정상적으로 "응답(Response)"하였습니다. 그리고 html이 어지럽게 출력이 되었네요.

html tag의 기본적인 형태를 확인해보겠습니다.[2]

![html tag](http://tutorial.techaltum.com/images/element.png)

### tag를 다루는 기본 함수들

1.  read\_html(x, encoding) : resp에 있는 html을 읽습니다. 이 때 encoding = "UTF-8"을 추가합니다.
2.  html\_node(x, css, xpath) : 괄호 안에 할당된 키워드에 맞는 tag 중 맨 처음 1개만 가져옵니다.
3.  html\_nodes(x, css, xpath) : 위에 해당하는 모든 tag를 가져옵니다.
4.  html\_text(x) : tag 사이에 있는 텍스트만 가져옵니다.
5.  html\_table(x) : html에서 표로 되어 있는 데이터를 그대로 가져올 수 있습니다.

우리가 원하는 데이터인 "실시간 검색어"와 관련된 tag를 확인하는 방법을 알려드리겠습니다. 웹브라우저로 크롬을 사용하신다면 원하는 부분으로 마우스를 옮긴 후, 오른쪽 버튼을 클릭하고 "검사(Inspect)"를 선택합니다.

먼저 `<ul class="ah_l" data-list="1to10" style="display: block;">`와 같은 tag를 확인할 수 있을 것입니다. 그리고 이 tag 아래에 `<li class="ah_item" ... >` tag들이 하위 tag로 붙어 있는 것이 보일 겁니다.

html에서 특정 부분을 찾으려면 tag에서 xpath나 CSS Selector를 확인한 후, 위에 보여드린 함수의 xpath 또는 css 인자에 할당을 해주어야 합니다.
- 속성명이 "id"나 "class"인 경우에는 아래와 같이 기호로 대체할 수 있습니다.
- id 대신 '\#' (예, "div\#post\_title")
- class 대신 '.' (예, "div.content")

``` r
# 전체 html 중에서 "실시간 검색어" 관련 tag만 추출하기
html <- read_html(x = resp, encoding = "UTF-8")
items <- html_nodes(x = html, css = "ul.ah_l li a span.ah_k")
print(items)
```

    ## {xml_nodeset (40)}
    ##  [1] <span class="ah_k">레이버 데이</span>
    ##  [2] <span class="ah_k">신촌세브란스병원</span>
    ##  [3] <span class="ah_k">한국사능력검정시험</span>
    ##  [4] <span class="ah_k">제주공항</span>
    ##  [5] <span class="ah_k">신촌세브란스병원 화재</span>
    ##  [6] <span class="ah_k">태양</span>
    ##  [7] <span class="ah_k">미스티</span>
    ##  [8] <span class="ah_k">오민석</span>
    ##  [9] <span class="ah_k">삼성전자서비스센터</span>
    ## [10] <span class="ah_k">민효린</span>
    ## [11] <span class="ah_k">영화순위</span>
    ## [12] <span class="ah_k">1987</span>
    ## [13] <span class="ah_k">nba</span>
    ## [14] <span class="ah_k">나혼자산다</span>
    ## [15] <span class="ah_k">오버워치 리그</span>
    ## [16] <span class="ah_k">메가박스</span>
    ## [17] <span class="ah_k">롯데시네마</span>
    ## [18] <span class="ah_k">그것이 알고싶다</span>
    ## [19] <span class="ah_k">윤식당 2</span>
    ## [20] <span class="ah_k">한국세무사회자격시험</span>
    ## ...

``` r
# tag에서 텍스트만 추출하기
searchWords <- html_text(x = items)
print(searchWords)
```

    ##  [1] "레이버 데이"           "신촌세브란스병원"     
    ##  [3] "한국사능력검정시험"    "제주공항"             
    ##  [5] "신촌세브란스병원 화재" "태양"                 
    ##  [7] "미스티"                "오민석"               
    ##  [9] "삼성전자서비스센터"    "민효린"               
    ## [11] "영화순위"              "1987"                 
    ## [13] "nba"                   "나혼자산다"           
    ## [15] "오버워치 리그"         "메가박스"             
    ## [17] "롯데시네마"            "그것이 알고싶다"      
    ## [19] "윤식당 2"              "한국세무사회자격시험" 
    ## [21] "레이버 데이"           "신촌세브란스병원"     
    ## [23] "한국사능력검정시험"    "제주공항"             
    ## [25] "신촌세브란스병원 화재" "태양"                 
    ## [27] "미스티"                "오민석"               
    ## [29] "삼성전자서비스센터"    "민효린"               
    ## [31] "영화순위"              "1987"                 
    ## [33] "nba"                   "나혼자산다"           
    ## [35] "오버워치 리그"         "메가박스"             
    ## [37] "롯데시네마"            "그것이 알고싶다"      
    ## [39] "윤식당 2"              "한국세무사회자격시험"

``` r
# 실시간 검색어가 2번 반복되고 있으므로 상위 20개만 추출하여 데이터 프레임으로 저장
searchWords <- data.frame(searchWords = searchWords[1:20])
```

그런데 위와 같이 하는 것보다 dplyr 패키지의 파이프 연산자(%&gt;%)를 사용하면 가독성 높은 코드로 만들 수 있습니다. 파이프 연산자는 앞에서 반환된 객체를 뒤에 오는 함수의 첫 번째 인자로 자동 할당을 해주는 기능을 합니다. 따라서 불필요한 객체들을 만들지 않아도 되는 효과가 있습니다.

``` r
read_html(x = resp, encoding = "UTF-8") %>% 
  html_nodes(css = "ul.ah_l li a span.ah_k") %>% 
  html_text() %>% 
  head(n = 20) %>% 
  data.frame() %>% 
  set_colnames("searchWords")
```

    ##              searchWords
    ## 1            레이버 데이
    ## 2       신촌세브란스병원
    ## 3     한국사능력검정시험
    ## 4               제주공항
    ## 5  신촌세브란스병원 화재
    ## 6                   태양
    ## 7                 미스티
    ## 8                 오민석
    ## 9     삼성전자서비스센터
    ## 10                민효린
    ## 11              영화순위
    ## 12                  1987
    ## 13                   nba
    ## 14            나혼자산다
    ## 15         오버워치 리그
    ## 16              메가박스
    ## 17            롯데시네마
    ## 18       그것이 알고싶다
    ## 19              윤식당 2
    ## 20  한국세무사회자격시험

위 코드에서 마지막에 오는 set\_colnames 함수는 magrittr 패키지에 속합니다. 이 함수를 추가하지 않으면 새로 생성된 데이터 프레임의 열이름에 '.'으로 설정됩니다.

이상 GET 함수를 이용하여 "네이버 실시간 검색어"를 수집하는 방법을 알아보았습니다. 다음에는 url이 바뀌지 않아 원하는 텍스트 데이터를 수집하지 못할 때 사용하는 POST 함수에 대해서 알아보겠습니다.

[1] 참조 : <https://ruslanspivak.com/lsbaws-part1/>

[2] 참조 : <http://tutorial.techaltum.com/htmlTags.html>
