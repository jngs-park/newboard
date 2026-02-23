# 02. Auth & JWT (Security)

## 목표
- “로그인 상태”를 세션이 아니라 **JWT**로 유지
- 게시글 생성/수정/삭제 등 보호 API는 토큰 없으면 차단
- 작성자만 수정/삭제 가능하도록 **인가(authorization)** 적용

## 구성요소
- **AuthController**
  - `/api/auth/signup` : 회원가입
  - `/api/auth/login` : 로그인 + 토큰 발급
- **AuthService**
  - 회원 생성/검증 로직
- **PasswordEncoder(BCrypt)**
  - 비밀번호 단방향 해싱
- **JwtUtil/JwtTokenProvider**
  - 토큰 생성/검증/Subject(username) 추출
- **JwtAuthenticationFilter**
  - 요청마다 Authorization 헤더 검사 → 토큰 검증 → SecurityContext 설정
- **SecurityConfig**
  - permitAll / authenticated 정책 설정
  - FilterChain에 JwtAuthenticationFilter 추가

## 필수 규칙
### Authorization 헤더 형식
- `Authorization: Bearer <JWT>` (Bearer 접두어 필수)

## 테스트 시나리오
1) 회원가입 → 2) 로그인(token) → 3) 보호 API에 Bearer token → 200  
토큰 없이 보호 API → 401/403

## 흔한 장애 포인트
- PasswordEncoder Bean 누락 → 앱 부팅 실패
- JwtAuthenticationFilter Bean 등록 누락 → SecurityConfig 주입 실패
- `/api/auth/*` permitAll 누락 → 회원가입/로그인 막힘
