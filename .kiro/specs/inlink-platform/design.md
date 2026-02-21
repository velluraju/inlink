# Design Document: InLink Platform

## Overview

The InLink platform is a full-stack web application that connects influencers with brands for promotional collaborations. The system architecture follows a modern microservices-inspired approach with clear separation between frontend, backend API, real-time communication, AI services, and data persistence layers.

### High-Level Architecture

The platform consists of five primary layers:

1. **Presentation Layer**: Next.js 14 frontend with TypeScript, Tailwind CSS, and Shadcn UI components
2. **API Layer**: Node.js 20 + Express backend providing RESTful endpoints
3. **Real-Time Layer**: Socket.IO server for WebSocket-based chat and notifications
4. **AI Services Layer**: Google Cloud Platform services for verification and fraud detection
5. **Data Layer**: PostgreSQL 15 for relational data, Redis 7 for caching and session management

### Technology Stack Rationale

**Frontend (Next.js 14 + TypeScript)**:
- Server-side rendering for SEO optimization on public pages
- App Router for improved routing and layouts
- TypeScript for type safety and better developer experience
- Tailwind CSS for rapid UI development with consistent design system

**Backend (Node.js 20 + Express + TypeScript)**:
- Non-blocking I/O ideal for real-time features and high concurrency
- Large ecosystem of packages for payment gateways, AI integration
- TypeScript provides type safety across full stack
- Express offers flexibility for custom middleware and routing

**Database (PostgreSQL 15)**:
- ACID compliance critical for financial transactions
- Strong support for complex queries and relationships
- JSON column support for flexible data structures
- Proven reliability for production workloads

**Cache (Redis 7)**:
- Sub-millisecond latency for session storage
- Pub/Sub capabilities for real-time features
- Support for Socket.IO adapter for horizontal scaling
- TTL support for automatic cache expiration

**Real-Time (Socket.IO)**:
- Automatic fallback from WebSocket to HTTP long-polling
- Built-in reconnection logic
- Room-based broadcasting for efficient message routing
- Easy integration with Redis adapter for multi-server deployments

**AI Services (Google Cloud Platform)**:
- Vision API for OCR and face detection with high accuracy
- Natural Language API for chat content analysis
- Vertex AI for custom fraud detection models
- Scalable infrastructure with pay-per-use pricing

**Payment Gateway (PayU)**:
- Leading payment gateway in India with 70%+ market share
- Support for UPI, cards, net banking, wallets
- Competitive transaction fees
- Robust API with webhook support for payment status

### Design Principles

1. **Security First**: All sensitive data encrypted, JWT-based authentication, input validation, CSRF protection
2. **Scalability**: Stateless API design, Redis for distributed sessions, horizontal scaling capability
3. **Reliability**: Transaction atomicity, idempotent operations, comprehensive error handling
4. **Performance**: Database indexing, Redis caching, CDN for static assets, optimized queries
5. **Maintainability**: Clear separation of concerns, TypeScript interfaces, comprehensive logging


## Architecture

### System Context Diagram

```
┌─────────────┐         ┌──────────────────────────────────────┐
│   Browser   │────────▶│         InLink Platform              │
│  (Users)    │◀────────│                                      │
└─────────────┘         │  ┌────────────┐   ┌──────────────┐  │
                        │  │  Next.js   │   │   Express    │  │
┌─────────────┐         │  │  Frontend  │──▶│   Backend    │  │
│   Mobile    │────────▶│  └────────────┘   └──────────────┘  │
│  (PWA)      │◀────────│         │                 │          │
└─────────────┘         │         │                 │          │
                        │         ▼                 ▼          │
┌─────────────┐         │  ┌────────────┐   ┌──────────────┐  │
│   Admin     │────────▶│  │ Socket.IO  │   │  PostgreSQL  │  │
│   Panel     │◀────────│  │   Server   │   │   Database   │  │
└─────────────┘         │  └────────────┘   └──────────────┘  │
                        │         │                 │          │
                        │         │                 │          │
                        │         ▼                 ▼          │
                        │  ┌────────────┐   ┌──────────────┐  │
                        │  │   Redis    │   │   AWS S3     │  │
                        │  │   Cache    │   │   Storage    │  │
                        │  └────────────┘   └──────────────┘  │
                        └──────────────────────────────────────┘
                                   │
                        ┌──────────┴──────────┐
                        │                     │
                        ▼                     ▼
              ┌──────────────────┐  ┌──────────────────┐
              │  Google Cloud    │  │      PayU        │
              │  AI Services     │  │  Payment Gateway │
              └──────────────────┘  └──────────────────┘
```

### Component Architecture


**Frontend Components**:
- Public Website (20 pages): Marketing, legal, blog content
- User Dashboard: Role-specific views for influencers and brands
- Admin Panel: Verification queues, dispute resolution, monitoring
- Super Admin Panel: System configuration, financial analytics
- Chat Interface: Real-time messaging with file attachments and quotes

**Backend Services**:
- Authentication Service: JWT generation, OTP verification, 2FA
- User Service: Profile management, role-based access control
- Deal Service: Lifecycle management, state transitions
- Payment Service: PayU integration, transaction recording, payouts
- Chat Service: Message persistence, file upload handling
- Verification Service: Document processing, AI integration
- Notification Service: Email, SMS, in-app notifications
- Analytics Service: Dashboard metrics, reporting

**AI Services**:
- Identity Verification: OCR extraction, face matching, duplicate detection
- Channel Auto-Fetch: Social media API integration, metrics synchronization
- Fraud Detection: NLP-based chat monitoring, risk scoring, pattern detection

**Data Models**:
- Users: Influencers, brands, admins with role-based attributes
- Channels: Social media accounts with metrics history
- Deals: Complete lifecycle from negotiation to completion
- Campaigns: Auction-based opportunities with bidding
- Messages: Chat history with attachments and quotes
- Transactions: Payment records, payouts, refunds
- Verifications: Identity documents, confidence scores, audit trail

### Data Flow Patterns

**Authentication Flow**:
1. User submits credentials → Backend validates → Issues JWT access token (15min) + refresh token (7 days)
2. Client stores tokens → Includes access token in Authorization header for API requests
3. Access token expires → Client sends refresh token → Backend issues new access token
4. Refresh token expires → User must re-authenticate

**Deal Creation Flow**:
1. Brand posts campaign OR initiates direct chat with influencer
2. Influencer submits bid OR negotiates via chat
3. Parties agree on terms → Quote accepted → Deal created in "Negotiation" state
4. Final terms confirmed → Deal moves to "Awaiting Payment"
5. Brand pays via PayU → Webhook received → Deal moves to "Funded"
6. Influencer completes work → Submits proof → Deal moves to "Under Review"
7. Brand approves (or 48hr auto-approval) → Deal moves to "Approved"
8. Commission deducted → Payout queued → Deal moves to "Completed"

**Real-Time Chat Flow**:
1. User opens chat → Frontend establishes WebSocket connection with JWT
2. Socket.IO server validates JWT → Joins user to room based on conversation ID
3. User sends message → Server receives → Persists to database → Broadcasts to room
4. AI fraud detection analyzes message → Assigns risk score → Flags if threshold exceeded
5. Recipient receives message in real-time → Sends read receipt

**Verification Flow**:
1. Influencer uploads ID document + selfie → Files stored in S3
2. Backend calls Google Vision API for OCR → Extracts name, ID number, DOB
3. Backend calls Google Vision API for face detection on both images
4. Face matching algorithm compares embeddings → Calculates confidence score
5. Check for duplicate ID number in database
6. If confidence ≥99%: Auto-approve, grant 2 months Pro
7. If 90-98%: Queue for manual admin review
8. If <90%: Auto-reject, process refund


## Components and Interfaces

### Authentication Service

