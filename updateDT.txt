# API Technical Documentation - Jejak Kebaikan Backend

## 1. Global API Info

- Base URL: `/api/v1`
- Main server (default): `http://localhost:3001`
- Full base URL (local): `http://localhost:3001/api/v1`
- Auth style:
  - Cookie `accessToken` (httpOnly), or
  - Header `Authorization: Bearer <token>`
- CSRF header (for endpoints protected by `CsrfGuard`):
  - `X-CSRF-Token: <token>`
- Get CSRF token from: `GET /api/v1/auth/csrf-token`

Security labels used in this document:
- `Public`: no authentication required
- `JWT`: requires authenticated user
- `JWT + Role(admin/user)`: requires JWT and role check
- `JWT + CSRF`: requires JWT and valid CSRF token
- `Skip CSRF`: endpoint explicitly bypasses CSRF guard

---

## 2. Root Endpoint

### GET /api/v1
- Description: Basic hello endpoint from root controller.
- Request JSON:
```json
{}
```
- Security: Public

---

## 3. Auth Module (`/auth`)

### GET /api/v1/auth/login
- Description: Generate ESS OAuth login URL and state.
- Request JSON:
```json
{}
```
- Query params: `force=true|false` (optional)
- Security: Public + throttled

### GET /api/v1/auth/ess/callback
- Description: OAuth callback from ESS, redirect to frontend with code and state.
- Request JSON:
```json
{}
```
- Query params: `code`, `state`
- Security: Public + throttled

### POST /api/v1/auth/ess/login
- Description: Exchange OAuth code and state, set auth cookies, return csrfToken.
- Request JSON:
```json
{
  "code": "authorization_code_from_ess",
  "state": "oauth_state_from_auth_login"
}
```
- Security: Skip CSRF (protected by OAuth state), throttled

### GET /api/v1/auth/me
- Description: Get current authenticated user profile.
- Request JSON:
```json
{}
```
- Security: JWT

### GET /api/v1/auth/csrf-token
- Description: Generate/return CSRF token for authenticated user.
- Request JSON:
```json
{}
```
- Security: JWT

### POST /api/v1/auth/logout
- Description: Logout user and clear auth cookies.
- Request JSON:
```json
{}
```
- Security: JWT + CSRF

### POST /api/v1/auth/refresh
- Description: Rotate refresh token and issue new access token + csrfToken.
- Request JSON:
```json
{}
```
- Security: Skip CSRF (automatic token rotation flow)

---

## 4. Campaign Module (`/campaign`)

### GET /api/v1/campaign/publish
- Description: List published campaigns (public view).
- Request JSON:
```json
{}
```
- Query params (optional): `page`, `limit`, `search`, `categoryId`, `sortBy`, `sortOrder`
- Security: Public + throttled

### GET /api/v1/campaign/closed
- Description: List closed campaigns (public view).
- Request JSON:
```json
{}
```
- Query params (optional): same pattern as publish list
- Security: Public + throttled

### GET /api/v1/campaign/publish/:slug
- Description: Get detail of published campaign by slug.
- Request JSON:
```json
{}
```
- Security: Public + throttled

### GET /api/v1/campaign/closed/:slug
- Description: Get detail of closed campaign by slug (includes disbursement docs).
- Request JSON:
```json
{}
```
- Security: Public + throttled

### GET /api/v1/campaign/my-campaigns
- Description: List campaigns owned by logged in user.
- Request JSON:
```json
{}
```
- Query params (optional): paging/filter/sort
- Security: JWT + Role(user/admin)

### GET /api/v1/campaign
- Description: Admin list all campaigns with filter.
- Request JSON:
```json
{}
```
- Query params (optional): paging/filter/sort
- Security: JWT + Role(admin)

### GET /api/v1/campaign/:slug
- Description: Get campaign detail by slug for admin/owner.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin/user)

### POST /api/v1/campaign
- Description: Create campaign step 1 (draft), image IDs must exist from upload service.
- Request JSON:
```json
{
  "title": "Bantuan Pendidikan Anak Yatim",
  "description": "Program bantuan biaya pendidikan untuk 100 anak yatim",
  "highlight": "Bantu wujudkan pendidikan berkualitas",
  "categoryId": 1,
  "targetAmount": 50000000,
  "imageIds": [1, 2, 3],
  "startDate": "2026-05-01",
  "endDate": "2026-12-31"
}
```
- Security: JWT + Role(admin/user) + CSRF

