# 0. set R Selenium
=================

### terminal에서 chromedriver가 설치되어 있는 폴더로 이동 후 아래 명령을 실행

-   $ cd ~/Documents/WebDrivers
-   $ java -Dwebdriver.chrome.driver="chromedriver" -jar
    selenium-server-standalone-3.5.3.jar -port 4445

# 1. get sports news urls
=======================

### import libraries

    library(RSelenium)
    library(httr)
    library(rvest)

    ## Loading required package: xml2

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(stringr)

### execute RSelenium

    remote <- remoteDriver(remoteServerAddr = 'localhost',
                           port = 4445L,
                           browserName = "Chrome")

### open a browser window

    remote$open()

    ## [1] "Connecting to remote server"

    ## Warning in strptime(x, fmt, tz = "GMT"): unknown timezone 'zone/tz/2017c.
    ## 1.0/zoneinfo/Asia/Seoul'

    ## $applicationCacheEnabled
    ## [1] FALSE
    ## 
    ## $rotatable
    ## [1] FALSE
    ## 
    ## $mobileEmulationEnabled
    ## [1] FALSE
    ## 
    ## $networkConnectionEnabled
    ## [1] FALSE
    ## 
    ## $chrome
    ## $chrome$chromedriverVersion
    ## [1] "2.33.506106 (8a06c39c4582fbfbab6966dbb1c38a9173bfb1a2)"
    ## 
    ## $chrome$userDataDir
    ## [1] "/var/folders/j7/vm8qb29s2rl4c5zwhxg895t40000gn/T/.org.chromium.Chromium.g1wLgJ"
    ## 
    ## 
    ## $takesHeapSnapshot
    ## [1] TRUE
    ## 
    ## $pageLoadStrategy
    ## [1] "normal"
    ## 
    ## $databaseEnabled
    ## [1] FALSE
    ## 
    ## $handlesAlerts
    ## [1] TRUE
    ## 
    ## $hasTouchScreen
    ## [1] FALSE
    ## 
    ## $version
    ## [1] "62.0.3202.94"
    ## 
    ## $platform
    ## [1] "Mac OS X"
    ## 
    ## $browserConnectionEnabled
    ## [1] FALSE
    ## 
    ## $nativeEvents
    ## [1] TRUE
    ## 
    ## $acceptSslCerts
    ## [1] TRUE
    ## 
    ## $locationContextEnabled
    ## [1] TRUE
    ## 
    ## $webStorageEnabled
    ## [1] TRUE
    ## 
    ## $browserName
    ## [1] "chrome"
    ## 
    ## $takesScreenshot
    ## [1] TRUE
    ## 
    ## $javascriptEnabled
    ## [1] TRUE
    ## 
    ## $cssSelectorsEnabled
    ## [1] TRUE
    ## 
    ## $setWindowRect
    ## [1] TRUE
    ## 
    ## $unexpectedAlertBehaviour
    ## [1] ""
    ## 
    ## $id
    ## [1] "1465d8284c9deca4a894a6962d1930f6"

### set date

    refDate <- Sys.Date() %>% 
      str_replace_all(pattern = "-", replacement = "")

### assembel a URL

    main <- "http://m.sports.naver.com"
    ctgr <- "/kbaseball/news/index.nhn?isPhoto="
    type <- paste0("&type=", "popular")
    date <- paste0("&date=", refDate)
    (url <- paste0(main, ctgr, type, date))

    ## [1] "http://m.sports.naver.com/kbaseball/news/index.nhn?isPhoto=&type=popular&date=20171130"

### get access to the URL

    remote$navigate(url)

### get html

    html <- remote$getPageSource()[[1]]

    # run below if you want to check html with the naked eyes! 
    # cat(html)

### read html

    html <- read_html(html)

### find nodes for news articles

    news <- html %>% html_nodes(css = "ul.article_lst li")