**Interface**:
```typescript
interface AuthService {
  register(email: string, password: string, role: UserRole): Promise<{userId: string, otpSent: boolean}>
  verifyOTP(userId: string, otp: string): Promise<{success: boolean, tokens?: TokenPair}>
  login(email: string, password: string): Promise<{success: boolean, tokens?: TokenPair, requires2FA?: boolean}>
  verify2FA(userId: string, code: string): Promise<{success: boolean, tokens?: TokenPair}>
  refreshToken(refreshToken: string): Promise<{accessToken: string}>
  requestPasswordReset(email: string): Promise<{success: boolean}>
  resetPassword(token: string, newPassword: string): Promise<{success: boolean}>
  logout(refreshToken: string): Promise<{success: boolean}>
}

interface TokenPair {
  accessToken: string  // JWT, 15 minutes expiry
  refreshToken: string // JWT, 7 days expiry
}

enum UserRole {
  INFLUENCER = 'influencer',
  BRAND_INDIVIDUAL = 'brand_individual',
  BRAND_BUSINESS = 'brand_business',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin'
}
```

**Implementation Details**:
- Password hashing: bcrypt with 12 salt rounds
- JWT signing: RS256 algorithm with private key
- OTP generation: 6-digit random number, 10-minute expiry
- OTP delivery: AWS SES for email, Twilio/MSG91 for SMS
- 2FA: TOTP (Time-based One-Time Password) using speakeasy library
- Rate limiting: 5 failed login attempts → 30-minute account lock
- Refresh token storage: Redis with 7-day TTL, blacklist on logout

### User Service

**Interface**:
```typescript
interface UserService {
  createProfile(userId: string, profileData: ProfileData): Promise<Profile>
  getProfile(userId: string): Promise<Profile>
  updateProfile(userId: string, updates: Partial<ProfileData>): Promise<Profile>
  deleteAccount(userId: string): Promise<{success: boolean}>
  linkChannel(userId: string, channel: ChannelLink): Promise<Channel>
  unlinkChannel(userId: string, channelId: string): Promise<{success: boolean}>
  getChannels(userId: string): Promise<Channel[]>
  calculateProfileStrength(userId: string): Promise<number>
}

interface ProfileData {
  name: string
  bio?: string
  avatar?: string
  location?: string
  niches?: string[]
  // Influencer-specific
  minDealAmount?: number
  maxDealAmount?: number
  // Brand-specific
  companyName?: string
  website?: string
  industry?: string
}

interface ChannelLink {
  platform: 'instagram' | 'youtube' | 'telegram' | 'linkedin' | 'twitter' | 'facebook'
  username: string
  profileUrl: string
}

interface Channel {
  id: string
  platform: string
  username: string
  profileUrl: string
  followerCount: number
  engagementRate: number
  postFrequency: number
  lastSyncedAt: Date
  metrics: ChannelMetrics[]
}

interface ChannelMetrics {
  date: Date
  followerCount: number
  engagementRate: number
  postFrequency: number
}
```

**Implementation Details**:
- Profile strength algorithm: Weighted score (0-100) based on:
  - Profile completeness: 20 points (name, bio, avatar, location)
  - Channel count: 20 points (5 points per channel, max 4)
  - Verification status: 30 points (verified = 30, pending = 10, unverified = 0)
  - Channel metrics: 30 points (follower count, engagement rate, consistency)
- Avatar storage: S3 with CloudFront CDN, max 5MB, formats: jpg, png, webp
- Soft delete: Mark account as deleted, retain data for 90 days, then hard delete


### Deal Service

**Interface**:
```typescript
interface DealService {
  createDeal(brandId: string, influencerId: string, terms: DealTerms): Promise<Deal>
  getDeal(dealId: string): Promise<Deal>
  updateDealState(dealId: string, newState: DealState, metadata?: any): Promise<Deal>
  submitPromotion(dealId: string, proof: PromotionProof): Promise<Deal>
  approveDeal(dealId: string, brandId: string): Promise<Deal>
  getDealsForUser(userId: string, filters?: DealFilters): Promise<Deal[]>
  cancelDeal(dealId: string, reason: string): Promise<{success: boolean}>
}

interface DealTerms {
  amount: number
  deliverables: string[]
  timeline: string
  additionalTerms?: string
}

enum DealState {
  NEGOTIATION = 'negotiation',
  AWAITING_PAYMENT = 'awaiting_payment',
  FUNDED = 'funded',
  PROMOTION_SUBMITTED = 'promotion_submitted',
  UNDER_REVIEW = 'under_review',
  APPROVED = 'approved',
  COMPLETED = 'completed',
  CANCELLED = 'cancelled',
  DISPUTED = 'disputed'
}

interface Deal {
  id: string
  brandId: string
  influencerId: string
  campaignId?: string
  state: DealState
  amount: number
  commissionRate: number
  commissionAmount: number
  netPayout: number
  deliverables: string[]
  timeline: string
  additionalTerms?: string
  createdAt: Date
  fundedAt?: Date
  submittedAt?: Date
  approvedAt?: Date
  completedAt?: Date
  stateHistory: StateTransition[]
}

interface StateTransition {
  fromState: DealState
  toState: DealState
  timestamp: Date
  triggeredBy: string
  metadata?: any
}

interface PromotionProof {
  screenshots: string[]  // S3 URLs
  postUrls: string[]
  description: string
}
```

**Implementation Details**:
- State machine validation: Only allow valid state transitions
- Auto-approval: Cron job runs every hour, checks deals in "Under Review" > 48 hours
- Commission calculation: 
  - Basic plan: amount * 0.01
  - Pro plan: amount * 0.005
- Transaction atomicity: Use database transactions for state changes + payment operations
- Deal limits enforcement:
  - Basic: Query active deals count before allowing new deal
  - Pro: No limit check needed


### Payment Service

**Interface**:
```typescript
interface PaymentService {
  createPaymentLink(dealId: string, amount: number, brandId: string): Promise<{paymentUrl: string, orderId: string}>
  handleWebhook(payload: PayUWebhook): Promise<{success: boolean}>
  recordTransaction(transaction: Transaction): Promise<Transaction>
  processRefund(transactionId: string, reason: string): Promise<{success: boolean, refundId: string}>
  queuePayout(dealId: string, influencerId: string, amount: number): Promise<Payout>
  processPayout(payoutId: string): Promise<{success: boolean, transactionRef: string}>
  getTransactionHistory(userId: string, filters?: TransactionFilters): Promise<Transaction[]}>
}

interface Transaction {
  id: string
  orderId: string
  dealId?: string
  userId: string
  type: 'payment' | 'payout' | 'refund' | 'subscription' | 'verification_fee'
  amount: number
  status: 'pending' | 'success' | 'failed' | 'refunded'
  paymentMethod?: string
  gatewayTransactionId?: string
  gatewayResponse?: any
  createdAt: Date
  completedAt?: Date
}

interface Payout {
  id: string
  dealId: string
  influencerId: string
  amount: number
  status: 'queued' | 'processing' | 'completed' | 'failed'
  bankAccount: BankAccount
  transactionRef?: string
  scheduledFor: Date
  processedAt?: Date
  failureReason?: string
}

interface BankAccount {
  accountNumber: string
  ifscCode: string
  accountHolderName: string
  bankName: string
}

interface PayUWebhook {
  status: string
  txnid: string
  amount: string
  productinfo: string
  firstname: string
  email: string
  mihpayid: string
  mode: string
  hash: string
}
```

**Implementation Details**:
- PayU integration:
  - Merchant key and salt from environment variables
  - Hash calculation: SHA512(key|txnid|amount|productinfo|firstname|email|||||||||salt)
  - Webhook verification: Recalculate hash and compare
  - Payment link expiry: 24 hours
- Payout processing:
  - Basic plan: Schedule for 5-7 business days after approval
  - Pro plan: Schedule for 2-3 business days after approval
  - Minimum threshold: ₹500 (hold smaller amounts)
  - Use NEFT/IMPS for bank transfers
  - Retry failed payouts up to 3 times with exponential backoff
- Reconciliation:
  - Daily cron job at 3 AM IST
  - Fetch PayU settlement report
  - Match against recorded transactions
  - Flag discrepancies for manual review
- Refund processing:
  - Call PayU refund API
  - Update transaction status
  - Send notification to user
  - Process within 5-7 business days


