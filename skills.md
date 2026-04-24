---
name: PlayerTradeHub Admin API
description: Complete admin API capabilities for PlayerTradeHub marketplace. Use this skill when managing users, orders, listings, disputes, AI agents, analytics, settings, content, fraud, KYC, or any platform operations. Triggers on requests to manage anything in the admin dashboard.
---

# PlayerTradeHub Admin API

Complete API reference for all admin operations in PlayerTradeHub. Requires admin authentication via session.

## Authentication

Admin endpoints require an authenticated session. The session is established via the website's normal authentication flow (NextAuth.js).

For programmatic access, include session cookies in requests.

---

## 1. ANALYTICS & STATS

### GET /api/admin/stats
Returns dashboard statistics: revenue, orders, users, listings, system health, trends, and chart data.

**Query Params:** `range` (24h|7d|30d|90d), `metric` (revenue|orders|listings|users|sellers)

**Response:** `{ totalOrders, orderTrend, totalUsers, userTrend, totalRevenue, grossRevenue, revenueTrend, activeListings, listingTrend, activeGames, totalReports, recentOrders, revenueSources, revenueThisMonth, revenueLastMonth, pendingActions, chartData, systemHealth: { cpuUsage, memUsage } }`

---

### GET /api/admin/sidebar-stats
Returns pending action counts for admin sidebar.

**Response:** `{ pendingListings, pendingSellers, pendingWithdrawals, openDisputes, pendingEscrowRequests, pendingReports, pendingSupportRequests, unansweredChats, activeChatSessions }`

---

### GET /api/admin/seller-analytics
Returns detailed seller analytics for a period.

**Query Params:** `period` (number of days, default 30, max 365)

**Response:** `{ analytics: [{ id, name, email, totalSales, totalSalesValue, sellerFees, platformFees, netEarnings, salesCount, averageOrderValue, topCategories }], platformTotals }`

---

### GET /api/admin/vc-metrics
Returns e-commerce metrics: GMV, revenue, conversion rates, repeat buyer rates.

**Response:** `{ allTime: { orders, gmv, revenue, averageOrderValue, takeRate, totalUsers, purchaserRate, repeatBuyerRate, disputeRate, refundRate }, q1, currentQuarter, last30, checkout, monthlySeries }`

---

### GET /api/admin/insights
Returns user acquisition insights, geographic distribution, top games, ratings, NPS.

**Response:** `{ totals, acquisition, purpose, countries, games, sellingReasons, ratings, nps: { promoters, detractors, score, total }, avgRating, globeMarkers }`

---

### GET /api/admin/online-users
Returns live and historical user activity.

**Query Params:** `view` (overview|tracking), `range` (24h|7d|30d|all)

**Response:** `{ count, sellers, buyers, admins, guests, users, guestCountries, activityBreakdown, sourceBreakdownLive, sourceBreakdown24h, visitTimeline, hourlyBreakdown, peakHour }`

---

### GET /api/admin/geo-distribution
Returns geographic distribution of users.

**Query Params:** `range` (24h|7d|30d|all)

**Response:** `{ totalRequests, uniqueCountries, countryData: [{ country, count, percentage }] }`

---

### GET /api/admin/user-activity
Returns aggregated user activity statistics.

**Query Params:** `days` (default 30, max 365)

---

## 2. USER MANAGEMENT

### GET /api/admin/users
Returns all non-managed users with IP history, ban status, account details.

**Response:** `{ users: [{ id, name, email, role, createdAt, emailVerified, sellerStatus, banned, banReason, fakeBuyEnabled, howDidYouFindUs, country, ipLogs }] }`

---

### PATCH /api/admin/users
Update user's ban status.

**Request:** `{ userId: string, banned: boolean, banReason?: string }`

**Response:** `{ success: true, user: { id, name, email, role, banned, banReason } }`

---

### GET /api/admin/user-details
Returns detailed user info: balance, XP stats, total sales, IP history.

**Query Params:** `userId`

---

### GET /api/admin/sellers
Returns sellers with listing counts, status, performance metrics.

**Query Params:** `q`, `status`

---

### GET /api/admin/managed-sellers
Returns managed (internal) sellers with notes and trust status.

---

### POST /api/admin/managed-sellers
Create a new managed seller.

**Request:** `{ name: string, email?: string, image?, bio?, notes? }`

---

### PATCH /api/admin/managed-sellers/[id]
Update managed seller's profile and trust status.

**Request:** `{ name?, image?, bio?, notes?, isTrusted? }`

