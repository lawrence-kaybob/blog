이번학기에 수강하는 강의 중 '오픈소스 소프트웨어 입문'이라는 강의가 있습니다. 말 그대로 오픈소스 소프트웨어에 대한 개념과 일부 오픈소스 소프트웨어를 직접 사용해보는 강의입니다. 전반기는 Git에 대해 배우고, 후반기에는 최종적으로 ELK(Elasticsearch, Logstash, Kibana)를 이용해 실제 데이터 분석을 체험해본다고 합니다.

그래서 교수님이 후반기에 들어가면서 내주신 과제가 AWS EC2 인스턴스를 생성하고 LAMP 스택을 설치하고, phpMyAdmin을 연결해서 외부에 접속해보도록 하는 것이었습니다. 이미 AWS EC2를 사용하고 있기 때문에 (AWS Educate 치트키) 혹시 새로운 인스턴스를 만들고 해야하는지 질문을 했더니, 클라우드 환경을 써보라는 의미에서 내는 과제이니 상관없다고 하셨습니다.

~~ㄱㅇㄷ!~~

문제는 지금 사용하고 있는 인스턴스가 도커와 함께 [리버스 프록시](https://www.lesstif.com/pages/viewpage.action?pageId=21430345)를 사용한다는 것입니다. 동아리 서버의 테스트 서버로서 사용하고 있으며, 여러 

그래서 목적은 `http://example.com/phpmyadmin`에서 phpMyAdmin을 접속할 수 있도록 하는 것이었습니다.

이전에 워드프레스 설치하는 글에서 설명했듯이, 라우팅 규칙은 `traefik.toml`에서 설정해주면 됩니다. 그 중 `frontend` 설정에서 `Path:` 값만 설정해주면 됩니다. 그래서 처음 시도했던 파일은 아래와 같습니다.

```toml
# traefik.toml
logLevel = "DEBUG"
defaultEntryPoints = ["http"]
[entryPoints]
  [entryPoints.http]
  address = ":80"

[web]
address = ":8080"
  [web.statistics]
  RecentErrors = 10

[file]

[backends]
  [backends.DoiTPage]
    [backends.DoiTPage.servers.Wordpress]
    url="http://172.18.0.4:80"
  [backends.myadmin]
    [backends.myadmin.servers.myadmin]
    url="http://172.18.0.3:80"

[frontends]
  [frontends.DoiTPage]
  backend = "DoiTPage"
  passHostHeader = true
    [frontends.DoiTPage.routes.default]
    rule="Host:192.168.99.100;"
   [frontends.myadmin]
   backend = "myadmin"
   passHostHeader = true
     [frontends.myadmin.routes.default]
     rule="PathPrefixStrip:/phpmyadmin"

```

phpMyAdmin은 Reverse Proxy에서 돌아가더라도, phpMyAdmin 자체 백엔드에서는 `/`(즉, root 디렉토리)에서 서비스되고 있기 때문에 요청받은 URL을 바꿔서 넘겨줘야했습니다. 따라서, 처음 의도는 사용자가 Path에 `/phpMyAdmin`으로 접속을 시도하면

BSNYwRS!8D5%jwNS4K