### Chat Service

**Interface**:
```typescript
interface ChatService {
  createConversation(userId1: string, userId2: string): Promise<Conversation>
  getConversation(conversationId: string): Promise<Conversation>
  sendMessage(conversationId: string, senderId: string, content: MessageContent): Promise<Message>
  getMessages(conversationId: string, pagination: Pagination): Promise<Message[]>
  markAsRead(conversationId: string, userId: string, messageId: string): Promise<{success: boolean}>
  sendQuote(conversationId: string, senderId: string, quote: Quote): Promise<Message>
  acceptQuote(messageId: string, userId: string): Promise<{success: boolean, dealId: string}>
}

interface Conversation {
  id: string
  participants: string[]  // User IDs
  lastMessage?: Message
  lastMessageAt?: Date
  createdAt: Date
}

interface Message {
  id: string
  conversationId: string
  senderId: string
  type: 'text' | 'file' | 'quote' | 'system'
  content: MessageContent
  readBy: string[]
  flagged: boolean
  riskScore?: number
  createdAt: Date
}

interface MessageContent {
  text?: string
  fileUrl?: string
  fileName?: string
  fileSize?: number
  quote?: Quote
}

interface Quote {
  amount: number
  deliverables: string[]
  timeline: string
  additionalTerms?: string
  expiresAt: Date
}

interface Pagination {
  limit: number
  offset: number
}
```

**Socket.IO Events**:
```typescript
// Client → Server
socket.emit('join_conversation', {conversationId: string})
socket.emit('send_message', {conversationId: string, content: MessageContent})
socket.emit('typing', {conversationId: string})
socket.emit('stop_typing', {conversationId: string})
socket.emit('mark_read', {conversationId: string, messageId: string})

// Server → Client
socket.on('message_received', (message: Message) => {})
socket.on('user_typing', {userId: string, conversationId: string})
socket.on('user_stopped_typing', {userId: string, conversationId: string})
socket.on('message_read', {messageId: string, userId: string})
socket.on('quote_accepted', {messageId: string, dealId: string})
```

**Implementation Details**:
- WebSocket authentication: Validate JWT in connection handshake
- Room management: Each conversation is a Socket.IO room
- Message persistence: Save to PostgreSQL before broadcasting
- File upload: Direct to S3 with pre-signed URL, max 10MB
- Allowed file types: jpg, png, pdf, doc, docx, xls, xlsx
- Message history: Load last 50 messages on conversation open, infinite scroll for older
- Typing indicators: Broadcast to room, auto-clear after 3 seconds of inactivity
- Read receipts: Update message.readBy array, broadcast to room
- Offline messages: Store in database, deliver when user reconnects


### Verification Service

**Interface**:
```typescript
interface VerificationService {
  submitVerification(userId: string, documents: VerificationDocuments): Promise<Verification>
  processVerification(verificationId: string): Promise<VerificationResult>
  getVerificationStatus(userId: string): Promise<Verification>
  approveVerification(verificationId: string, adminId: string): Promise<{success: boolean}>
  rejectVerification(verificationId: string, adminId: string, reason: string): Promise<{success: boolean}>
  checkDuplicateId(idNumber: string): Promise<{isDuplicate: boolean, existingUserId?: string}>
}

interface VerificationDocuments {
  idDocument: File  // Government ID (Aadhaar, PAN, Passport, Driver's License)
  selfie: File
}

interface Verification {
  id: string
  userId: string
  status: 'pending' | 'processing' | 'approved' | 'rejected' | 'manual_review'
  idDocumentUrl: string
  selfieUrl: string
  extractedData?: ExtractedData
  faceMatchConfidence?: number
  submittedAt: Date
  processedAt?: Date
  reviewedBy?: string
  rejectionReason?: string
}

interface ExtractedData {
  name: string
  idNumber: string
  dateOfBirth: string
  address?: string
  documentType: string
}

interface VerificationResult {
  success: boolean
  status: 'approved' | 'rejected' | 'manual_review'
  confidence: number
  reason?: string
}
```

**AI Processing Pipeline**:
1. **Document Upload**: Store in S3 with encryption
2. **OCR Extraction**: Call Google Vision API `DOCUMENT_TEXT_DETECTION`
   - Extract text blocks
   - Parse name, ID number, DOB using regex patterns
   - Validate extracted data format
3. **Face Detection**: Call Google Vision API `FACE_DETECTION` on both images
   - Extract face landmarks and embeddings
   - Ensure single face detected in each image
4. **Face Matching**: Calculate similarity score
   - Use cosine similarity between face embeddings
   - Confidence score: 0-100%
5. **Duplicate Check**: Query database for existing ID number
6. **Decision Logic**:
   - Confidence ≥99% AND no duplicate: Auto-approve
   - Confidence 90-98% AND no duplicate: Manual review queue
   - Confidence <90% OR duplicate found: Auto-reject
7. **Post-Approval Actions**:
   - Update user verification status
   - Grant 2 months free Pro plan (first-time only)
   - Send confirmation notification

**Implementation Details**:
- Google Cloud Vision API configuration:
  - Service account with Vision API permissions
  - Rate limit: 1800 requests per minute
  - Implement exponential backoff for rate limit errors
- Face matching algorithm:
  - Use dlib or face-api.js for embedding extraction
  - Cosine similarity threshold: 0.85 for 99% confidence
- Document validation:
  - Check image quality (resolution, blur, lighting)
  - Verify document authenticity markers
  - Flag suspicious patterns (photoshopped, screenshots)
- Manual review queue:
  - Admin dashboard shows pending verifications
  - Display side-by-side images with extracted data
  - Provide approve/reject buttons with reason field


### Fraud Detection Service

**Interface**:
```typescript
interface FraudDetectionService {
  analyzeMessage(message: Message): Promise<FraudAnalysis>
  analyzeConversation(conversationId: string): Promise<ConversationRisk>
  flagSuspiciousActivity(userId: string, activityType: string, metadata: any): Promise<{success: boolean}>
  getUserRiskProfile(userId: string): Promise<RiskProfile>
  applyPenalty(userId: string, penaltyType: PenaltyType, reason: string): Promise<{success: boolean}>
}

interface FraudAnalysis {
  riskScore: number  // 0-100
  flagged: boolean
  detectedPatterns: string[]
  keywords: string[]
  confidence: number
}

interface ConversationRisk {
  conversationId: string
  overallRiskScore: number
  flaggedMessages: string[]
  recommendedAction: 'none' | 'warn' | 'review' | 'suspend'
}

interface RiskProfile {
  userId: string
  totalRiskScore: number
  flaggedActivities: FlaggedActivity[]
  penaltyHistory: Penalty[]
  trustScore: number  // 0-100, inverse of risk
}

interface FlaggedActivity {
  id: string
  type: 'bypass_attempt' | 'suspicious_message' | 'fake_proof' | 'duplicate_account'
  description: string
  riskScore: number
  timestamp: Date
  reviewed: boolean
  reviewedBy?: string
}

enum PenaltyType {
  WARNING = 'warning',
  FIRST_BYPASS = 'first_bypass',      // 30 days + ₹5,000
  SECOND_BYPASS = 'second_bypass',    // 90 days + ₹15,000
  PERMANENT_BAN = 'permanent_ban'
}

interface Penalty {
  id: string
  type: PenaltyType
  reason: string
  suspensionDays?: number
  fineAmount?: number
  appliedAt: Date
  expiresAt?: Date
  paidAt?: Date
}
```

**NLP-Based Chat Monitoring**:

**Bypass Keywords Detection**:
```typescript
const bypassKeywords = {
  contactInfo: ['phone', 'whatsapp', 'telegram', 'email', 'instagram dm', 'direct message'],
  externalPayment: ['paytm', 'gpay', 'phonepe', 'bank transfer', 'cash', 'outside platform'],
  avoidance: ['off platform', 'direct deal', 'skip commission', 'no fee', 'private'],
  numbers: /\b\d{10}\b/,  // Phone numbers
  emails: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/
}
```