---

### DELETE /api/admin/managed-sellers/[id]
Deactivates all listings and deletes a managed seller.

---

### POST /api/admin/managed-sellers/[id]/reply
Send a message as a managed seller in a buyer conversation.

**Request:** `{ conversationId: string, body: string }`

---

### POST /api/admin/users/auto-fulfill-limits
Configure auto-fulfillment limits for a user.

**Request:** `{ userId: string, dailyLimit: number, lifetimeLimit: number }`

---

## 3. LISTINGS MANAGEMENT

### GET /api/admin/listings
Returns listings with filtering, search, pagination (excludes managed sellers).

**Query Params:** `q`, `status`, `hasLoginInfo` (true/false), `excludeKinguin` (true/false), `page`

**Response:** `{ listings, total, page, pages }`

---

### POST /api/admin/listings/[id]
Perform action on a listing: approve, reject, dismiss-report.

**Request:** `{ action: 'approve'|'reject'|'dismiss-report', reportId? }`

---

### DELETE /api/admin/listings/[id]
Delete a listing.

---

### POST /api/admin/listings/approve-all
Approve all pending listings in bulk.

---

### POST /api/admin/listings/[id]/promote
Promote a listing.

**Request:** `{ durationDays: 1|3|7|14|30|90 }`

---

### DELETE /api/admin/listings/[id]/promote
Remove promotion from a listing.

---

### GET /api/admin/listings/pending
Return all pending listings for review.

---

### GET /api/admin/listings/reported
Return reported listings.

---

### GET /api/admin/managed-listings
Return managed (scraped source) listings.

**Query Params:** `seller`, `status`, `q`, `page`

---

### PATCH /api/admin/managed-listings
Update status of managed listing.

**Request:** `{ id, status }`

---

### DELETE /api/admin/managed-listings
Delete managed listing.

**Query Params:** `id`

---

### GET /api/admin/key-inventory
Return key inventory for listings.

**Query Params:** `listingId` (optional)

---

### POST /api/admin/key-inventory
Add keys to listing inventory.

**Request:** `{ listingId: string, keys: string[] }` or `{ listingId, keysText: string }`

---

### DELETE /api/admin/key-inventory
Delete an unused key.

**Query Params:** `id`

---

## 4. ORDERS MANAGEMENT

### GET /api/admin/orders
Return orders with filtering, pagination, P&L statistics.

**Query Params:** `page`, `limit`, `status`, `buyerId`, `sellerId`, `search`, `sortBy`, `sortOrder`

**Response:** `{ orders, pagination: { currentPage, totalPages, totalItems, itemsPerPage }, stats: { totalOrders, totalEscrowHeld, statusDistribution, pnl } }`

---

### GET /api/admin/orders/[id]
Return detailed order info including Kinguin cost, payment method, delivery info.

---

### POST /api/admin/orders
Create a manual order (admin-assisted purchase or off-platform recovery).

**Request:** `{ listingId?: string, kinguinProductId?: string, buyerEmail: string, quantity?, amount?, status?, deliverNow?, forceKinguin? }`

---

### PUT /api/admin/orders
Update order status with valid transition checking.

**Request:** `{ orderId, status, escrowStatus?, reason? }`

---

### PATCH /api/admin/orders/[id]
Update order details (status, amount, currency, listing, wholesale price, or clear delivery).

**Request:** `{ status?, amount?, currency?, wholesalePrice?, listingTitle?, listingId?, silent?, clearDelivery? }`

---

### DELETE /api/admin/orders/[id]
Delete an order and related records.

---

### POST /api/admin/orders/[id]/release-escrow
Release escrow and credit seller's platform balance.

---

### POST /api/admin/orders/[id]/deliver
Manually deliver a redeem code to buyer.

**Request:** `{ code: string }`

---

### POST /api/admin/orders/[id]/fulfill-kinguin
Purchase and fulfill a Kinguin order with key polling.

**Request:** `{ offerId?, force?: boolean }`

---

## 5. DISPUTES & ESCROW

### GET /api/admin/disputes
Return disputes with pagination and status filtering.

**Query Params:** `status`, `page`, `limit`

---

### POST /api/admin/disputes/[id]
Handle dispute actions.

**Request:** `{ action, resolution?, refundAmount?, winnerId? }`

**Actions:**
- `in_progress`: Mark as under review, notify parties
- `resolve`: Resolve with winner, handle fund distribution (full refund, full release, or partial split)
- `cancel`: Cancel dispute, revert order to escrow hold
- `reopen`: Reopen a closed dispute
- `close`: Close without fund changes

