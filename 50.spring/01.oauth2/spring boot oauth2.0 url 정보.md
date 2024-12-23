해당 JSON은 OAuth 2.0 및 OpenID Connect에 따라 설정된 **Authorization Server의 메타데이터**입니다.
 이를 통해 서버가 지원하는 기능, 엔드포인트, 인증 방법 등을 확인할 수 있습니다.
 각 항목의 의미를 간단히 정리하면 다음과 같습니다.

------

### **Key 정보와 지원 기능**

#### **1. `issuer`**

- **값**: `"http://localhost:9000"`
- 이 Authorization Server의 **Base URL**로, 토큰 발급 및 인증 처리에 사용됩니다.

------

#### **2. `authorization_endpoint`**

- **값**: `"http://localhost:9000/oauth2/authorize"`
- 클라이언트 애플리케이션이 사용자에게 인증을 요청할 때 사용하는 엔드포인트입니다.
  - 예: Authorization Code Grant Flow에서 사용.
  - 브라우저로 리다이렉션되며 사용자 로그인 화면이 표시됩니다.

------

#### **3. `token_endpoint`**

- **값**: `"http://localhost:9000/oauth2/token"`
- 액세스 토큰을 발급받기 위한 엔드포인트입니다.
  - Authorization Code, Client Credentials, Refresh Token 등 다양한 Grant Type에서 사용.

------

#### **4. `token_endpoint_auth_methods_supported`**

- 지원되는 클라이언트 인증 방식:
  - `client_secret_basic`: 클라이언트 ID와 Secret을 Base64로 인코딩해 Authorization 헤더에 전달.
  - `client_secret_post`: 클라이언트 ID와 Secret을 POST 요청의 Body에 포함.
  - `client_secret_jwt`: 클라이언트 Secret을 사용해 JWT를 서명하여 인증.
  - `private_key_jwt`: 클라이언트의 개인 키로 JWT를 서명하여 인증.

------

#### **5. `jwks_uri`**

- **값**: `"http://localhost:9000/oauth2/jwks"`
- 서버가 사용하는 공개 키의 JSON Web Key Set (JWKS) 정보를 제공합니다.
  - JWT 서명을 검증하거나 클라이언트에서 ID 토큰을 확인할 때 사용.

------

#### **6. `response_types_supported`**

- 지원되는 응답 타입:
  - `"code"`: Authorization Code Grant Flow에서 사용되는 응답 타입.

------

#### **7. `grant_types_supported`**

- 지원되는 Grant Type:
  - `authorization_code`: 사용자 인증 및 승인을 통한 액세스 토큰 발급.
  - `client_credentials`: 클라이언트 자격 증명을 통한 액세스 토큰 발급.
  - `refresh_token`: 만료된 액세스 토큰을 갱신하기 위한 Refresh Token 발급.

------

#### **8. `revocation_endpoint`**

- **값**: `"http://localhost:9000/oauth2/revoke"`
- 기존의 액세스 토큰 및 Refresh Token을 **무효화(revoke)**하기 위한 엔드포인트입니다.

------

#### **9. `revocation_endpoint_auth_methods_supported`**

- 지원되는 인증 방식:
  - `client_secret_basic`
  - `client_secret_post`
  - `client_secret_jwt`
  - `private_key_jwt`

------

#### **10. `introspection_endpoint`**

- **값**: `"http://localhost:9000/oauth2/introspect"`
- 액세스 토큰의 상태(유효성, 만료 여부 등)를 확인하기 위한 엔드포인트입니다.

------

#### **11. `introspection_endpoint_auth_methods_supported`**

- 지원되는 인증 방식:
  - `client_secret_basic`
  - `client_secret_post`
  - `client_secret_jwt`
  - `private_key_jwt`

------

#### **12. `code_challenge_methods_supported`**

- 지원되는 Code Challenge 방식:
  - `"S256"`: PKCE(Proof Key for Code Exchange) 방식에서 SHA256 해싱을 사용하는 방법.

------

### **결론**

위 Authorization Server는 다음과 같은 기능을 지원합니다:

1. `Authorization Code`, `Client Credentials`, `Refresh Token`을 통한 토큰 발급.
2. 토큰 상태 확인(Introspection).
3. 토큰 무효화(Revocation).
4. PKCE 지원(SHA256).

이를 통해 **OAuth 2.0 클라이언트 애플리케이션**을 다양한 인증 플로우에 따라 구성할 수 있습니다.
 Refresh Token을 활용하려면 `authorization_code` Grant Type을 사용해야 하며, `client_credentials`는 Refresh Token을 제공하지 않습니다.