**Risk Scoring Algorithm**:
```typescript
function calculateRiskScore(message: string, userHistory: RiskProfile): number {
  let score = 0
  
  // Keyword matching
  if (containsContactInfo(message)) score += 30
  if (containsExternalPayment(message)) score += 40
  if (containsAvoidanceTerms(message)) score += 20
  if (containsPhoneNumber(message)) score += 25
  if (containsEmail(message)) score += 25
  
  // Context analysis using NLP
  const sentiment = analyzeSentiment(message)  // Google Natural Language API
  if (sentiment.urgency > 0.7) score += 10
  if (sentiment.secrecy > 0.7) score += 15
  
  // User history factor
  if (userHistory.penaltyHistory.length > 0) score += 20
  if (userHistory.flaggedActivities.length > 3) score += 15
  
  // Deal context
  if (isDealFunded(message.conversationId)) score += 10  // Higher risk after funding
  
  return Math.min(score, 100)
}
```

**Automated Actions**:
- Risk score 0-50: No action
- Risk score 51-70: Log for monitoring
- Risk score 71-85: Flag for admin review + warn users
- Risk score 86-100: Auto-warn users + immediate admin notification

**Post-Deal Content Monitoring**:
- Scrape influencer's social media posts after deal completion
- Check for undisclosed collaborations (missing #ad, #sponsored tags)
- Use image recognition to detect brand products/logos
- Compare post timing with deal timeline
- Flag suspicious patterns for admin review


### Channel Auto-Fetch Service

**Interface**:
```typescript
interface ChannelAutoFetchService {
  syncAllChannels(): Promise<SyncResult>
  syncChannel(channelId: string): Promise<ChannelMetrics>
  scheduleSync(): void
  getChannelMetrics(channelId: string, dateRange: DateRange): Promise<ChannelMetrics[]>
}

interface SyncResult {
  totalChannels: number
  successCount: number
  failureCount: number
  errors: SyncError[]
  duration: number
}

interface SyncError {
  channelId: string
  platform: string
  error: string
  retryCount: number
}

interface DateRange {
  startDate: Date
  endDate: Date
}
```

**Platform API Integrations**:

**Instagram**:
- API: Instagram Graph API (requires Business/Creator account)
- Metrics: follower_count, media_count, engagement (likes + comments / followers)
- Rate limit: 200 calls per hour per user
- Authentication: OAuth 2.0 with long-lived access tokens

**YouTube**:
- API: YouTube Data API v3
- Metrics: subscriberCount, videoCount, viewCount, average engagement
- Rate limit: 10,000 quota units per day
- Authentication: OAuth 2.0 with API key

**Twitter/X**:
- API: Twitter API v2
- Metrics: followers_count, tweet_count, engagement_rate
- Rate limit: 300 requests per 15 minutes
- Authentication: OAuth 2.0

**LinkedIn**:
- API: LinkedIn Marketing API
- Metrics: followerCount, postCount, engagement metrics
- Rate limit: Varies by endpoint
- Authentication: OAuth 2.0

**Facebook**:
- API: Facebook Graph API
- Metrics: fan_count, engagement, post_frequency
- Rate limit: 200 calls per hour per user
- Authentication: OAuth 2.0

**Telegram**:
- API: Telegram Bot API
- Metrics: member_count (for channels), post_frequency
- Rate limit: 30 messages per second
- Authentication: Bot token

**Sync Schedule**:
- Cron job: Daily at 2:00 AM IST
- Process channels in batches of 100
- Implement exponential backoff for rate limits
- Retry failed syncs up to 3 times
- Complete all syncs by 6:00 AM IST (4-hour window)

**Metric Calculations**:
```typescript
function calculateEngagementRate(platform: string, metrics: any): number {
  switch(platform) {
    case 'instagram':
      return ((metrics.likes + metrics.comments) / metrics.followers) * 100
    case 'youtube':
      return ((metrics.likes + metrics.comments) / metrics.views) * 100
    case 'twitter':
      return ((metrics.likes + metrics.retweets + metrics.replies) / metrics.followers) * 100
    case 'linkedin':
      return ((metrics.likes + metrics.comments + metrics.shares) / metrics.followers) * 100
    case 'facebook':
      return ((metrics.reactions + metrics.comments + metrics.shares) / metrics.fans) * 100
    case 'telegram':
      return (metrics.views / metrics.members) * 100
    default:
      return 0
  }
}

function calculatePostFrequency(posts: Post[], days: number): number {
  const recentPosts = posts.filter(p => isWithinDays(p.createdAt, days))
  return recentPosts.length / days
}
```

**Error Handling**:
- API rate limit exceeded: Queue for next available slot
- Invalid credentials: Notify user to re-authenticate
- Account deleted/private: Mark channel as inactive
- API downtime: Retry with exponential backoff, max 3 attempts
- Significant metric drop (>20%): Flag for admin review


### Campaign Service

**Interface**:
```typescript
interface CampaignService {
  createCampaign(brandId: string, campaign: CampaignData): Promise<Campaign>
  getCampaign(campaignId: string): Promise<Campaign>
  updateCampaign(campaignId: string, updates: Partial<CampaignData>): Promise<Campaign>
  closeCampaign(campaignId: string): Promise<{success: boolean}>
  searchCampaigns(filters: CampaignFilters): Promise<Campaign[]>
  submitBid(campaignId: string, influencerId: string, bid: Bid): Promise<Bid>
  getBids(campaignId: string): Promise<Bid[]>
  acceptBid(bidId: string, brandId: string): Promise<{success: boolean, dealId: string}>
  rejectBid(bidId: string, brandId: string, reason?: string): Promise<{success: boolean}>
}

interface CampaignData {
  title: string
  description: string
  budgetMin: number
  budgetMax: number
  targetPlatforms: string[]
  targetNiches: string[]
  targetFollowerRange: {min: number, max: number}
  deliverables: string[]
  timeline: string
  deadline: Date
  isPrivate: boolean
  invitedInfluencers?: string[]
}

interface Campaign {
  id: string
  brandId: string
  title: string
  description: string
  budgetMin: number
  budgetMax: number
  targetPlatforms: string[]
  targetNiches: string[]
  targetFollowerRange: {min: number, max: number}
  deliverables: string[]
  timeline: string
  deadline: Date
  status: 'active' | 'closed' | 'expired'
  isPrivate: boolean
  invitedInfluencers?: string[]
  bidCount: number
  createdAt: Date
  closedAt?: Date
}

interface Bid {
  id: string
  campaignId: string
  influencerId: string
  proposedAmount: number
  deliverables: string[]
  timeline: string
  pitch: string
  status: 'pending' | 'accepted' | 'rejected'
  submittedAt: Date
  respondedAt?: Date
}

interface CampaignFilters {
  platforms?: string[]
  niches?: string[]
  budgetMin?: number
  budgetMax?: number
  status?: string
}
```

**Implementation Details**:
- Campaign visibility:
  - Public campaigns: Visible to all verified influencers matching criteria
  - Private campaigns: Only visible to invited influencers
- Bid limits:
  - Basic plan: Max 5 active bids across all campaigns
  - Pro plan: Unlimited bids
- Campaign expiry:
  - Cron job runs hourly
  - Campaigns past deadline automatically closed
  - Pending bids auto-rejected with notification
- Bid ranking:
  - Sort by: influencer profile strength, engagement rate, proposed amount
  - Display influencer metrics alongside bid
- Notification triggers:
  - New campaign matching influencer criteria
  - New bid received on campaign
  - Bid accepted/rejected
  - Campaign closing soon (24 hours before deadline)


### Notification Service

**Interface**:
```typescript
interface NotificationService {
  sendNotification(userId: string, notification: NotificationData): Promise<{success: boolean}>
  sendBulkNotifications(notifications: BulkNotification[]): Promise<{successCount: number, failureCount: number}>
  getUserNotifications(userId: string, filters?: NotificationFilters): Promise<Notification[]>
  markAsRead(notificationId: string): Promise<{success: boolean}>
  updatePreferences(userId: string, preferences: NotificationPreferences): Promise<{success: boolean}>
}

interface NotificationData {
  type: NotificationType
  title: string
  message: string
  actionUrl?: string
  metadata?: any
}

enum NotificationType {
  DEAL_STATE_CHANGE = 'deal_state_change',
  NEW_BID = 'new_bid',
  BID_ACCEPTED = 'bid_accepted',
  BID_REJECTED = 'bid_rejected',
  PAYMENT_RECEIVED = 'payment_received',
  PAYOUT_PROCESSED = 'payout_processed',
  VERIFICATION_APPROVED = 'verification_approved',
  VERIFICATION_REJECTED = 'verification_rejected',
  NEW_MESSAGE = 'new_message',
  CAMPAIGN_MATCH = 'campaign_match',
  SUSPENSION = 'suspension',
  PENALTY = 'penalty'
}

interface Notification {
  id: string
  userId: string
  type: NotificationType
  title: string
  message: string
  actionUrl?: string
  metadata?: any
  read: boolean
  createdAt: Date
  readAt?: Date
}

interface NotificationPreferences {
  email: {
    dealUpdates: boolean
    messages: boolean
    payments: boolean
    marketing: boolean
  }
  inApp: {
    dealUpdates: boolean
    messages: boolean
    payments: boolean
  }
  sms: {
    criticalOnly: boolean
  }
}

interface BulkNotification {
  userId: string
  notification: NotificationData
}
```

**Delivery Channels**:

**Email (AWS SES)**:
- Transactional emails: Deal updates, payment confirmations, verification results
- Digest emails: Weekly summary of activity
- Marketing emails: New features, promotions (opt-in only)
- Templates: Use Handlebars for dynamic content
- Rate limit: 14 emails per second (AWS SES limit)
- Bounce handling: Mark email as invalid after 3 bounces

**In-App Notifications**:
- Real-time delivery via Socket.IO
- Persistent storage in PostgreSQL
- Badge count on notification icon
- Dropdown list with last 20 notifications
- Mark as read on click
- Auto-mark as read after 7 days

**SMS (Twilio/MSG91)**:
- Critical notifications only: Suspension, payment issues, security alerts
- OTP delivery for authentication
- Rate limit: 10 SMS per minute per user
- Cost optimization: Only send if email delivery fails or critical

**Push Notifications (Future - PWA)**:
- Web Push API for browser notifications
- Service worker for offline delivery
- Require user permission
- Respect notification preferences

**Implementation Details**:
- Queue-based delivery: Use Bull queue with Redis
- Retry logic: 3 attempts with exponential backoff
- Batch processing: Group notifications by user for digest emails
- Unsubscribe handling: One-click unsubscribe link in emails
- Notification history: Retain for 90 days, then archive


## Data Models

### Database Schema (PostgreSQL)

**Users Table**:
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL,
  email_verified BOOLEAN DEFAULT FALSE,
  phone VARCHAR(20),
  phone_verified BOOLEAN DEFAULT FALSE,
  two_fa_enabled BOOLEAN DEFAULT FALSE,
  two_fa_secret VARCHAR(255),
  status VARCHAR(50) DEFAULT 'active',
  suspension_end_date TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_status ON users(status);
```

**Profiles Table**:
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  bio TEXT,
  avatar_url VARCHAR(500),
  location VARCHAR(255),
  niches TEXT[],
  profile_strength INTEGER DEFAULT 0,
  verification_status VARCHAR(50) DEFAULT 'unverified',
  is_beta_user BOOLEAN DEFAULT FALSE,
  plan_type VARCHAR(50) DEFAULT 'basic',
  plan_expires_at TIMESTAMP,
  min_deal_amount INTEGER,
  max_deal_amount INTEGER,
  company_name VARCHAR(255),
  website VARCHAR(500),
  industry VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_profiles_user_id ON profiles(user_id);
CREATE INDEX idx_profiles_verification_status ON profiles(verification_status);
CREATE INDEX idx_profiles_plan_type ON profiles(plan_type);
```

**Channels Table**:
```sql
CREATE TABLE channels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  platform VARCHAR(50) NOT NULL,
  username VARCHAR(255) NOT NULL,
  profile_url VARCHAR(500) NOT NULL,
  follower_count INTEGER DEFAULT 0,
  engagement_rate DECIMAL(5,2) DEFAULT 0,
  post_frequency DECIMAL(5,2) DEFAULT 0,
  last_synced_at TIMESTAMP,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_channels_user_id ON channels(user_id);
CREATE INDEX idx_channels_platform ON channels(platform);
```

**Channel Metrics Table**:
```sql
CREATE TABLE channel_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  channel_id UUID REFERENCES channels(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  follower_count INTEGER NOT NULL,
  engagement_rate DECIMAL(5,2) NOT NULL,
  post_frequency DECIMAL(5,2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_channel_metrics_channel_id ON channel_metrics(channel_id);
CREATE INDEX idx_channel_metrics_date ON channel_metrics(date);
```

**Deals Table**:
```sql
CREATE TABLE deals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  brand_id UUID REFERENCES users(id) ON DELETE CASCADE,
  influencer_id UUID REFERENCES users(id) ON DELETE CASCADE,
  campaign_id UUID REFERENCES campaigns(id),
  state VARCHAR(50) NOT NULL,
  amount INTEGER NOT NULL,
  commission_rate DECIMAL(5,4) NOT NULL,
  commission_amount INTEGER NOT NULL,
  net_payout INTEGER NOT NULL,
  deliverables TEXT[] NOT NULL,
  timeline VARCHAR(255) NOT NULL,
  additional_terms TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  funded_at TIMESTAMP,
  submitted_at TIMESTAMP,
  approved_at TIMESTAMP,
  completed_at TIMESTAMP,
  cancelled_at TIMESTAMP,
  cancellation_reason TEXT
);

CREATE INDEX idx_deals_brand_id ON deals(brand_id);
CREATE INDEX idx_deals_influencer_id ON deals(influencer_id);
CREATE INDEX idx_deals_state ON deals(state);
CREATE INDEX idx_deals_created_at ON deals(created_at);
```

**Deal State History Table**:
```sql
CREATE TABLE deal_state_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID REFERENCES deals(id) ON DELETE CASCADE,
  from_state VARCHAR(50),
  to_state VARCHAR(50) NOT NULL,
  triggered_by UUID REFERENCES users(id),
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_deal_state_history_deal_id ON deal_state_history(deal_id);
```

**Campaigns Table**:
```sql
CREATE TABLE campaigns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  brand_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT NOT NULL,
  budget_min INTEGER NOT NULL,
  budget_max INTEGER NOT NULL,
  target_platforms TEXT[] NOT NULL,
  target_niches TEXT[],
  target_follower_min INTEGER,
  target_follower_max INTEGER,
  deliverables TEXT[] NOT NULL,
  timeline VARCHAR(255) NOT NULL,
  deadline TIMESTAMP NOT NULL,
  status VARCHAR(50) DEFAULT 'active',
  is_private BOOLEAN DEFAULT FALSE,
  invited_influencers UUID[],
  bid_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  closed_at TIMESTAMP
);

CREATE INDEX idx_campaigns_brand_id ON campaigns(brand_id);
CREATE INDEX idx_campaigns_status ON campaigns(status);
CREATE INDEX idx_campaigns_deadline ON campaigns(deadline);
```

**Bids Table**:
```sql
CREATE TABLE bids (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
  influencer_id UUID REFERENCES users(id) ON DELETE CASCADE,
  proposed_amount INTEGER NOT NULL,
  deliverables TEXT[] NOT NULL,
  timeline VARCHAR(255) NOT NULL,
  pitch TEXT NOT NULL,
  status VARCHAR(50) DEFAULT 'pending',
  submitted_at TIMESTAMP DEFAULT NOW(),
  responded_at TIMESTAMP
);

CREATE INDEX idx_bids_campaign_id ON bids(campaign_id);
CREATE INDEX idx_bids_influencer_id ON bids(influencer_id);
CREATE INDEX idx_bids_status ON bids(status);
```


**Conversations Table**:
```sql
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  participant_1 UUID REFERENCES users(id) ON DELETE CASCADE,
  participant_2 UUID REFERENCES users(id) ON DELETE CASCADE,
  last_message_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_conversations_participant_1 ON conversations(participant_1);
CREATE INDEX idx_conversations_participant_2 ON conversations(participant_2);
CREATE UNIQUE INDEX idx_conversations_participants ON conversations(LEAST(participant_1, participant_2), GREATEST(participant_1, participant_2));
```

**Messages Table**:
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
  sender_id UUID REFERENCES users(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL,
  content JSONB NOT NULL,
  read_by UUID[],
  flagged BOOLEAN DEFAULT FALSE,
  risk_score INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_messages_sender_id ON messages(sender_id);
CREATE INDEX idx_messages_created_at ON messages(created_at);
CREATE INDEX idx_messages_flagged ON messages(flagged) WHERE flagged = TRUE;
```

**Transactions Table**:
```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id VARCHAR(255) UNIQUE NOT NULL,
  deal_id UUID REFERENCES deals(id),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL,
  amount INTEGER NOT NULL,
  status VARCHAR(50) NOT NULL,
  payment_method VARCHAR(100),
  gateway_transaction_id VARCHAR(255),
  gateway_response JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP
);

CREATE INDEX idx_transactions_order_id ON transactions(order_id);
CREATE INDEX idx_transactions_deal_id ON transactions(deal_id);
CREATE INDEX idx_transactions_user_id ON transactions(user_id);
CREATE INDEX idx_transactions_type ON transactions(type);
CREATE INDEX idx_transactions_status ON transactions(status);
```

**Payouts Table**:
```sql
CREATE TABLE payouts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID REFERENCES deals(id) ON DELETE CASCADE,
  influencer_id UUID REFERENCES users(id) ON DELETE CASCADE,
  amount INTEGER NOT NULL,
  status VARCHAR(50) DEFAULT 'queued',
  bank_account_number VARCHAR(255) NOT NULL,
  ifsc_code VARCHAR(20) NOT NULL,
  account_holder_name VARCHAR(255) NOT NULL,
  bank_name VARCHAR(255) NOT NULL,
  transaction_ref VARCHAR(255),
  scheduled_for TIMESTAMP NOT NULL,
  processed_at TIMESTAMP,
  failure_reason TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_payouts_deal_id ON payouts(deal_id);
CREATE INDEX idx_payouts_influencer_id ON payouts(influencer_id);
CREATE INDEX idx_payouts_status ON payouts(status);
CREATE INDEX idx_payouts_scheduled_for ON payouts(scheduled_for);
```

**Verifications Table**:
```sql
CREATE TABLE verifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  status VARCHAR(50) DEFAULT 'pending',
  id_document_url VARCHAR(500) NOT NULL,
  selfie_url VARCHAR(500) NOT NULL,
  extracted_data JSONB,
  face_match_confidence DECIMAL(5,2),
  submitted_at TIMESTAMP DEFAULT NOW(),
  processed_at TIMESTAMP,
  reviewed_by UUID REFERENCES users(id),
  rejection_reason TEXT
);

