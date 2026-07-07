JWT Authentication Bypass via Unverified Signature

Description

This repository contains a technical walkthrough for a laboratory environment focusing on broken token validation. The vulnerability stems from a flaw where backend APIs rely on JSON Web Tokens (JWTs) for session management but fail to enforce cryptographic signature verification. When an API processes a token payload without validating it against its secret key, an attacker can modify token claims to bypass authentication controls entirely.

Exploit Steps

1. Token Interception
   
After authenticating with a standard user account, I used Burp Suite to intercept the baseline HTTP requests and capture the live session cookie containing the JWT structure (⁠Header.Payload.Signature⁠).

2. Payload Modification
   
Using the token editor, I left the cryptographic signature entirely unaltered but modified the decoded payload block. Specifically, I changed the ⁠"sub"⁠ (subject) claim from ⁠wiener⁠ to ⁠administrator⁠ to attempt vertical privilege escalation:

{
  "sub": "administrator",
  "iat": 1516239022
}

3. Signature Bypass Verification
   
The modified token was sent back to the server. Because the backend application skipped the integrity check on the third component of the JWT (the signature), it accepted the altered payload parameters at face value.

4. Admin Access & Lab Completion

With the forged administrative token active, I targeted the user management endpoint:
          GET /admin/delete?username=carlos HTTP/1.1

          <img width="802" height="325" alt="Screenshot 2026-07-07 044444" src="https://github.com/user-attachments/assets/ee7cf73c-b170-4c9a-9e59-7c196581e4be" />


The server executed the request under administrative context, successfully removing the user ⁠carlos⁠ and completing the challenge objectives.

<img width="858" height="328" alt="Screenshot 2026-07-07 044551" src="https://github.com/user-attachments/assets/e786f6d8-6038-483a-a13f-7dfe07faf682" />


Remediation

To mitigate this vulnerability, backend authentication handlers must never trust the header or payload data of a JWT before validating its signature. The application framework should be explicitly configured to reject any incoming tokens that fail integrity validation against the server's private or secret key.
