# ROA Claims API - cURL Commands

This document contains cURL commands for all API endpoints from the Postman collection.

---

## 🔐 Authentication

### 1. Get JWT Token for Admin

```bash
curl -X POST "http://localhost:5158/api/Auth" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "password"
  }'
```

**Response:**
```json
{
  "isSuccess": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresAt": "2026-01-30T01:32:33Z",
  "message": "Login Successful as Admin"
}
```

### 2. Get JWT Token for User

```bash
curl -X POST "http://localhost:5158/api/Auth" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "user",
    "password": "password"
  }'
```

---

## 📝 Claims Management

### 3. Create Claim (Admin)

```bash
curl -X POST "http://localhost:5158/api/Claims" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
  -d '{
    "policyNumber": "TKK-123",
    "claimantName": "Anil Pillai",
    "claimantEmail": "anil.pillai@solera.com",
    "incidentDate": "2026-01-25T00:04:05.112Z",
    "description": "test",
    "claimedAmount": 999999999.99,
    "tags": ["Q1"]
  }'
```

### 4. Create Claim (User)

```bash
curl -X POST "http://localhost:5158/api/Claims" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_USER_TOKEN" \
  -d '{
    "policyNumber": "TKK-1245",
    "claimantName": "Anil Pillai",
    "claimantEmail": "anil.pillai@solera.com",
    "incidentDate": "2026-01-25T00:00:05.112Z",
    "description": "test desc",
    "claimedAmount": 999999999.99,
    "tags": ["Q1"]
  }'
```

### 5. Get Claims Collection (with filters)

```bash
curl -X GET "http://localhost:5158/api/Claims?Status=0&DateFrom=2026-01-24T00:04:05.112Z&Page=1&PageSize=2&SortDescending=false" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Query Parameters:**
- `Status` - Filter by claim status (0=Submitted, 1=UnderReview, 2=Approved, 3=Denied)
- `DateFrom` - Filter claims from this date
- `DateTo` - Filter claims until this date (optional)
- `Page` - Page number (default: 1)
- `PageSize` - Items per page (default: 10)
- `SortDescending` - Sort order (default: false)

### 6. Get Claims Summary

```bash
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### 7. Get Single Claim by ID

```bash
curl -X GET "http://localhost:5158/api/Claims/16e24b96-a10c-49c0-9247-bef129dfc964" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Note:** Replace `16e24b96-a10c-49c0-9247-bef129dfc964` with actual claim ID.

### 8. Update Claim Status

```bash
curl -X PUT "http://localhost:5158/api/Claims/16e24b96-a10c-49c0-9247-bef129dfc964" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "status": 3,
    "notes": "Denied after review"
  }'
```

**Status Values:**
- `0` - Submitted
- `1` - UnderReview
- `2` - Approved
- `3` - Denied

### 9. Delete Claim

```bash
curl -X DELETE "http://localhost:5158/api/Claims/16e24b96-a10c-49c0-9247-bef129dfc964" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## 🧪 Testing Scenarios

### Scenario 1: Complete Admin Workflow

```bash
# Step 1: Login as admin
TOKEN=$(curl -s -X POST "http://localhost:5158/api/Auth" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}' | jq -r '.token')

# Step 2: Create a claim
CLAIM_ID=$(curl -s -X POST "http://localhost:5158/api/Claims" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "policyNumber": "POL-001",
    "claimantName": "John Doe",
    "claimantEmail": "john@example.com",
    "incidentDate": "2026-01-25T10:00:00Z",
    "description": "Car accident",
    "claimedAmount": 5000.00,
    "tags": ["Q1", "Auto"]
  }' | jq -r '.id')

# Step 3: Get the claim
curl -X GET "http://localhost:5158/api/Claims/$CLAIM_ID" \
  -H "Authorization: Bearer $TOKEN"

# Step 4: Update claim status
curl -X PUT "http://localhost:5158/api/Claims/$CLAIM_ID" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "status": 2,
    "notes": "Approved after review"
  }'

# Step 5: Get summary
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer $TOKEN"
```

### Scenario 2: User Workflow

```bash
# Step 1: Login as user
TOKEN=$(curl -s -X POST "http://localhost:5158/api/Auth" \
  -H "Content-Type: application/json" \
  -d '{"username":"user","password":"password"}' | jq -r '.token')

# Step 2: Create own claim
curl -X POST "http://localhost:5158/api/Claims" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "policyNumber": "POL-002",
    "claimantName": "Jane Smith",
    "claimantEmail": "jane@example.com",
    "incidentDate": "2026-01-26T14:30:00Z",
    "description": "Home damage",
    "claimedAmount": 10000.00,
    "tags": ["Q1", "Home"]
  }'

# Step 3: View own claims
curl -X GET "http://localhost:5158/api/Claims?Page=1&PageSize=10" \
  -H "Authorization: Bearer $TOKEN"
```

