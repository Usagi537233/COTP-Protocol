# COTP 在 [Zero-Trust-Lite (ZTL)](https://github.com/Usagi537233/Zero-Trust-Lite/tree/main) 文档

### [公共 COTP 计算器网页](https://cotp.537233.xyz/)

### 如果你是 [ipm专业站](https://ipm.537233.xyz/) 的用户，可以访问你自己的路径 `/cotp`，例如：`https://ipm.537233.xyz/yourpath/cotp`。

---

## 1. 介绍：挑战-响应认证

在 **[Zero-Trust-Lite (ZTL)](https://github.com/Usagi537233/Zero-Trust-Lite/tree/main)** 框架中，**COTP（基于挑战的一次性密码）** 是一种安全强化的认证方法。它采用 **挑战-响应（Challenge-Response）** 模型，确保每次登录尝试都是唯一的，并在特定会话上下文中进行加密验证。

---

## 2. 高级防钓鱼：`Challenge#Host` 逻辑

ZTL 实现了对 **中间人攻击（MITM）** 和 **钓鱼攻击** 的强防护机制，将最终的 COTP 验证绑定到目标主机名和系统身份令牌。

* **绑定机制（前端）**：当用户通过登录界面请求挑战时，前端会捕获当前浏览器的主机名（`window.location.host`），并将其附加到服务器生成的随机挑战上，然后形成字符串：`Challenge#TargetHost`。

* **验证流程（后端）**：

  * **独立验证**：当用户提交 8 位 **COTP** 响应时，ZTL 后端会结合三个独立元素进行验证：**随机挑战（Random Challenge）**、真实 **Host**（来自 HTTP `Host` 头）以及授权的 **TOKEN**（在 ZTL 启动时配置，每个部署实例唯一）。
  * **防钓鱼逻辑**：如果用户访问的是钓鱼网站，则计算时使用的 `$Host` 会不正确。即便用户提交了看似有效的验证码，后端验证仍会失败，因为计算结果必须与真实 `$Host` 和正确的 **TOKEN** 匹配。
  * **关键代理要求**：为了保证该防护生效，在反向代理（如 **Nginx**）后部署 ZTL 的管理员 **必须** 配置代理转发原始 Host 头，例如：`proxy_set_header Host $host;`。错误的 Host 转发或不当的跨域（CORS）设置可能破坏安全边界。

* **安全结果**：此设计确保 COTP 响应仅在同时匹配特定 **Challenge**、正确 **Host** 以及授权 **TOKEN** 时才有效。

---

## 3. 安全性与约束

* **一次性策略**：挑战在验证尝试（无论成功或失败）完成后，会立即从服务器内存 (`Challenges`) 中删除，以防止重放攻击。
* **可配置 TTL**：挑战的有效期（TTL）由运行 Zero-Trust-Lite 程序的用户配置。一旦 TTL 到期，挑战会自动失效。