---

### GET /api/admin/escrow/release-requests
Return all escrow release requests.

---

### PATCH /api/admin/escrow/release-requests
Approve or reject escrow release request.

**Request:** `{ requestId: string, status: 'APPROVED'|'REJECTED' }`

---

## 6. FINANCE & WITHDRAWALS

### GET /api/admin/withdrawals
Return withdrawals with filtering.

**Query Params:** `status`, `method`, `page`, `limit`

---

### POST /api/admin/withdrawals
Process a withdrawal.

**Request:** `{ withdrawalId: string, action: 'process'|'cancel'|'fail'|'complete', notes? }`

---

### POST /api/admin/test-email
Send test email via Brevo SMTP.

---

## 7. FRAUD & RISK

### GET /api/admin/fraud/orders
Return orders flagged with risk holds or disputed status.

---

### POST /api/admin/fraud/block
Add value to Stripe Radar block list, optionally ban user.

**Request:** `{ type: 'email'|'ip'|'card_fingerprint', value: string, userId? }`

---

### GET /api/admin/fraud/block
Get all blocked values from Stripe Radar.

---

### POST /api/admin/fraud/release-hold
Release risk hold on an order.

---

## 8. KYC (Know Your Customer)

### GET /api/admin/kyc
Return all KYC requests.

---

### GET /api/admin/kyc/[id]
Return full KYC details including images.

---

### PATCH /api/admin/kyc/[id]
Approve or reject KYC request.

**Request:** `{ action: 'approve'|'reject', adminNote? }`

---

## 9. CONTENT MANAGEMENT

### GET /api/admin/blog
Return blog posts.

**Query Params:** `q`, `page`, `limit`

---

### POST /api/admin/blog
Create blog post.

**Request:** `{ title: string, slug: string, content: string, summary?, image?, author?, tags?, published? }`

---

### GET /api/admin/blog/[id]
Return single blog post.

---

### PUT /api/admin/blog/[id]
Update blog post.

---

### DELETE /api/admin/blog/[id]
Delete blog post.

---

### GET /api/admin/games
Return games.

**Query Params:** `q`, `page`, `limit`

---

### POST /api/admin/games
Create or update game (upsert).

**Request:** `{ id?, name: string, description?, keywords?, coverImage? }`

---

### DELETE /api/admin/games
Delete game (must have no listings).

**Query Params:** `id`

---

### GET /api/admin/categories
Return all categories.

---

### POST /api/admin/categories
Create category.

**Request:** `{ name: string, gameId: string }`

---

### PUT /api/admin/categories/[id]
Update category.

**Request:** `{ name: string, gameId: string }`

---

### DELETE /api/admin/categories/[id]
Delete category.

---

## 10. SETTINGS & CONFIGURATION

### GET /api/admin/settings
Return site settings.

---

### POST /api/admin/settings
Update site settings.

**Request:** `{ siteName: string, maintenanceMode?, allowRegistration?, supportEmail?, twitterUrl?, facebookUrl?, instagramUrl?, discordUrl?, feePercentage? }`

---

### GET /api/admin/telegram
Return Telegram config.

---

### POST /api/admin/telegram
Save Telegram config or send test/summary messages.

**Request:** `{ action?: 'save'|'test'|'motivational'|'daily-summary', botToken?, chatId?, enabled? }`

---

### GET /api/admin/kinguin
Return Kinguin settings and stats.

---

### POST /api/admin/kinguin
Update Kinguin settings.

**Request:** `{ enabled?, defaultMarkupPct?, autoSync? }`

---

## 11. API KEYS

### GET /api/admin/api-keys
List all API keys.

---

### POST /api/admin/api-keys
Create new API key.

**Request:** `{ name: string, scopes: string[], twoFactorToken?, metadata? }`

**Response (raw key shown only once):** `{ apiKey: string, key: { id, name, prefix, scopes, createdAt } }`

---

### DELETE /api/admin/api-keys/[id]
Revoke API key.

**Request:** `{ twoFactorToken? }` (required if 2FA enabled)

---

## 12. AI OPERATIONS

### GET /api/admin/ai
Return available AI actions.

---

### POST /api/admin/ai
Process AI chat messages or execute actions.

---

### GET /api/admin/ai-config
Return current AI instructions.

---

### POST /api/admin/ai-config
Save AI instructions or test message.

**Request:** `{ action: 'save'|'test', instructions?, message? }`

