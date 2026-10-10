# 회사 Spring vs Spring Boot
- 회사에서는 db연결 설정을 XML로 등록했는데, Boot에서는 application.properties에 주소와 계정만 적으면 연결이 자동으로 구성됨.
- 회사에서는 서버를 war를 외부 tomcat에 배포하는 방식으로 띄우는데, Boot에서는 앱 내부에 서버가 존재
- 라이브러리 버전은 boot가 자동으로 맞춰줌
- 아직 모르겠는 것: DispatcherServlet, ViewResolver