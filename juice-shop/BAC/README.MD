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
