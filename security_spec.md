# Security Specification: JPhone Web Admin Panel & Firestore Rules

## 1. System Invariants & Access Control Policy
1. **Zero-Trust Client Operations**: Ordinary client users cannot read other users' documents or private admin data.
2. **Authoritative Admin Gate**: An administrator is strictly verified by checking existence and active status in `/databases/$(database)/documents/admins/$(request.auth.uid)`. Client-supplied tokens or claims are never trusted alone.
3. **Immutability of Audit Trails**: Records in `/audit_logs/{logId}` can ONLY be written by verified active administrators and can NEVER be modified or deleted by anyone once created.
4. **Subscription Entitlement Integrity**: The user's `subscriptionTier` ('free' | 'premium') is authoritative. Client apps cannot self-promote to 'premium'. Only an active administrator (or verified server function) can modify `subscriptionTier` and `subscriptionExpiry`.
5. **PII and Identity Protection**: Plain user credentials, passwords, and raw tokens are never persisted in Firestore. Phone number, email, and verification states are guarded.

---

## 2. The "Dirty Dozen" Adversarial Payloads
The following 12 attack vectors represent common vulnerability patterns and must be guaranteed to return `PERMISSION_DENIED`:

1. **Ghost Admin Self-Registration**:
   - Attack: An unauthenticated or standard user writes `{ "id": "attackerUid", "role": "super_admin", "active": true }` into `/admins/attackerUid`.
   - Expected: `PERMISSION_DENIED`.

2. **Self-Upgrade to Premium**:
   - Attack: Standard authenticated user submits an update to their own document `/users/userId` with `{ "subscriptionTier": "premium" }`.
   - Expected: `PERMISSION_DENIED`.

3. **Orphaned Write / ID Poisoning**:
   - Attack: Document write attempt with a malicious non-alphanumeric or oversized (>128 chars) path key: `/users/../../evil_collection`.
   - Expected: `PERMISSION_DENIED`.

4. **Tampering with Audit Logs**:
   - Attack: Administrator or rogue user attempts an `update` or `delete` on an existing `/audit_logs/{logId}` record.
   - Expected: `PERMISSION_DENIED` (Audit logs are append-only; update/delete blocked).

5. **Cross-Tenant User Modification**:
   - Attack: Authenticated user A attempts to update `/users/userB`.
   - Expected: `PERMISSION_DENIED`.

6. **Shadow Field Injection**:
   - Attack: Submitting extra undeclared fields (e.g. `{ "isSuperUser": true }`) in `/users/userId` or `/settings/general`.
   - Expected: `PERMISSION_DENIED`.

7. **Unverified Email Impersonation**:
   - Attack: Attempting administrative actions with an auth token whose `email_verified == false`.
   - Expected: `PERMISSION_DENIED`.

8. **Blanket Query Scraping**:
   - Attack: Unauthenticated or non-admin user attempts a collection query `list /users` without administrator credentials.
   - Expected: `PERMISSION_DENIED`.

9. **Destructive App Settings Wipe**:
   - Attack: Non-admin or anonymous user attempts to write or delete `/settings/general`.
   - Expected: `PERMISSION_DENIED`.

10. **Plan Pricing Sabotage**:
    - Attack: Standard user or client attempts to write `{ "price": 0.00 }` to `/plans/{planId}`.
    - Expected: `PERMISSION_DENIED`.

11. **Spoofed Push Notification Injection**:
    - Attack: Non-admin user writes a fraudulent notification to `/notifications/{notificationId}`.
    - Expected: `PERMISSION_DENIED`.

12. **Denial-of-Wallet Long Payload Attack**:
    - Attack: Submitting string properties with 1MB junk text exceeding max string boundaries.
    - Expected: `PERMISSION_DENIED`.
