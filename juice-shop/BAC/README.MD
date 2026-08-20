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
