# COTP in [Zero-Trust-Lite (ZTL)](https://github.com/Usagi537233/Zero-Trust-Lite/tree/main) Documentation

### [Pubilc COTP Calculator Web](https://cotp.537233.xyz/)

### If you are a user of the [ipm pro site](https://ipm.537233.xyz/),you can access the your path/cotp ,just like https://ipm.537233.xyz/yourpath/cotp .



## 1. Introduction: Challenge-Response Authentication
In the **[Zero-Trust-Lite (ZTL)](https://github.com/Usagi537233/Zero-Trust-Lite/tree/main)** framework, **COTP (Challenge-based One-Time Password)** is a security-hardened authentication method. It utilizes a **Challenge-Response** model, ensuring that each login attempt is unique and cryptographically validated within a specific session context.

## 2. Advanced Anti-Phishing: The `Challenge#Host` Logic
ZTL implements a robust defense against **Man-in-the-Middle (MITM)** and **Phishing** attacks by binding the final COTP verification to the destination hostname and the system's identity token.

* **Binding Mechanism (Frontend)**: When a user requests a challenge via the login UI, the frontend captures the current browser hostname using `window.location.host` and appends it to the server-generated random challenge, then copy this string: `Challenge#TargetHost`.

* **Defense & Verification Flow (Backend)**:
    * **Independent Verification**: When the user submits the 8-digit **COTP** response, the ZTL backend performs a verification combining three distinct elements: the **Random Challenge**, the real **Host** (from `Host`), and the authorized **TOKEN** (configured during ZTL startup and unique per deployment instance).
    * **Anti-Phishing Logic**: If a user is on a phishing site, the `$Host` used for the calculation will be incorrect. Even if the user provides a valid-looking code, the backend will fail verification because the calculation must match the real `$Host` and the correct **TOKEN**.
    * **Critical Proxy Requirement**: For this defense to function correctly, administrators deploying ZTL behind a reverse proxy (like **Nginx**) **must** configure the proxy to pass the original Host header (e.g., `proxy_set_header Host $host;`).Incorrect Host headers settings or wrong cross-origin (CORS) settings will break the security boundary.

* **Security Result**: This design ensures that a COTP response is only valid when it matches the specific **Challenge**, the correct **Host**, and the authorized **TOKEN** simultaneously.

## 3. Security & Constraints
* **Single-Use Policy**: Challenges are deleted from the server's memory (`Challenges`) immediately after a verification attempt (successful or failed) to prevent replay attacks.
* **Customizable TTL**: The Time-to-Live (TTL) for challenges is configurable by the user running the Zero-Trust-Lite program. Once TTL expires, the challenge automatically becomes invalid.
