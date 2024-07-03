---
date: 2024-07-01
author: gpt-4o
---

JWT(JSON Web Token)는 인증 및 권한 부여를 위해 사용되는 토큰입니다. 그러나 JWT의 유효 기간이 만료되었을 때, 사용자의 지속적인 인증을 보장하기 위해 **Refresh Token**을 사용하는 것이 일반적입니다. Refresh Token을 사용하는 주요 이유는 다음과 같습니다:

#### 1. 보안 강화
Access Token의 유효 기간을 짧게 설정하여 만료되도록 함으로써, 토큰이 탈취되었을 때의 보안 위험을 줄일 수 있습니다. 짧은 유효 기간으로 인해 탈취된 토큰이 오래 사용될 가능성이 낮아집니다.

#### 2. 사용자 경험 향상
사용자가 로그인 세션을 지속적으로 유지할 수 있도록 합니다. Access Token이 만료되면, 사용자는 새로운 Access Token을 발급받기 위해 Refresh Token을 사용할 수 있습니다. 이를 통해 사용자는 재로그인할 필요 없이 지속적으로 서비스를 이용할 수 있습니다.

#### 3. 서버 부하 감소
매번 사용자 인증을 위해 사용자 자격 증명(예: 아이디와 비밀번호)을 검증하는 대신, Refresh Token을 통해 새 Access Token을 발급받을 수 있어 서버 부하가 감소합니다. 데이터베이스 조회 횟수를 줄여 성능을 향상시킵니다.

#### 4. 재사용 방지
Refresh Token은 일반적으로 한 번만 사용되도록 설계됩니다. 새로운 Access Token을 발급받을 때마다 새로운 Refresh Token이 발급되며, 이전의 Refresh Token은 폐기됩니다. 이를 통해 토큰 재사용을 방지할 수 있습니다.

#### 사용 예시
1. **로그인 시 Access Token과 Refresh Token 발급**:
   사용자가 로그인하면 서버는 Access Token과 Refresh Token을 발급합니다. Access Token은 짧은 유효 기간을 가지고 있으며, Refresh Token은 상대적으로 긴 유효 기간을 가집니다.

2. **Access Token 만료 시**:
   클라이언트는 만료된 Access Token 대신 Refresh Token을 서버에 전송하여 새로운 Access Token을 요청합니다.

3. **새로운 Access Token 발급**:
   서버는 Refresh Token의 유효성을 검증한 후, 새로운 Access Token을 발급합니다. 이 과정에서 새로운 Refresh Token도 함께 발급될 수 있습니다.

#### 결론
Refresh Token은 JWT 기반 인증 시스템에서 보안과 사용자 경험을 모두 향상시키기 위한 중요한 구성 요소입니다. 이를 통해 Access Token의 짧은 유효 기간이 가지는 보안적 이점을 유지하면서도 사용자의 불편을 최소화할 수 있습니다.