### create a data frame for news articles

    newsDf <- data.frame()

    for (i in 1:length(news)) {
      
      # get information of each article
      slink <- news[[i]] %>% html_node(css = "a") %>% html_attr(name = "href")
      title <- news[[i]] %>% html_node(css = "div.text span") %>% html_text()
      paper <- news[[i]] %>% html_node(css = "div.source span.provider") %>% html_text()
      visit <- news[[i]] %>% html_node(css = "div.source span.visit") %>% html_text()
      
      # assemble each information and add to newsDf
      df <- data.frame(slink, title, paper, visit)
      newsDf <- rbind(newsDf, df)
    }

### remove comma in the column of "visit" and convert into a numeric vector

    newsDf$visit <- str_replace_all(string = newsDf$visit, pattern = ",", replacement = "")
    newsDf$visit <- as.numeric(newsDf$visit)

### check the data frame

    head(newsDf, 10)

    ##                                              slink
    ## 1  /kbaseball/news/read.nhn?oid=109&aid=0003672077
    ## 2  /kbaseball/news/read.nhn?oid=109&aid=0003672105
    ## 3  /kbaseball/news/read.nhn?oid=109&aid=0003672078
    ## 4  /kbaseball/news/read.nhn?oid=001&aid=0009718685
    ## 5  /kbaseball/news/read.nhn?oid=076&aid=0003186596
    ## 6  /kbaseball/news/read.nhn?oid=108&aid=0002664040
    ## 7  /kbaseball/news/read.nhn?oid=477&aid=0000100875
    ## 8  /kbaseball/news/read.nhn?oid=382&aid=0000610248
    ## 9  /kbaseball/news/read.nhn?oid=109&aid=0003671976
    ## 10 /kbaseball/news/read.nhn?oid=109&aid=0003672084
    ##                                                           title
    ## 1          [오피셜] KIA 외인트리오 전원 재계약…정상수성 첫 단추
    ## 2  [일문일답] 삼성 강민호, "삼성 유니폼을 입으니 이적 실감난다"
    ## 3                ‘외인 재계약 완료’ KIA, 양현종-김주찬만 남았다
    ## 4             정성훈·김경언 등 79명 방출…KBO 보류선수 명단 공시
    ## 5          [인터뷰] '트레이드' 한기주 "이제 두려울 것도 없어요"
    ## 6     [오피셜] 롯데, 레일리·번즈와 재계약..린드블럼과 협상 지속
    ## 7          '우승해도 무리 없다' KIA가 세운 연봉 협상 가이드라인
    ## 8            ‘조용한 속도전’ 양현종까지 시야 넣는 SK의 선발수집
    ## 9                 '삼민호 환영합니다' 삼성, 강민호 맞이 준비 끝
    ## 10                   '200만 달러' 헥터, 니퍼트 기록은 못 넘겼다
    ##           paper  visit
    ## 1          OSEN 178481
    ## 2          OSEN  63910
    ## 3          OSEN  52146
    ## 4      연합뉴스  43815
    ## 5    스포츠조선  36618
    ## 6      스타뉴스  25305
    ## 7  스포티비뉴스  24109
    ## 8    스포츠동아  23811
    ## 9          OSEN  18230
    ## 10         OSEN  17347

### close the browser window

    remote$close()

### check r objects to remove all except the news article data frame

    rm(list = ls()[! ls() %in% c("newsDf", "main")])

# 3. use javascript api to get replies fast ----
==============================================

### sample url for comments api

-   api &lt;-
    "<http://apis.naver.com/commentBox/cbox/web_naver_list_jsonp.json?ticket=sports&pool=cbox2&lang=ko&country=KR&objectId=news001%2C0009717615&page=1>"

### import libraries

    library(httr)
    library(rvest)
    library(jsonlite)
    library(stringr)
    library(jsonlite)

### set user agent

    ua <- "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36"

### assign 1 to i which is for a foo-loop later

    i <- 1

### set 'ref' for getting access to the javascript api

    (ref <- paste0(main, newsDf$slink[i]))

    ## [1] "http://m.sports.naver.com/kbaseball/news/read.nhn?oid=109&aid=0003672077"