CREATE INDEX idx_verifications_user_id ON verifications(user_id);
CREATE INDEX idx_verifications_status ON verifications(status);
```

**Flagged Activities Table**:
```sql
CREATE TABLE flagged_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL,
  description TEXT NOT NULL,
  risk_score INTEGER NOT NULL,
  reviewed BOOLEAN DEFAULT FALSE,
  reviewed_by UUID REFERENCES users(id),
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_flagged_activities_user_id ON flagged_activities(user_id);
CREATE INDEX idx_flagged_activities_reviewed ON flagged_activities(reviewed);
CREATE INDEX idx_flagged_activities_risk_score ON flagged_activities(risk_score);
```

**Penalties Table**:
```sql
CREATE TABLE penalties (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL,
  reason TEXT NOT NULL,
  suspension_days INTEGER,
  fine_amount INTEGER,
  applied_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP,
  paid_at TIMESTAMP
);

CREATE INDEX idx_penalties_user_id ON penalties(user_id);
CREATE INDEX idx_penalties_type ON penalties(type);
```

**Notifications Table**:
```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL,
  title VARCHAR(255) NOT NULL,
  message TEXT NOT NULL,
  action_url VARCHAR(500),
  metadata JSONB,
  read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  read_at TIMESTAMP
);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_read ON notifications(read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at);
```

### Redis Data Structures

**Session Storage**:
```
Key: session:{userId}
Type: Hash
Fields: {accessToken, refreshToken, expiresAt, deviceInfo}
TTL: 7 days
```

**Rate Limiting**:
```
Key: ratelimit:{userId}:{endpoint}
Type: String (counter)
TTL: 60 seconds
```

**Cache - User Profiles**:
```
Key: profile:{userId}
Type: Hash
TTL: 1 hour
```

**Cache - Channel Metrics**:
```
Key: channel:{channelId}:metrics
Type: Hash
TTL: 24 hours
```

**Socket.IO Adapter**:
```
Key: socket.io#{namespace}#{room}
Type: Set (socket IDs)
```

**Payout Queue**:
```
Key: payout:queue
Type: Sorted Set (score = scheduled timestamp)
```


## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

The following properties are derived from the requirements and will be validated through property-based testing. Each property uses universal quantification ("for all" or "for any") to ensure correctness across all possible inputs, not just specific examples.

### Authentication and Authorization Properties

**Property 1: Registration and OTP Flow**
*For any* valid email and password combination, when a user registers, the system should create an account, generate a valid OTP, and activate the account when the correct OTP is provided within the time window.
**Validates: Requirements 1.1, 1.2**

**Property 2: Token Refresh Round-Trip**
*For any* valid refresh token, the system should be able to generate a new access token, and that new access token should be valid for authentication.
**Validates: Requirements 1.3, 1.4**

**Property 3: Password Validation**
*For any* string, the password validation function should correctly identify whether it meets all requirements (8+ characters, uppercase, lowercase, number, special character).
**Validates: Requirements 1.8**

**Property 4: Password Hashing Consistency**
*For any* password, hashing it twice with bcrypt should produce different hashes, but both hashes should verify successfully against the original password.
**Validates: Requirements 22.2**

### Profile and Channel Properties

**Property 5: Profile Strength Calculation Bounds**
*For any* influencer profile with any combination of channels and metrics, the calculated profile strength score should always be between 0 and 100 inclusive.
**Validates: Requirements 2.5**

**Property 6: Zero Channels Exclusion**
*For any* search query, the results should never include influencer profiles that have zero active channels.
**Validates: Requirements 2.8**

**Property 7: Verified Influencers Only**
*For any* brand search without explicit filters, the results should only include influencers with verification status "verified".
**Validates: Requirements 6.1**

**Property 8: Search Result Ranking Consistency**
*For any* set of influencer profiles, the ranking algorithm should produce a consistent ordering when given the same profiles multiple times.
**Validates: Requirements 6.7**

### Verification Properties

**Property 9: Confidence Threshold Auto-Approval**
*For any* verification submission with face matching confidence ≥99% and no duplicate ID, the system should auto-approve the verification.
**Validates: Requirements 3.3**

**Property 10: Duplicate ID Rejection**
*For any* ID number that already exists in the verified accounts database, a new verification submission with that ID should be rejected.
**Validates: Requirements 3.6**

### Subscription and Plan Properties

**Property 11: Plan Transitions Update Commission**
*For any* influencer, when their plan changes from Basic to Pro or Pro to Basic, their commission rate should update to 0.5% or 1% respectively.
**Validates: Requirements 4.3, 4.4**

**Property 12: Basic Plan Limits Enforcement**
*For any* influencer on Basic plan with 3 active deals, attempting to create a new deal should be rejected. Similarly, with 5 active bids, new bid submissions should be rejected.
**Validates: Requirements 4.5, 7.4**

**Property 13: Beta User Benefits**
*For any* user marked as beta user, completing verification should grant exactly 6 months of Pro plan access.
**Validates: Requirements 24.2**


### Deal Lifecycle Properties

**Property 14: Deal State Machine Validity**
*For any* deal, all state transitions should follow the valid state machine: Negotiation → Awaiting Payment → Funded → Promotion Submitted → Under Review → Approved → Completed. Invalid transitions should be rejected.
**Validates: Requirements 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.8**

**Property 15: Payment Triggers Funded State**
*For any* deal in "Awaiting Payment" state, when a successful payment is recorded, the deal should transition to "Funded" state.
**Validates: Requirements 9.3**

**Property 16: Auto-Approval After 48 Hours**
*For any* deal in "Under Review" state for more than 48 hours without brand response, the system should automatically transition it to "Approved" state.
**Validates: Requirements 9.6**

**Property 17: Commission Calculation Correctness**
*For any* deal amount and plan type (Basic or Pro), the calculated commission should be exactly 1% or 0.5% of the amount respectively, and net payout should equal amount minus commission.
**Validates: Requirements 9.7, 11.1**

**Property 18: Bid to Deal Conversion**
*For any* accepted bid, the system should create a deal with the same amount, deliverables, and timeline as specified in the bid.
**Validates: Requirements 7.6**

**Property 19: Quote to Deal Conversion**
*For any* accepted quote in chat, the system should create a deal in "Negotiation" state with the quote's terms.
**Validates: Requirements 8.5, 9.1**

### Payment and Payout Properties

**Property 20: Transaction Recording Completeness**
*For any* successful payment, the recorded transaction should include transaction ID, amount, timestamp, payment method, and user ID.
**Validates: Requirements 10.3**

**Property 21: Payment Timeout Cancellation**
*For any* deal in "Awaiting Payment" state for more than 48 hours without payment, the system should automatically cancel the deal.
**Validates: Requirements 10.5**

**Property 22: Payout Threshold Enforcement**
*For any* payout amount less than ₹500, the payout should be held and not processed until the accumulated amount reaches the threshold.
**Validates: Requirements 11.7**

**Property 23: Payout Timing by Plan**
*For any* approved deal, if the influencer is on Basic plan, payout should be scheduled 5-7 days after approval; if on Pro plan, 2-3 days after approval.
**Validates: Requirements 11.2, 11.3**

### Fraud Detection Properties

**Property 24: Risk Score Bounds**
*For any* chat message analyzed by the fraud detection system, the assigned risk score should always be between 0 and 100 inclusive.
**Validates: Requirements 12.2**

**Property 25: Bypass Keyword Detection**
*For any* message containing known bypass keywords (phone numbers, external payment mentions), the system should flag it and assign a risk score above 0.
**Validates: Requirements 12.1**

**Property 26: Input Sanitization**
*For any* user input containing XSS or SQL injection patterns, the sanitization function should remove or escape the malicious content.
**Validates: Requirements 22.4**

### File Management Properties

**Property 27: File Type Validation**
*For any* uploaded file, if the file extension is not in the allowed list (jpg, png, pdf, doc, docx), the upload should be rejected.
**Validates: Requirements 19.1**

**Property 28: File Size Limit**
*For any* file exceeding 10MB, the upload should be rejected with an appropriate error message.
**Validates: Requirements 19.2** (edge-case)

### Channel Auto-Fetch Properties

**Property 29: Metrics Update Triggers Recalculation**
*For any* channel, when new metrics are fetched and stored, the influencer's profile strength score should be recalculated.
**Validates: Requirements 16.5, 2.5**

**Property 30: Platform API Integration**
*For any* supported platform (Instagram, YouTube, Telegram, LinkedIn, Twitter, Facebook), the auto-fetch should successfully retrieve follower count, engagement rate, and post frequency.
**Validates: Requirements 16.2**

### Circuit Breaker Property

**Property 31: Circuit Breaker Activation**
*For any* external API, after 3 consecutive failures, the circuit breaker should open and prevent further calls for 30 seconds.
**Validates: Requirements 23.4**


## Error Handling

### Error Classification

**Client Errors (4xx)**:
- 400 Bad Request: Invalid input data, validation failures
- 401 Unauthorized: Missing or invalid authentication token
- 403 Forbidden: Insufficient permissions for requested action
- 404 Not Found: Resource does not exist
- 409 Conflict: Resource state conflict (e.g., duplicate email)
- 422 Unprocessable Entity: Valid syntax but semantic errors
- 429 Too Many Requests: Rate limit exceeded

**Server Errors (5xx)**:
- 500 Internal Server Error: Unhandled exceptions
- 502 Bad Gateway: External service failure
- 503 Service Unavailable: System overload or maintenance
- 504 Gateway Timeout: External service timeout

### Error Response Format

All API errors follow a consistent JSON structure:

```typescript
interface ErrorResponse {
  error: {
    code: string           // Machine-readable error code
    message: string        // Human-readable error message
    details?: any          // Additional context (validation errors, etc.)
    timestamp: string      // ISO 8601 timestamp
    requestId: string      // Unique request identifier for tracking
  }
}
```

Example:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Invalid email format",
      "password": "Password must be at least 8 characters"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_abc123xyz"
  }
}
```

