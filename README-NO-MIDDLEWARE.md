# MSA DevOps Lab - No Eureka / Redis / Kafka / Zookeeper

이 버전은 DevOps 수업용으로 Eureka, Redis, Kafka, Zookeeper를 제거한 버전입니다.

## 유지되는 기능
- 회원가입
- 로그인
- Access Token / Refresh Token 발급
- Refresh Token으로 Access Token 재발급 (Redis 없이 JWT 자체 검증)
- 상품 등록
- S3 상품 이미지 업로드
- 상품 목록/상세 조회 및 이미지 표시
- 주문 생성
- Frontend 구매 버튼 -> ordering-service 주문 생성
- 주문 시 상품 재고 감소 (Kafka 대신 ordering-service -> product-service Feign 동기 호출)
- API Gateway / Ingress / HPA / ArgoCD / Prometheus / Grafana 구조 유지

## 제거된 구성
- Eureka Client 의존성 및 설정
- Redis 의존성, RedisConfig, redis.yml
- Kafka 의존성 및 Producer/Consumer Config, kafka.yml
- Zookeeper manifest, docker-compose Kafka/Zookeeper

## 실습 전 수정 필요
1. 모든 K8s Deployment의 ECR Account ID를 학생 AWS Account ID로 수정
2. `msa/product/src/main/resources/application-prod.yml`의 S3 bucket 값을 학생 버킷명으로 수정
3. Kubernetes Secret `my-app-secrets`: DB_HOST, DB_PW
4. Kubernetes Secret `aws-secret`: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
5. S3 Browser 이미지 표시를 위해 `products/*` 읽기 정책 또는 CloudFront 구성

## 주의
교육용 단순화를 위해 Refresh Token은 서버 저장소(배포 전 Redis)에 저장하지 않고 서명/만료시간만 검증합니다. 따라서 Redis 기반 강제 로그아웃/Refresh Token 폐기 기능은 제공하지 않습니다. 회원가입/로그인/토큰 재발급이라는 사용자 기능은 유지됩니다.

## 검증 결과
- Eureka / Redis / Kafka / Zookeeper 관련 소스 및 manifest 참조 제거 확인
- ordering-service의 재고 차감이 `productFeign.updateProductStock()` 직접 호출로 변경됨
- ordering Deployment의 잘못된 ECR 이미지(member-service)를 ordering-service로 수정함
- 이 제작 환경은 외부 인터넷이 차단되어 Gradle Wrapper가 Gradle 배포본을 다운로드하지 못했으므로 `bootJar` 실제 빌드 검증은 수행하지 못함
