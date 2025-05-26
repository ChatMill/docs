# JWT 认证与加密实践指南

本指南介绍 Chatmill 平台中 JWT（JSON Web Token）的字段规范、加密与验证示例代码，以及最佳实践。

---

## 1. JWT 在 Chatmill 中的用途

- 保障平台各端（捕获端、Agent 能力插件、发布端）之间接口通信的身份可信与数据安全。
- 用于 API、Webhook、事件流等所有敏感操作的认证与授权。
- 支持多平台、多 Agent、多租户安全扩展。

---

## 2. JWT payload 与 header 字段规范

### Header

| 字段名 | 说明      | 示例值      |
|--------|-----------|-------------|
| alg    | 签名算法  | HS256/RS256 |
| typ    | Token 类型 | JWT         |

**示例：**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

| 字段名      | 说明                                 | 必填 | 示例值                |
|-------------|--------------------------------------|------|----------------------|
| sub         | 主题（session_id，会话唯一标识）         | 是   | "sess-001"           |
| exp         | 过期时间（Unix 时间戳）                | 是   | 1718000000            |
| iat         | 签发时间（Unix 时间戳）                | 是   | 1717990000            |
| aud         | 受众（接收方微服务/agent唯一英文标识）   | 是   | "miss-spec-agent"    |
| iss         | 签发方（签发方微服务/agent唯一英文标识） | 是   | "discord-capture"    |

> aud 字段用于接收方身份校验，iss 字段用于签发方身份校验，所有字段均为必填，具体命名见[《平台适配与配置》](../deployment/PLATFORM_ADAPTERS.md)。

**payload 示例：**
```json
{
  "sub": "sess-001",
  "exp": 1718000000,
  "iat": 1717990000,
  "aud": "miss-spec-agent",
  "iss": "discord-capture"
}
```

---

## 3. JWT 签发与验证 Python 示例（PyJWT）

### 3.1 对称加密（HS256）

```python
import jwt
import time

# 签发 JWT
SECRET_KEY = "your_secret_key"
payload = {
    "sub": "sess-001",  # 主题为 session_id
    "exp": int(time.time()) + 3600,  # 1小时后过期
    "iat": int(time.time()),
    "aud": "miss-spec-agent",  # 接收方微服务唯一标识
    "iss": "discord-capture"   # 签发方微服务唯一标识
}
token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")
print(f"JWT Token: {token}")

# 验证 JWT
try:
    decoded = jwt.decode(
        token,
        SECRET_KEY,
        algorithms=["HS256"],
        audience="miss-spec-agent",
        issuer="discord-capture"
    )
    print("Decoded payload:", decoded)
except jwt.ExpiredSignatureError:
    print("Token expired!")
except jwt.InvalidTokenError:
    print("Invalid token!")
```

### 3.2 非对称加密（RS256）

```python
import jwt
from pathlib import Path
import time

# 读取密钥
private_key = Path("private.pem").read_text()
public_key = Path("public.pem").read_text()

# 签发 JWT
payload = {
    "sub": "sess-001",  # 主题为 session_id
    "exp": int(time.time()) + 3600,
    "iat": int(time.time()),
    "aud": "miss-spec-agent",  # 接收方微服务唯一标识
    "iss": "discord-capture"   # 签发方微服务唯一标识
}
token = jwt.encode(payload, private_key, algorithm="RS256")
print(f"JWT Token: {token}")

# 验证 JWT
try:
    decoded = jwt.decode(
        token,
        public_key,
        algorithms=["RS256"],
        audience="miss-spec-agent",
        issuer="discord-capture"
    )
    print("Decoded payload:", decoded)
except jwt.ExpiredSignatureError:
    print("Token expired!")
except jwt.InvalidTokenError:
    print("Invalid token!")
```

---

## 4. 适用场景与最佳实践

- 所有 API、Webhook、Agent 能力插件通信均必须启用 JWT 认证。
- 生产环境必须使用非对称加密（RS256），密钥分离，提升安全性。
- Token 有效期不宜过长，建议 1-2 小时，过期需重新签发。
- 必须校验 aud 字段，确保 JWT 只被指定微服务/agent 接收和处理。
- 必须校验 iss 字段，确保 JWT 只由指定微服务/agent 签发。
- 密钥/公钥应通过安全渠道分发与管理，严禁硬编码在代码仓库。

---

## 5. 相关文档引用

- [CML 协议与事件类型](./CML_PROTOCOL.md)
- [平台适配与配置](../deployment/PLATFORM_ADAPTERS.md)
- [架构与分层设计](./ARCHITECTURE(discord-capture).md)

---

如有更多安全需求或定制场景，请联系平台维护者。 