### Error Handling Strategies

**Database Errors**:
- Connection failures: Retry up to 3 times with exponential backoff (1s, 2s, 4s)
- Query timeouts: Log slow query, return 503 if critical operation
- Constraint violations: Return 409 with specific constraint information
- Transaction failures: Rollback and return appropriate error

**External API Errors**:
- Timeout: Implement 30-second timeout, return 504
- Rate limiting: Implement exponential backoff, queue requests
- Authentication failures: Refresh credentials, retry once
- Service unavailable: Circuit breaker pattern, fallback to cached data if available

**Payment Gateway Errors**:
- Payment declined: Return user-friendly message, allow retry
- Gateway timeout: Mark transaction as pending, implement webhook reconciliation
- Invalid credentials: Alert admin, prevent further transactions
- Webhook signature mismatch: Log security incident, reject webhook

**File Upload Errors**:
- Size exceeded: Return 413 Payload Too Large
- Invalid type: Return 415 Unsupported Media Type
- Virus detected: Quarantine file, notify admin, return 400
- S3 upload failure: Retry up to 3 times, return 503 if all fail

**WebSocket Errors**:
- Connection failure: Client implements automatic reconnection with exponential backoff
- Message delivery failure: Store message in database, deliver when connection restored
- Authentication failure: Close connection, require re-authentication