### POST /api/v1/campaign/:slug/recipient
- Description: Save recipient data (draft flow).
- Request JSON:
```json
{
  "recipientName": "Ahmad Suherman",
  "phone": "081234567890",
  "address": "Jl. Merdeka No. 123, Jakarta Pusat",
  "bankName": "Bank Mandiri",
  "accountNumber": "1234567890",
  "identityFileId": 10,
  "bankBookFileId": 11
}
```
- Security: JWT + Role(admin/user) + CSRF

### POST /api/v1/campaign/:slug/submit
- Description: Submit campaign for review/publish workflow.
- Request JSON:
```json
{
  "recipientName": "Ahmad Suherman",
  "phone": "081234567890",
  "address": "Jl. Merdeka No. 123, Jakarta Pusat",
  "bankName": "Bank Mandiri",
  "accountNumber": "1234567890",
  "identityFileId": 10,
  "bankBookFileId": 11
}
```
- Security: JWT + Role(admin/user) + CSRF

### GET /api/v1/campaign/:slug/recipient
- Description: Get recipient info for a campaign.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin/user)

### PATCH /api/v1/campaign/:slug
- Description: Update campaign.
- Request JSON:
```json
{
  "title": "Bantuan Pendidikan Anak Yatim - Update",
  "description": "Deskripsi terbaru",
  "highlight": "Highlight terbaru",
  "categoryId": 2,
  "targetAmount": 75000000,
  "imageIds": [4, 5],
  "startDate": "2026-06-01T00:00:00.000Z",
  "endDate": "2026-12-31T23:59:59.000Z"
}
```
- Security: JWT + Role(admin/user) + CSRF

### PATCH /api/v1/campaign/:slug/status
- Description: Admin update status campaign.
- Request JSON:
```json
{
  "status": "VERIFIED",
  "rejectionNote": "Isi jika status REJECTED"
}
```
- Security: JWT + Role(admin) + CSRF

### DELETE /api/v1/campaign/:slug
- Description: Delete campaign with role/status policy rules.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin/user) + CSRF

### POST /api/v1/campaign/test/auto-publish
- Description: Dev/staging endpoint to trigger auto publish cron manually.
- Request JSON:
```json
{}
```
- Security: Skip CSRF (environment restricted)

### POST /api/v1/campaign/test/auto-close-expired
- Description: Dev/staging endpoint to trigger auto close cron manually.
- Request JSON:
```json
{}
```
- Security: Skip CSRF (environment restricted)

---

## 5. Donation Module (`/donation`)

### POST /api/v1/donation
- Description: Create donation and create payment invoice.
- Request JSON:
```json
{
  "campaignSlug": "help-gaza-children",
  "amount": 100000,
  "donorType": "insan_kalla",
  "donorName": "John Doe",
  "donorEmail": "john@example.com",
  "donorPhone": "081234567890",
  "paymentPlatformId": 1,
  "isAnonymous": false,
  "message": "Semoga bermanfaat"
}
```
- Notes:
  - `donorName`, `donorEmail`, `donorPhone` required if `donorType=non_kalla`
- Security: JWT + CSRF

### POST /api/v1/donation/callback
- Description: Xendit webhook callback to update donation payment status.
- Request JSON:
```json
{
  "id": "invoice_id",
  "external_id": "donation_public_id",
  "status": "PAID",
  "paid_amount": 102500
}
```
- Required headers:
  - `x-callback-token`
  - `x-callback-signature`
- Security: Skip CSRF + callback token + HMAC signature

### GET /api/v1/donation/my-donations
- Description: Get current user donation history.
- Request JSON:
```json
{}
```
- Security: JWT

### GET /api/v1/donation/campaigns-list
- Description: List campaigns with donation statistics.
- Request JSON:
```json
{}
```
- Query params (optional): `title`, `endDate`
- Security: JWT

### GET /api/v1/donation/campaign/:campaignSlug
- Description: Get donations per campaign (admin or campaign owner).
- Request JSON:
```json
{}
```
- Query params (optional): `donorName`
- Security: JWT + CampaignAccessGuard

### GET /api/v1/donation/:publicId
- Description: Get donation detail by public ID.
- Request JSON:
```json
{}
```
- Security: JWT

---

## 6. Upload Module (`/upload`)

Note: Upload endpoints use `multipart/form-data`, not raw JSON. JSON below is request representation.

### POST /api/v1/upload/public
- Description: Upload public files (campaign/documentation).
- Request JSON (representation):
```json
{
  "files": ["<binary-file-1>", "<binary-file-2>"],
  "category": "campaign"
}
```
- Allowed category: `campaign`, `documentation`
- Security: JWT + CSRF

### POST /api/v1/upload/private
- Description: Upload private files (identity, bankbook, receipt, transfer_proof).
- Request JSON (representation):
```json
{
  "files": ["<binary-file-1>", "<binary-file-2>"],
  "category": "identity"
}
```
- Allowed category: `identity`, `bankbook`, `receipt`, `transfer_proof`
- Security: JWT + CSRF