---

### GET /api/admin/ai-manager
Return available AI manager actions.

---

### POST /api/admin/ai-manager
Process AI manager requests.

**Request:** `{ intent?: 'chat'|'execute', message?, history?, context?, actionId?, input?, confirmed? }`

---

### GET /api/admin/ai-ops
Return admin AI memory/ops state.

---

### POST /api/admin/ai-ops
Perform AI operations.

**Ops:** `create_task`, `mark_task_done`, `delete_task`, `upsert_template`, `delete_template`, `upsert_schedule`, `delete_schedule`, `pin_memory`, `forget_memory`, `clear_action_logs`, `run_due_schedules`

---

### GET /api/admin/ai-state
Return full AI state for admin.

---

### POST /api/admin/ai-state
Save AI state.

**Request:** `{ assistantMode?, qualityMode?, cmoPlaybook?, sessions?, learnedFacts?, templates?, schedules?, actionLogs?, tasks?, documents? }`

---

### GET /api/admin/ai/integrations
Return AI tool connections.

---

### POST /api/admin/ai/integrations
Save AI tool connections.

**Request:** `{ connections: { [tool]: { config } }, automationEnabled? }`

---

## 13. AI-ORG (Multi-Agent Organization)

### GET /api/admin/ai-org/agents
Return all AI agents.

---

### POST /api/admin/ai-org/agents
Create new AI agent.

**Request:** `{ name: string, role: string, status?, description?, modelProvider?, modelName?, heartbeatCron?, monthlyBudgetUsd?, endpointUrl?, endpointAuthToken? }`

---

### PATCH /api/admin/ai-org/agents
Update AI agent.

**Request:** `{ id: string, name?, role?, status?, description?, modelProvider?, modelName?, heartbeatCron?, monthlyBudgetUsd?, endpointUrl?, endpointAuthToken? }`

---

### DELETE /api/admin/ai-org/agents
Delete AI agent.

**Query Params:** `id`

---

### GET /api/admin/ai-org/agents/[agentId]/skills
Return skills attached to agent.

---

### POST /api/admin/ai-org/agents/[agentId]/skills
Attach skill to agent.

**Request:** `{ skillId: string }`

---

### DELETE /api/admin/ai-org/agents/[agentId]/skills/[skillId]
Remove skill from agent.

---

### GET /api/admin/ai-org/projects
Return AI projects.

---

### POST /api/admin/ai-org/projects
Create AI project.

**Request:** `{ title: string, description?, status?, goalId? }`

---

### PATCH /api/admin/ai-org/projects
Update project.

**Request:** `{ id: string, title?, description?, status?, goalId? }`

---

### DELETE /api/admin/ai-org/projects
Delete project.

**Query Params:** `id`

---

### GET /api/admin/ai-org/users
Return organization members.

---

### POST /api/admin/ai-org/users
Add user to organization.

**Request:** `{ email: string }`

---

### GET /api/admin/ai-org/tickets
Return AI tickets.

---

### POST /api/admin/ai-org/tickets
Create ticket.

**Request:** `{ title: string, description?, status?, priority?, projectId?, assignedAgentId? }`

---

### PATCH /api/admin/ai-org/tickets
Update ticket.

**Request:** `{ id: string, title?, description?, status?, priority?, projectId?, assignedAgentId? }`

---

### DELETE /api/admin/ai-org/tickets
Delete ticket.

**Query Params:** `id`

---

### GET /api/admin/ai-org/costs
Return AI cost events.

---

### POST /api/admin/ai-org/costs
Log cost event.

**Request:** `{ amountUsd: number, agentId?, source?, notes? }`

---

### GET /api/admin/ai-org/goals
Return AI goals.

---

### POST /api/admin/ai-org/goals
Create goal.

**Request:** `{ title: string, description?, status?, priority?, dueDate? }`

---

### PATCH /api/admin/ai-org/goals
Update goal.

**Request:** `{ id: string, title?, description?, status?, priority?, dueDate? }`

---

### DELETE /api/admin/ai-org/goals
Delete goal.

**Query Params:** `id`

---

### GET /api/admin/ai-org/heartbeats
Return AI agent heartbeats.

---

### POST /api/admin/ai-org/heartbeats
Create heartbeat.

**Request:** `{ agentId: string, cron: string, status? }`

---

### PATCH /api/admin/ai-org/heartbeats
Update heartbeat.

**Request:** `{ id: string, cron?, status?, lastRunAt?, nextRunAt? }`

---