---

## 🔍 Testing Error Scenarios

### Test 1: Invalid Credentials (401)

```bash
curl -X POST "http://localhost:5158/api/Auth" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "wrongpassword"
  }' \
  -v
```

**Expected Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc7235#section-3.1",
  "title": "Authentication failed",
  "status": 401,
  "detail": "The username or password you entered is incorrect. Please check your credentials and try again.",
  "instance": "/api/auth",
  "traceId": "00-...",
  "timestamp": "2026-01-30T..."
}
```

### Test 2: Expired JWT Token (401)

```bash
# Use an expired or invalid token
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.EXPIRED_TOKEN" \
  -v
```

**Expected Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc7235#section-3.1",
  "title": "Unauthorized",
  "status": 401,
  "detail": "Your authentication token has expired. Please login again to get a new token.",
  "instance": "/api/claims/summary",
  "traceId": "00-...",
  "timestamp": "2026-01-30T..."
}
```

### Test 3: Missing JWT Token (401)

```bash
# No Authorization header
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -v
```

**Expected Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc7235#section-3.1",
  "title": "Unauthorized",
  "status": 401,
  "detail": "No authentication token was provided. Please include a valid JWT token in the Authorization header.",
  "instance": "/api/claims/summary",
  "traceId": "00-...",
  "timestamp": "2026-01-30T..."
}
```

### Test 4: Claim Not Found (404)

```bash
curl -X GET "http://localhost:5158/api/Claims/00000000-0000-0000-0000-000000000000" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -v
```

**Expected Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Claim not found",
  "status": 404,
  "detail": "The claim with ID '00000000-0000-0000-0000-000000000000' was not found or you don't have permission to access it.",
  "instance": "/api/claims/00000000-0000-0000-0000-000000000000",
  "traceId": "00-...",
  "timestamp": "2026-01-30T..."
}
```

### Test 5: Validation Error (400)

```bash
curl -X POST "http://localhost:5158/api/Claims" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "policyNumber": "",
    "claimantName": "",
    "claimantEmail": "invalid-email",
    "incidentDate": "2026-01-25T00:04:05.112Z",
    "description": "",
    "claimedAmount": -100,
    "tags": []
  }' \
  -v
```

**Expected Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "PolicyNumber": ["Policy number is required"],
    "ClaimantName": ["Claimant name is required"],
    "ClaimantEmail": ["Invalid email format"],
    "ClaimedAmount": ["Claimed amount must be greater than 0"]
  },
  "traceId": "00-...",
  "timestamp": "2026-01-30T..."
}
```

---

## 📝 Notes

### Using Variables in Bash

To avoid repeating the token in every request:

```bash
# Set the token as an environment variable
export TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# Use it in requests
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer $TOKEN"
```

### Pretty Print JSON Responses

Use `jq` to format JSON responses:

```bash
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer $TOKEN" | jq
```

### Save Response to File

```bash
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer $TOKEN" \
  -o response.json
```

### Include Response Headers

```bash
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer $TOKEN" \
  -i
```

### Verbose Output (for debugging)

```bash
curl -X GET "http://localhost:5158/api/Claims/summary" \
  -H "Authorization: Bearer $TOKEN" \
  -v
```

---

## 🔗 Quick Reference

| Endpoint | Method | Auth Required | Description |
|----------|--------|---------------|-------------|
| `/api/Auth` | POST | No | Get JWT token |
| `/api/Claims` | POST | Yes | Create new claim |
| `/api/Claims` | GET | Yes | Get claims list (with filters) |
| `/api/Claims/{id}` | GET | Yes | Get single claim |
| `/api/Claims/{id}` | PUT | Yes | Update claim status |
| `/api/Claims/{id}` | DELETE | Yes | Delete claim |
| `/api/Claims/summary` | GET | Yes | Get claims summary |

---

## 🎯 Common Query Parameters

### GET /api/Claims

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Status` | int | - | Filter by status (0-3) |
| `DateFrom` | datetime | - | Start date filter |
| `DateTo` | datetime | - | End date filter |
| `Page` | int | 1 | Page number |
| `PageSize` | int | 10 | Items per page |
| `SortDescending` | bool | false | Sort order |

---

## ✅ Summary

This document provides cURL commands for all 8 endpoints in the ROA Claims API:

1. ✅ **Authentication** - Get JWT tokens for admin and user
2. ✅ **Create Claims** - Both admin and user examples
3. ✅ **Get Claims** - List with filters and single claim
4. ✅ **Update Claims** - Status updates
5. ✅ **Delete Claims** - Remove claims
6. ✅ **Summary** - Get claims statistics
7. ✅ **Complete Workflows** - End-to-end scenarios
8. ✅ **Error Testing** - 401, 404, 400 examples

**All commands are ready to use - just replace `YOUR_TOKEN` with actual JWT tokens!** 🚀

