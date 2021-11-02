##  이클립스를 통해 SVN 이용하기



SVN을 이용할 수 있는 클라이언트 프로그램에는 Visual SVN, Tortoise SVN 등이 있다. 필자가 사용하고자 하는 것은 위 프로그램 외에, 기본적인 개발 프로그램이자 SVN을 사용할 수 있는 **이클립스**를 통해 SVN 서버에 접속해 관리해보겠다.



#### 1. 이클립스에 SVN 플러그인 설치하기

이클립스에 SVN을 설치하는 방법은 3가지가 있다.

1. 이클립스 마켓플레이스 이용
2. 플러그인 링크 URL 이용
3. 플러그인을 파일들을 다운받아 직접 이클립스 디렉토리에 복사



위 방법 중 간편하게 이클립스 마켓플레이스를 이용하는 것을 선택했다.

이클립스 `help` - `Eclipse Marketplace` 선택

* Subversive - SVN Team Provider 4.0.5 검색 및 인스톨

  [기본 선택사항] - [라이센스 동의] - 재시작



위에서 설치한 Subversive - SVN Team Provider는 실질적인 SVN 서버 연결 기능(SVN 프로토콜)을 직접 제공하지 않는다. SVN 서버와 통신을 위해 SVN Connector가 필요하다.

> SVN Connector는 오픈 소스이긴 하지만, EPL 라이센스가 아니기 때문에 이클립스에서 직접 배포할 수 없어서, 외부 웹사이트를 통해 설치해야 한다.



#### 2. SVN Connector 설치

이클립스 `Window` - `Preferences` 메뉴 - [SVN] - [SVN connector] 탭 - Get Connectors 선택 

* Subversive Connector Discovery 목록 중 SVN Kit 1.8.14 설치

[기본 선택사항]에 SVN Connectors가 포함되어있는지 확인 및 설치 - [라이센스 동의]

> 이클립스 서버를 통해 설치되지 않은 플러그인에 대해 경고 메시지가 표시된다. 앞서 말했던 EPL라이센스가 없어서 이클립스 외부를 통해 설치하기 때문에 그런 것이다.



#### 3. 이클립스로 SVN 접속하기

이클립스에서 SVN을 사용하기 위한 준비가 끝났다. SVN 레포지토리 접속 URL과 Authentication 정보를 이용해 접속하여 이용한다. 이클립스의 SVN 사용방법은 SVN 기본 사용법과 유사하며 SVN 구축에 관한 문서에 설명되어있다.



ref) https://recipes4dev.tistory.com/155

ref) https://blog.naver.com/hj_kim97/222317744478































