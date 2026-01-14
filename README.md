# COTP — Challenge-based One-Time Password

COTP is a **challenge-response–based authentication mechanism** designed to eliminate the inherent replay and phishing weaknesses of traditional time-based OTP systems.

Unlike passive OTP schemes that generate credentials in isolation,COTP enforces an **interactive, context-bound handshake** between client and server.

---

## Philosophy: Active Defense

Traditional One-Time Password (OTP) systems are fundamentally **passive**.

They generate a valid code based solely on a shared secret and a clock, regardless of **who is requesting it**, **from where**, or **for what interaction**.

In modern threat models—transparent phishing proxies, credential replay, and real-time interception—this passivity becomes the weakest link.

**COTP transforms authentication from a passive broadcast into an active handshake.**

---

## 1. The Challenge–Response Mandate

The core principle of COTP is simple:

> **No credential should exist in a vacuum.**

- In traditional OTP systems, the user provides a code.
- In COTP, the server first issues a **Challenge**:
  a unique, high-entropy, short-lived cryptographic value.
- The client’s proof is not merely a password,
  but a **targeted response** to that specific challenge.

Authentication succeeds only when the response is bound to **that exact challenge**, issued **by that server**, **for that interaction**.

---

## 2. Dynamic Contextual Binding

COTP rejects the assumption that **time alone is a sufficient security variable**.

By incorporating a server-generated challenge into the authentication process,
COTP binds user presence to a **specific interaction context** rather than
a generic time window.

### Redefining “Once”

In COTP, *“One-Time”* does **not** mean:

- “Once every 30 seconds”
- “Valid within a time slice”

It means:

> **“Valid only for this exact challenge, right here, right now.”**

A response generated for one challenge is cryptographically useless for any other.

---

## 3. Ephemeral Authorization (Zero-Persistence Trust)

COTP is designed around **ephemeral authorization**.

- Each challenge is valid for **exactly one successful verification**.
- Once consumed, the challenge is permanently invalidated.
- Any captured credential—challenge, proof, or full request—becomes immediately worthless after use.

This model assumes a hostile network environment and treats credential exposure as inevitable, but **non-repeatable**.

---

## 4. Minimal-State Integrity

COTP minimizes server-side state without sacrificing replay resistance.

While challenges may be cryptographically signed to prevent tampering,the server **must enforce a consume-once semantic** to guarantee true non-replayability.

This is typically achieved via:

- Short-lived, in-memory challenge tracking
- Atomic verification and invalidation
- Lock-free or minimal-lock data structures

This approach preserves horizontal scalability while ensuring that **no challenge can ever be successfully verified more than once**.

---

## 5. Replay Resistance Model

COTP explicitly defends against the following attack classes:

| Attack Type | Result |
|------------|--------|
| Time-window replay | ❌ Ineffective |
| Credential reuse | ❌ Impossible |
| Intercepted request replay | ❌ Rejected |
| Concurrent replay attempts | ❌ One succeeds at most |
| Cross-session reuse | ❌ Invalid |

Replay resistance is enforced **server-side**, not delegated to client behavior or timing assumptions.

---

## 6. Comparison with Existing Models

| Feature | TOTP | COTP |
|------|------|------|
| Credential generation | Passive | **Interactive** |
| Trust anchor | Time | **Server-issued challenge** |
| Replay window | Yes | **None** |
| Context binding | No | **Yes** |
| Challenge consumption | N/A | **Exactly once** |
| Authorization lifetime | Time-bound | **Interaction-bound** |

---

## 7. Relationship to WebAuthn

COTP and WebAuthn share a common philosophical foundation:
**challenge-response authentication with replay resistance**.

Key differences:

| Aspect | COTP | WebAuthn |
|-----|-----|-----|
| Hardware trust root | No | Yes |
| Private key extractability | Software-defined | Hardware-enforced |
| Deployment complexity | Low | Medium–High |
| Browser API dependency | None | Required |

COTP does **not** aim to replace WebAuthn.
COTP does not aim to replace WebAuthn. Instead, it represents an exploration of **pushing security boundaries to their limits within the constraints of pure HTTP and software-based environments**, offering a high-security alternative where hardware-backed trust roots are not viable.

---

## Summary

By replacing static time-based codes with a dynamic,
challenge-bound authentication flow, COTP ensures that
authorization is never reusable, never ambient, and never passive.

Authentication is no longer a broadcast.
It is a negotiation—**one that can only succeed once**.
