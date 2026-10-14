# Validation Report

## 검증 완료 항목

- Eureka / Redis / Kafka / Zookeeper 관련 문자열 및 파일: 0건
- YAML 파일 43개 파싱: 성공
- Frontend JavaScript `app.js` 문법 검사: 성공
- Java 소스 parser/syntax smoke test: apigateway/member/ordering/product 모두 parser 오류 없음
- HPA -> Deployment 이름 연결: 정상
- Ingress -> Service 이름 연결: 정상
  - `argocd-server` Service는 Argo CD 공식 설치 manifest가 생성하므로 ZIP 내부에 없는 것이 정상
- Argo CD Application path: 모두 실제 경로 존재
- Argo CD Application 이름 중복: 제거 완료
- GitHub Actions 서비스 경로/ECR repository 명칭: 확인
- GitHub Actions kubectl: v1.35.0으로 수정

## 기능 연결 검증

### 회원
- 회원가입: `/member-service/member/create`
- 로그인: `/member-service/member/doLogin`
- Access Token / Refresh Token 발급 유지
- Refresh Token 재발급: Redis 없이 JWT 서명/만료 검증

### 상품
- 상품 등록: `/product-service/product/create`
- multipart 이미지 업로드 유지
- S3 업로드 유지
- S3 Object Key는 `products/` prefix 사용
- 상품 목록: `/product-service/product/list`
- 상품 상세: `/product-service/product/{id}`
- frontend가 `imageUrl`을 사용해 이미지 표시

### 주문
- frontend 구매 버튼이 `/ordering-service/ordering/create` 호출하도록 수정
- ordering-service가 Kubernetes DNS `http://product-service`로 직접 호출
- 주문 생성 전에 재고 확인
- 주문 시 product-service `/product/updatestock` 호출로 재고 감소
- ordering 테이블에 주문 레코드 생성
- Kafka 없이 기존 주문/재고 기능 유지

## 함께 수정한 기존 오류

1. `ordering-depl`이 `member-service` 이미지를 사용하던 문제
   - `ordering-service:latest`로 수정
2. `monitoring-https.yml`의 YAML 탭 문자 오류
   - 공백으로 수정
3. Argo CD Application 이름 `msa-services` 중복
   - `msa-argocd`로 분리
4. 예전 모놀리식 저장소 `2.ordersystem/...`을 가리키던 잔여 Application 제거
5. OpenFeign `@PathVariable` 이름을 명시하여 런타임 파라미터 해석 안정화
6. GitHub Actions kubectl 1.25.9 -> 1.35.0

## 실제 빌드 검증 제한

Gradle Wrapper 실행까지 시도했지만, 현재 제작 환경은 외부 DNS/인터넷이 차단되어 다음 단계에서 중단됨:

`https://services.gradle.org/distributions/gradle-8.12.1-bin.zip`

오류: `java.net.UnknownHostException: services.gradle.org`

따라서 Gradle 의존성을 내려받는 `compileJava` / `bootJar` 전체 빌드는 이 환경에서 완료할 수 없었음.

대신 Java parser/syntax smoke test와 소스/설정/리소스 정합성 검사를 수행함.

## 학생 실습 전 필수 설정

1. ECR Account ID 수정
   - `.github/workflows/deploy_ordermsa_with_k8s.yml`
   - 각 서비스 `k8s/depl_svc.yml`
2. S3 Bucket 이름 수정
   - `msa/product/src/main/resources/application-prod.yml`
3. Secret 생성
   - `my-app-secrets`: `DB_HOST`, `DB_PW`
   - `aws-secret`: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
4. S3 `products/*` Object를 브라우저에서 읽을 수 있도록 Bucket Policy 또는 CloudFront 정책 구성
5. Route53 / cert-manager / Ingress 도메인 값이 실제 수업 도메인과 일치하는지 확인