### GET /api/v1/upload
- Description: List uploaded files owned by authenticated user.
- Request JSON:
```json
{}
```
- Security: JWT

### GET /api/v1/upload/file/:id
- Description: Get uploaded file detail.
- Request JSON:
```json
{}
```
- Security: JWT (owner/admin policy in service)

### GET /api/v1/upload/private/view/:id
- Description: View private file inline (supports ETag caching).
- Request JSON:
```json
{}
```
- Security: JWT (owner/admin)

### GET /api/v1/upload/private/:id
- Description: Download private file.
- Request JSON:
```json
{}
```
- Security: JWT (owner/admin)

### PATCH /api/v1/upload/public/:id
- Description: Replace existing public file.
- Request JSON (representation):
```json
{
  "file": "<binary-file>"
}
```
- Security: JWT + CSRF

### PATCH /api/v1/upload/private/:id
- Description: Replace existing private file.
- Request JSON (representation):
```json
{
  "file": "<binary-file>"
}
```
- Security: JWT + CSRF

### DELETE /api/v1/upload/:id
- Description: Delete uploaded file if not in use.
- Request JSON:
```json
{}
```
- Security: JWT + CSRF

---

## 7. Disbursement Module (`/disbursement`)

All disbursement endpoints are protected by JWT + Role(admin). Endpoints that modify data also require CSRF.

### POST /api/v1/disbursement
- Description: Step 1 create disbursement.
- Request JSON:
```json
{
  "campaignId": 1,
  "amount": 5000000,
  "purpose": "Pembangunan tahap 1",
  "useRecipientData": true,
  "recipientName": "Ahmad Suherman",
  "bankName": "Bank Mandiri",
  "bankAccountNumber": "1234567890"
}
```
- Security: JWT + Role(admin) + CSRF

### POST /api/v1/disbursement/:id/receipt
- Description: Step 2 link receipt file.
- Request JSON:
```json
{
  "receiptFileId": 123
}
```
- Security: JWT + Role(admin) + CSRF

### POST /api/v1/disbursement/:id/final-documentation
- Description: Step 3 link final documentation and transfer proof.
- Request JSON:
```json
{
  "documentationFileIds": [101, 102, 103],
  "recipientTransferFileId": 200
}
```
- Security: JWT + Role(admin) + CSRF

### PATCH /api/v1/disbursement/:id
- Description: Update disbursement during allowed status.
- Request JSON:
```json
{
  "amount": 5500000,
  "purpose": "Update kebutuhan",
  "useRecipientData": false,
  "recipientName": "Penerima Baru",
  "bankName": "Bank BNI",
  "bankAccountNumber": "999888777",
  "receiptFileId": 300
}
```
- Security: JWT + Role(admin) + CSRF

### GET /api/v1/disbursement/campaigns
- Description: List campaigns eligible for disbursement.
- Request JSON:
```json
{}
```
- Query params (optional): paging/filter from eligible query DTO
- Security: JWT + Role(admin)

### GET /api/v1/disbursement
- Description: List disbursements with filters/paging.
- Request JSON:
```json
{}
```
- Query params (optional): `campaignId`, `status`, `page`, `limit`, etc.
- Security: JWT + Role(admin)

### GET /api/v1/disbursement/campaign/:campaignId
- Description: List disbursements by campaign.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

### GET /api/v1/disbursement/:id
- Description: Get disbursement detail.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

---

## 8. Payment Platform Module (`/payment-platforms`)

### POST /api/v1/payment-platforms
- Description: Create payment platform.
- Request JSON:
```json
{
  "name": "OVO",
  "type": "EWALLET",
  "platformFeeType": "PERCENTAGE",
  "platformFeeValue": 2.5,
  "vatPercentage": 11,
  "isActive": true,
  "xenditChannelCode": "OVO"
}
```
- Security: JWT + Role(admin)

### GET /api/v1/payment-platforms
- Description: List payment platforms with filter and pagination.
- Request JSON:
```json
{}
```
- Query params (optional): `type`, `isSupportedByXendit`, `isActive`, `search`, `page`, `limit`
- Security: Public

### GET /api/v1/payment-platforms/active-for-payment
- Description: List active Xendit-supported platforms for checkout.
- Request JSON:
```json
{}
```
- Security: Public

### GET /api/v1/payment-platforms/xendit-channels
- Description: Get available Xendit channels.
- Request JSON:
```json
{}
```
- Query params (optional): `type`
- Security: Public