### Logging and Monitoring

**Log Levels**:
- ERROR: Unhandled exceptions, critical failures requiring immediate attention
- WARN: Handled errors, degraded functionality, approaching limits
- INFO: Significant events (user registration, deal creation, payments)
- DEBUG: Detailed execution flow for troubleshooting

**Structured Logging Format**:
```typescript
interface LogEntry {
  level: 'ERROR' | 'WARN' | 'INFO' | 'DEBUG'
  timestamp: string
  requestId: string
  userId?: string
  service: string
  message: string
  error?: {
    name: string
    message: string
    stack: string
  }
  metadata?: any
}
```

**Monitoring and Alerting**:
- Error rate threshold: Alert if error rate exceeds 5% of requests
- Response time threshold: Alert if p95 latency exceeds 500ms
- Database connection pool: Alert if utilization exceeds 80%
- Payment failures: Alert if failure rate exceeds 10%
- WebSocket connections: Alert if connection count drops suddenly
- Disk space: Alert if usage exceeds 85%
- Memory usage: Alert if usage exceeds 90%

**Error Tracking**:
- Use Sentry or similar service for error aggregation
- Group errors by type, endpoint, and user impact
- Track error frequency and affected user count
- Implement error fingerprinting for duplicate detection
- Provide error replay capability for debugging


## Testing Strategy

### Overview

The InLink platform requires a comprehensive testing approach combining unit tests, property-based tests, integration tests, and end-to-end tests. This multi-layered strategy ensures both specific functionality and universal correctness properties are validated.

### Property-Based Testing

Property-based testing validates universal properties across randomly generated inputs, catching edge cases that example-based tests might miss.

