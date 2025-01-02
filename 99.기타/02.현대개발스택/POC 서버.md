# POC 서버 스택정리



| 항목                     | 상세내용                                                     |
| ------------------------ | ------------------------------------------------------------ |
| 어플리케이션 이름        | Spring Boot POC Server                                       |
| 기능/역할                | 배포하는 내용에 대한 인증 서버                               |
| 언어 및 프레임워크       | JAVA 17 + Spring Boot 2.7                                    |
| 아키텍처                 | TBD                                                          |
| 데이터베이스             | Aurora Mysql 3                                               |
| 사용 프로토콜            | - HTTPS                                                      |
| 캐시                     | TBD                                                          |
| CI/CD                    | Argo Rollouts + Jenkins                                      |
| 사용 도구 및  라이브러리 | - org.springframework.boot                                <br/>- org.springframework.boot:spring-boot-starter-web |
| 호스팅 플랫폼            | AWS EKS                                                      |
| 보안 설정                | OAuth2, TLS 1.2                                              |
| 배포 환경                | develop, prod                                                |



### JAVA

| JAVA | version |
| ---- | ------- |
| JDK  | 17      |



### Application

| dependency                                       | version |
| ------------------------------------------------ | ------- |
| org.springframework.boot                         | 2.7.4   |
| org.springframework.boot:spring-boot-starter-web | 2.7.4   |



### Protocol

| protocol |
| -------- |
| HTTPS    |



### DB

| DB           | version |
| ------------ | ------- |
| Aurora Mysql | 3       |



# POC 화면 스택정리

| 항목               | 상세내용                           |
| ------------------ | ---------------------------------- |
| 어플리케이션 이름  | Vue POC 화면                       |
| 페이지  역할       | 배포에 필요한 기능 제공            |
| 언어 및 프레임워크 | Vue 3 + TypeScript                 |
| UI 라이브러리      | Tailwind CSS                       |
| 상태 관리          | Pinia                              |
| 라우팅             | Vue Router                         |
| API 연동           | Axios로 어플리케이션 REST API 호출 |
| 빌드 도구          | Vite                               |
| 배포 환경          | S3 + CloudFront                    |
| 호환 브라우저      | 최신 Chrome, Edge                  |
