# COTP — Challenge-based One-Time Password

COTP is a **challenge-response–based authentication mechanism** designed to eliminate the inherent replay and phishing weaknesses of traditional time-based OTP systems.

Unlike passive OTP schemes that generate credentials in isolation, COTP enforces an **interactive, context-bound handshake** between client and server.

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
- In COTP, the server first issues a **challenge**:
  a unique, high-entropy, short-lived cryptographic value.
- The client’s proof is not merely a password,
  but a **targeted response** to that specific challenge.

Authentication succeeds only when the response is bound to **that exact challenge**, issued **by that server**, **for that interaction**.

---

## 2. Dynamic Contextual Binding

COTP rejects the assumption that **time alone is a sufficient security variable**.Authentication in COTP is bound to a **specific interaction context**, not a time window.

By tying each server-issued challenge to a **unique server identity**, a response becomes valid **only within the exact context and on the exact server where it was requested**.  
A response issued for one server, one challenge, or one interaction is cryptographically meaningless outside of that context.

This design prevents credentials from being transferable across servers or sessions, effectively mitigating replay and man-in-the-middle attacks.

### Redefining “Once”

In COTP, *“One-Time”* does **not** mean:

- “Once every 30 seconds”
- “Valid within a predefined time slice”

It means:

> **“Valid only for this exact interaction — on this server, for this challenge, at this moment.”**

A response generated for one context has no value in any other, even if fully intercepted by an attacker.

---

## 3. Ephemeral Authorization (Zero-Persistence Trust)

COTP operates on a model of **ephemeral authorization**.

Authorization is granted **only for the duration of a single successful verification**.

- Each challenge may be used exactly once.
- Upon successful verification, the challenge is permanently invalidated.
- No authorization state persists beyond the immediate interaction.

This model assumes a **hostile network environment**, where credential exposure is possible or even expected.  Instead of preventing exposure, COTP ensures that any captured data is **non-reusable, non-transferable, and immediately obsolete**.

Trust is never stored — it is momentarily proven, then discarded.

---

## 4. Minimal-State Integrity

COTP minimizes server-side state without sacrificing replay resistance.

While challenges may be cryptographically protected to prevent tampering, the server **must enforce a consume-once semantic** to guarantee true non-replayability.

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
| Trust root | Time | **Server-issued challenge** |
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
Instead, it represents an exploration of **pushing security boundaries to their limits within the constraints of pure HTTP and software-based environments**, offering a high-security alternative where hardware-backed trust roots are not viable.

---

## Summary

By replacing static time-based codes with a dynamic, challenge-bound authentication flow, COTP ensures that authorization is never reusable, never ambient, and never passive.

Authentication is no longer a broadcast.  
It is a negotiation — **one that can only succeed once**.