**Library Selection**:
- **JavaScript/TypeScript**: fast-check (most mature PBT library for Node.js)
- **Installation**: `npm install --save-dev fast-check @types/fast-check`

**Configuration**:
- Minimum 100 iterations per property test (due to randomization)
- Seed-based reproducibility for failed tests
- Shrinking enabled to find minimal failing examples
- Timeout: 30 seconds per property test

**Property Test Structure**:
```typescript
import fc from 'fast-check'

describe('Feature: inlink-platform, Property 1: Registration and OTP Flow', () => {
  it('should create account and activate with valid OTP for any valid email/password', () => {
    fc.assert(
      fc.property(
        fc.emailAddress(),
        fc.string({ minLength: 8 }).filter(isValidPassword),
        async (email, password) => {
          // Feature: inlink-platform, Property 1: Registration and OTP Flow
          const { userId, otp } = await authService.register(email, password)
          expect(userId).toBeDefined()
          expect(otp).toMatch(/^\d{6}$/)
          
          const result = await authService.verifyOTP(userId, otp)
          expect(result.success).toBe(true)
          expect(result.tokens).toBeDefined()
        }
      ),
      { numRuns: 100 }
    )
  })
})
```

**Tagging Convention**:
Each property test MUST include a comment tag referencing the design document property:
```typescript
// Feature: inlink-platform, Property {number}: {property_text}
```

**Generators for Domain Objects**:
```typescript
// User generators
const userArbitrary = fc.record({
  email: fc.emailAddress(),
  password: fc.string({ minLength: 8 }).filter(isValidPassword),
  role: fc.constantFrom('influencer', 'brand_individual', 'brand_business')
})

// Deal generators
const dealArbitrary = fc.record({
  amount: fc.integer({ min: 1000, max: 1000000 }),
  deliverables: fc.array(fc.string(), { minLength: 1, maxLength: 5 }),
  timeline: fc.string({ minLength: 10, maxLength: 100 })
})

// Channel metrics generators
const channelMetricsArbitrary = fc.record({
  followerCount: fc.integer({ min: 100, max: 10000000 }),
  engagementRate: fc.float({ min: 0, max: 100 }),
  postFrequency: fc.float({ min: 0, max: 10 })
})
```

### Unit Testing

Unit tests validate specific examples, edge cases, and error conditions for individual functions and components.

**Framework**: Jest with TypeScript support
**Coverage Target**: 80% code coverage minimum

**Unit Test Focus Areas**:
- Input validation functions
- Business logic calculations (commission, profile strength, risk score)
- State machine transitions
- Error handling paths
- Edge cases (empty arrays, null values, boundary conditions)

**Example Unit Tests**:
```typescript
describe('CommissionCalculator', () => {
  it('should calculate 1% commission for Basic plan', () => {
    const result = calculateCommission(10000, 'basic')
    expect(result).toBe(100)
  })
  
  it('should calculate 0.5% commission for Pro plan', () => {
    const result = calculateCommission(10000, 'pro')
    expect(result).toBe(50)
  })
  
  it('should handle zero amount', () => {
    const result = calculateCommission(0, 'basic')
    expect(result).toBe(0)
  })
  
  it('should round to nearest integer', () => {
    const result = calculateCommission(10001, 'basic')
    expect(result).toBe(100) // 100.01 rounded down
  })
})
```

### Integration Testing

Integration tests validate interactions between multiple components and external services.

**Test Scenarios**:
- Database operations with real PostgreSQL test database
- Redis caching behavior
- WebSocket message flow
- Payment gateway integration (sandbox mode)
- File upload to S3 (test bucket)
- Email sending (test SMTP server)

**Setup**:
- Use Docker Compose for test infrastructure
- Separate test database with migrations
- Mock external APIs (Google Cloud, PayU) with realistic responses
- Clean database state between tests

**Example Integration Test**:
```typescript
describe('Deal Lifecycle Integration', () => {
  beforeEach(async () => {
    await cleanDatabase()
    await seedTestData()
  })
  
  it('should complete full deal lifecycle from creation to payout', async () => {
    // Create users
    const brand = await createTestBrand()
    const influencer = await createTestInfluencer()
    
    // Create deal
    const deal = await dealService.createDeal(brand.id, influencer.id, testTerms)
    expect(deal.state).toBe('negotiation')
    
    // Transition to awaiting payment
    await dealService.updateDealState(deal.id, 'awaiting_payment')
    
    // Process payment
    await paymentService.processPayment(deal.id, deal.amount)
    const fundedDeal = await dealService.getDeal(deal.id)
    expect(fundedDeal.state).toBe('funded')
    
    // Submit promotion
    await dealService.submitPromotion(deal.id, testProof)
    
    // Auto-approve after 48 hours
    await advanceTime(49 * 60 * 60 * 1000)
    await runAutoApprovalJob()
    
    const approvedDeal = await dealService.getDeal(deal.id)
    expect(approvedDeal.state).toBe('approved')
    
    // Verify payout queued
    const payout = await payoutService.getPayoutForDeal(deal.id)
    expect(payout).toBeDefined()
    expect(payout.amount).toBe(deal.netPayout)
  })
})
```

### End-to-End Testing

E2E tests validate complete user workflows through the UI.

**Framework**: Playwright or Cypress
**Test Scenarios**:
- User registration and verification flow
- Influencer profile creation with channel linking
- Brand campaign creation and bid submission
- Chat negotiation and deal creation
- Payment processing and deal completion
- Admin verification approval workflow

**Example E2E Test**:
```typescript
test('Influencer can complete verification and receive Pro plan', async ({ page }) => {
  // Register
  await page.goto('/register')
  await page.fill('[name="email"]', 'test@example.com')
  await page.fill('[name="password"]', 'Test123!@#')
  await page.click('button[type="submit"]')
  
  // Verify OTP (mock)
  const otp = await getTestOTP('test@example.com')
  await page.fill('[name="otp"]', otp)
  await page.click('button:has-text("Verify")')
  
  // Navigate to verification
  await page.goto('/dashboard/verification')
  await page.setInputFiles('[name="idDocument"]', 'test-id.jpg')
  await page.setInputFiles('[name="selfie"]', 'test-selfie.jpg')
  await page.click('button:has-text("Submit")')
  
  // Wait for auto-approval (mock high confidence)
  await page.waitForSelector('text=Verification Approved')
  
  // Verify Pro plan granted
  await page.goto('/dashboard/settings')
  await expect(page.locator('text=Pro Plan')).toBeVisible()
  await expect(page.locator('text=2 months free')).toBeVisible()
})
```

### Performance Testing

**Load Testing**:
- Tool: k6 or Artillery
- Scenarios:
  - 1000 concurrent users browsing platform
  - 100 concurrent WebSocket connections
  - 50 deals created per minute
  - 200 API requests per second
- Metrics: Response time (p50, p95, p99), error rate, throughput

**Stress Testing**:
- Gradually increase load until system breaks
- Identify bottlenecks and failure points
- Verify graceful degradation

### Security Testing

**Automated Security Scans**:
- OWASP ZAP for vulnerability scanning
- npm audit for dependency vulnerabilities
- Snyk for continuous security monitoring

**Manual Security Testing**:
- Penetration testing by security professionals
- Authentication bypass attempts
- Authorization boundary testing
- SQL injection and XSS testing
- CSRF protection validation

### Test Execution

**CI/CD Pipeline**:
1. Unit tests: Run on every commit (< 2 minutes)
2. Property tests: Run on every commit (< 5 minutes)
3. Integration tests: Run on every PR (< 10 minutes)
4. E2E tests: Run on every PR to main (< 15 minutes)
5. Performance tests: Run nightly
6. Security scans: Run weekly

**Test Environment**:
- Staging environment mirrors production
- Test data generation scripts
- Database snapshots for consistent state
- Feature flags for gradual rollout

### Test Maintenance

**Best Practices**:
- Keep tests independent and isolated
- Use factories for test data generation
- Avoid test interdependencies
- Clean up test data after execution
- Use descriptive test names
- Document complex test scenarios
- Review and update tests with code changes
- Monitor test flakiness and fix immediately