### DELETE /api/admin/ai-org/heartbeats
Delete heartbeat.

**Query Params:** `id`

---

## 14. A/B TESTING

### GET /api/admin/ab-tests/experiments
Return all A/B experiments.

---

### POST /api/admin/ab-tests/experiments
Create experiment.

**Request:** `{ name: string, status?, type?, description?, hypothesis?, targetRoutes?, audienceRules?, primaryMetric?, secondaryMetrics?, holdbackPercent?, startAt?, endAt? }`

---

### GET /api/admin/ab-tests/experiments/[id]
Return experiment with variants.

---

### PATCH /api/admin/ab-tests/experiments/[id]
Update experiment.

---

### DELETE /api/admin/ab-tests/experiments/[id]
Delete experiment.

---

## 15. GAMIFICATION

### POST /api/admin/wheel-spin
Process wheel spin for user.

**Request:** `{ userId: string }`

**Response:** `{ userId, discountPercentage, wonDiscount, pointsWon, streakBonus, canSpin, spinCount }`

---

### GET /api/admin/wheel-spin/count
Return spin count and user info.

**Query Params:** `userId`

---

## 16. MISCELLANEOUS

### POST /api/admin/reprice-listings
Reprice all Kinguin listings using configured markup.

---

### POST /api/admin/google-merchant/sync
Sync products to Google Merchant Center.

---

### POST /api/admin/google-merchant/register
Register products with Google Merchant Center.

---

### GET /api/admin/ip-graph
Return IP address relationship graph.

---

### GET /api/admin/export-user-data
Export user data.

---

### POST /api/admin/fake-buy
Process fake buy for testing.

---

### POST /api/admin/users/[id]/fake-buy
Trigger fake buy for specific user.

---

### GET /api/admin/crypto-payments
Return cryptocurrency payment data.

---

### GET /api/admin/discount-codes
Manage discount codes.

---

### GET /api/admin/game-requests
Return game requests.

---

### GET /api/admin/security-review
Return security review data.

---

### GET /api/admin/sheets-export
Export data to Google Sheets.

---

### GET /api/admin/verify-role
Verify user role assignment.

---

## RESPONSE CODES

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 202 | Accepted (async processing) |
| 400 | Bad request / validation error |
| 401 | Unauthorized (not logged in) |
| 403 | Forbidden (not admin) |
| 404 | Not found |
| 409 | Conflict |
| 500 | Internal server error |

## ERROR RESPONSE FORMAT

```json
{
  "error": "Error message describing what went wrong"
}
```

## 2FA REQUIREMENTS

Some endpoints (api-keys management, Telegram config) require 2FA verification when enabled on admin dashboard. Include `twoFactorToken` in request body when required.

---

## USE CASES

### Ban a User
```bash
curl -X PATCH -H "Session: cookie" \
  -H "Content-Type: application/json" \
  -d '{"userId":"uuid","banned":true,"banReason":"Terms violation"}' \
  "/api/admin/users"
```

### Create an Order Manually
```bash
curl -X POST -H "Session: cookie" \
  -H "Content-Type: application/json" \
  -d '{"listingId":"uuid","buyerEmail":"buyer@example.com","quantity":1}' \
  "/api/admin/orders"
```

### Approve All Pending Listings
```bash
curl -X POST -H "Session: cookie" "/api/admin/listings/approve-all"
```

### Resolve a Dispute
```bash
curl -X POST -H "Session: cookie" \
  -H "Content-Type: application/json" \
  -d '{"action":"resolve","winnerId":"seller-uuid","resolution":"Seller provided proof"}' \
  "/api/admin/disputes/dispute-uuid"
```

### Create an AI Agent
```bash
curl -X POST -H "Session: cookie" \
  -H "Content-Type: application/json" \
  -d '{"name":"Support Bot","role":"support","description":"Handles customer inquiries"}' \
  "/api/admin/ai-org/agents"
```

### Update Site Settings
```bash
curl -X POST -H "Session: cookie" \
  -H "Content-Type: application/json" \
  -d '{"siteName":"PlayerTradeHub","feePercentage":5.5}' \
  "/api/admin/settings"
```

### Promote a Listing
```bash
curl -X POST -H "Session: cookie" \
  -H "Content-Type: application/json" \
  -d '{"durationDays":7}' \
  "/api/admin/listings/listing-uuid/promote"
```

### Release Order Escrow
```bash
curl -X POST -H "Session: cookie" "/api/admin/orders/order-uuid/release-escrow"
```