### GET /api/v1/payment-platforms/:id
- Description: Get payment platform detail.
- Request JSON:
```json
{}
```
- Security: Public

### PUT /api/v1/payment-platforms/:id
- Description: Update payment platform.
- Request JSON:
```json
{
  "name": "OVO Updated",
  "platformFeeValue": 3.0,
  "isActive": true
}
```
- Security: JWT + Role(admin)

### DELETE /api/v1/payment-platforms/:id
- Description: Soft delete payment platform (`is_active=false`).
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

---

## 9. Category Module (`/categories`)

### POST /api/v1/categories
- Description: Create category.
- Request JSON:
```json
{
  "name": "Kesehatan",
  "description": "Kampanye kesehatan dan pengobatan",
  "isActive": true
}
```
- Security: JWT + Role(admin)

### GET /api/v1/categories
- Description: List categories.
- Request JSON:
```json
{}
```
- Query params (optional): `isActive=true|false`
- Security: Public

### GET /api/v1/categories/:id
- Description: Get category detail.
- Request JSON:
```json
{}
```
- Security: Public

### PATCH /api/v1/categories/:id
- Description: Update category.
- Request JSON:
```json
{
  "name": "Pendidikan",
  "description": "Kategori pendidikan",
  "isActive": true
}
```
- Security: JWT + Role(admin)

### DELETE /api/v1/categories/:id
- Description: Delete category.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

---

## 10. Role Module (`/roles`)

All role endpoints require JWT + Role(admin).

### POST /api/v1/roles
- Description: Create role.
- Request JSON:
```json
{
  "name": "operator",
  "description": "Operator dashboard",
  "isActive": true
}
```
- Security: JWT + Role(admin)

### GET /api/v1/roles
- Description: List active roles.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

### GET /api/v1/roles/:id
- Description: Get role detail.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

### PATCH /api/v1/roles/:id
- Description: Update role.
- Request JSON:
```json
{
  "name": "operator-updated",
  "description": "Operator updated",
  "isActive": true
}
```
- Security: JWT + Role(admin)

### DELETE /api/v1/roles/:id
- Description: Delete role.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

---

## 11. User Management Module (`/user-management`)

All endpoints require JWT + Role(admin).

### GET /api/v1/user-management
- Description: List users with role info.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

### GET /api/v1/user-management/users-select
- Description: List minimal users for dropdown/select.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

### PUT /api/v1/user-management/:userId/role
- Description: Assign/update role for a user.
- Request JSON:
```json
{
  "roleId": 1
}
```
- Security: JWT + Role(admin) + CSRF

### GET /api/v1/user-management/:userId
- Description: Get user detail with role.
- Request JSON:
```json
{}
```
- Security: JWT + Role(admin)

---

## 12. Notification Module (`/notifications`)

All notification endpoints require JWT.

### GET /api/v1/notifications
- Description: List notifications for authenticated user.
- Request JSON:
```json
{}
```
- Query params (optional): `type`, `isRead`, `limit`, `offset`
- Security: JWT

### GET /api/v1/notifications/unread-count
- Description: Get unread notification count.
- Request JSON:
```json
{}
```
- Security: JWT

### PATCH /api/v1/notifications/:id/read
- Description: Mark one notification as read.
- Request JSON:
```json
{}
```
- Security: JWT

### PATCH /api/v1/notifications/read-all
- Description: Mark all notifications as read.
- Request JSON:
```json
{}
```
- Security: JWT

### DELETE /api/v1/notifications/:id
- Description: Delete notification.
- Request JSON:
```json
{}
```
- Security: JWT

---

## 13. Dashboard Module (`/dashboard`)

### GET /api/v1/dashboard
- Description: Get dashboard statistics summary.
- Request JSON:
```json
{}
```
- Security: JWT

---

## 14. Security Summary Per Endpoint Pattern

- Read-only public endpoints:
  - Campaign publish/closed public routes
  - Category GET routes
  - Payment platform public routes
- JWT only endpoints:
  - Profile, notifications, dashboard, several list/detail endpoints
- JWT + CSRF endpoints:
  - Most create/update/delete actions in auth/campaign/donation/upload/disbursement/user-management
- Webhook security endpoint:
  - `POST /api/v1/donation/callback` uses callback token + HMAC signature, CSRF skipped

---

## 15. Notes For Frontend Integration

- Always call `GET /api/v1/auth/csrf-token` after login refresh cycle before state-changing actions.
- For multipart endpoint, send `FormData` and do not set manual `Content-Type` boundary.
- Store ID references returned by upload endpoint (`fileId`) because many domain endpoints depend on those IDs.
- For donation callback, backend expects raw request body for signature verification.
