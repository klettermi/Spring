# Spring Security

<img width="1428" alt="스크린샷 2023-12-18 오전 12 21 01(2)" src="https://github.com/klettermi/Spring/assets/95194606/9c4022c1-c82b-4ce7-b673-2c752dde0281">

SecurityContextPersistenceFilter: SecurityContextRepository에서 SecurityContext를 가져오거나 생성

- SecurityContextRepository 클래스
    - 처음 인증하거나 혹은 익명 사용자일 경우
        - SecurityContext 생성 후 SecurityContextHolder 안에 저장을 하고 다음 필터 실행
    - 이력이 있을 경우
        - SecurityContext를 꺼내와서 SecurityContextHolder에 저장
- 최종적으로 클라이언트에 인증하기 직전에는 항상 ClearSecurityContext를 실행

LogoutFilter: 로그아웃 요청을 처리

UsernamePasswordAuthenticationFilter: ID와 PW를 사용하는 실제 Form 기반 유저 인증 처리

- Authentication 객체를 만들어 ID, PW를 저장하고 AuthenticationManager에게 인증 처리를 맡김
- AuthenticationManager가 검증 단계를 총괄하는 클래스인 AuthenticationProvider에게 인증 처리 위임
- AuthenticationProvider가 UserDetailsService와 같은 서비스를 사용해서 인증을 검증함
- 최종적으로 인증을 성공한 경우, 인증에 성공한 결과를 담은 Authentication를 생성한 다음 SecurityContext에 저장

ConcurrentSessionFilter: 동시 세션과 관련된 필터

- 매 요청마다 현재 사용자가 세션이 만료되었는지 확인
- session.expireNow 값으로 만료 설정을 구분

RememberMeAuthenticationFilter: 세션이 사라지거나 만료되더라도 쿠키 또는 DB를 사용하여 저장된 토큰 기반으로 인증 처리

- 세션이 만료되거나 무효화되어서 세션 안에 있는 SecurityContext 내의 인증 객체가 Null일 경우 작동
- Authentication이 null일 경우 현재 사용자가 요청하는 request header에 remember-me cookie 값을 헤더에 저장한 상태로 왔을 때 이 필터가 접속한 사용자 대신에 인증처리를 시도함

AnonymousAuthenticationFilter: 사용자 정보가 인증되지 않았다면 익명 사용자 토큰을 반환

SessionManagementFilter: 로그인 후 Session과 관련된 작업을 처리

- 조건이 현재 세션에 SecurityContext가 없거나 세션이 null인 경우에 동작
- 진행 작업
    - Register SessionInfo: 사용자의 세션 정보를 등록
    - SessionFixation: 세션 고정 보호로 인증에 성공한 시점에 새로운 쿠키가 발급되며, 인증을 시도하기 전에 이전 쿠키가 삭제되고 새로운 쿠키가 발급되도록 작동
    - ConcurrentSession: 사용자가 인증에 성공했다면 해당 사용자 계정으로 동시점에 세션이 존재하는지 확인, UsernamePasswordAuthenticationFilter인증을 진행하면서 동시 진행
        - 현재 사용자 인증 시도 차단
            - 인증 자체를 실행하지 못하도록 인증 관련 예외를 날림
            - SessionAuthenticationException을 던짐
        - 이전 사용자 인증 만료
            - 현재 사용자는 인증을 계속 사용하고, 이전 사용자의 세션 만료

ExceptionTranslationFilter: 필터 체인 내에서 발생되는 인증, 인가 예외를 처리

FilterSecurityInterceptor: 권한 부여와 관련한 결정을 AccessDecisionManager에게 위임해 권한 부여 결정 및 접근 제어 처리