### get oid and aid from the news article url

    oid <- ref %>% 
      str_extract(pattern = "(oid=)[[:alnum:]]{1,}") %>% 
      str_split(pattern = "=") %>% 
      sapply(FUN = "[", 2)

    aid <- ref %>% 
      str_extract(pattern = "(aid=)[[:alnum:]]{1,}") %>% 
      str_split(pattern = "=") %>% 
      sapply(FUN = "[", 2)

### assemble comment api url

    apis <- "http://apis.naver.com/commentBox/cbox/web_naver_list_jsonp.json"
    cond <- "?ticket=sports&pool=cbox2&lang=ko&country=KR"
    objt <- paste0("&objectId=", "news", oid, "%2C", aid)
    page <- paste0("&page=", i)

    (apiUrl <- paste0(apis, cond, objt, page))

    ## [1] "http://apis.naver.com/commentBox/cbox/web_naver_list_jsonp.json?ticket=sports&pool=cbox2&lang=ko&country=KR&objectId=news109%2C0003672077&page=1"

### get comment api response

    resp <- GET(url = apiUrl,
                add_headers(Referer = ref),
                user_agent(ua))

### check response status\_code

    status_code(resp)

    ## [1] 200

    # run if you want to see html with the naked eyes
    # content(resp, as = 'text')

### modify json object to get comments using fromJSON()

    comments <- content(resp, as = 'text') %>% 
      str_replace_all(pattern = "_callback", replacement = "") %>% 
      str_replace_all(pattern = ";", replacement = "") %>%
      str_replace_all(pattern = "\n", replacement = "") %>% 
      str_replace_all(pattern = "\\(", replacement = "[") %>%
      str_replace_all(pattern = "\\)", replacement = "]")

### get comments

    comments <- fromJSON(comments)$result$commentList[[1]]$contents

### print comments

    print(comments)

    ##  [1] "개 굿 기아의 보물들로 남아라"                                                                                                                        
    ##  [2] "이제 양현종만 잡자 ㅅ ㅅ ㅅ ㅅ ㅅ ㅅ ㅅ ㅅ ㅅ"                                                                                                       
    ##  [3] "쑤아릿ㅅㅅㅅㅅㅅㅅㅅㅅㅅㅅㅅ"                                                                                                                        
    ##  [4] "재계약 ㅅㅅㅅㅅㅅㅅㅅ"                                                                                                                               
    ##  [5] "오!!기아빠르군!!!"                                                                                                                                   
    ##  [6] "3명한꺼번에 발표하는 클라스보소"                                                                                                                     
    ##  [7] "싸네 3명 다해도 이대호 보다 조금 더 비싸네 ㅋㅋㅋ"                                                                                                   
    ##  [8] "주처하고...현종이는?????? 빨리 계약하자!!! 현기증 난다!!!!!"                                                                                         
    ##  [9] "기아 프런트 일잘한다 매우칭찬해!"                                                                                                                    
    ## [10] "핵터 일본안가네"                                                                                                                                     
    ## [11] "생각보다 그렇게 안비싸게들 잘 잡은듯!"                                                                                                               
    ## [12] "역시 기아 프론트 조용하게 일 잘하시네요. 연속우승을 향해서 ㄱㄱ"                                                                                     
    ## [13] "이제 양현종이랑 본격 협상 하겠네..."                                                                                                                 
    ## [14] "자~ 이제 양현종 김주찬 마무리하자!!!"                                                                                                                
    ## [15] "이제 양현종만"                                                                                                                                       
    ## [16] "나는 솔직히 이번 퐈나 잔류는 걱정안됐음 기사가빨리뜨길바랬을뿐 외인들완조니 기사모고 현종이는 나보다더 기아팬임 차니는 뭐당연히 남을거고 말랑피셜 ㅋ"
    ## [17] "기아에 남아줘서 고마워요~\n내년에도 헬멧 세레모니~ 굿!!!"                                                                                            
    ## [18] "드뎌~~!!!기아프런트 열일하네!!!아주 칭찬해~"                                                                                                         
    ## [19] "최강기아"                                                                                                                                            
    ## [20] "ㅅㅅㅅㅅㅅㅅㅅ 화이팅입니다 내년시즌도"
