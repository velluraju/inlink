# Requirements Document: InLink Platform

## Introduction

InLink is an influencer-brand collaboration marketplace platform that connects social media influencers with brands for promotional campaigns. The platform differentiates itself through low commission rates (0.5-1% vs industry standard 10-20%), AI-powered verification systems, multi-platform social media integration, real-time communication, secure payment processing, and fraud detection mechanisms.

The system serves four primary user types: Influencers (content creators seeking brand deals), Brands (businesses seeking influencer partnerships), Admins (platform moderators), and Super Admins (system administrators). The platform facilitates the complete deal lifecycle from discovery and negotiation through payment and promotion verification.

## Glossary

- **System**: The InLink platform (web application, backend services, and AI systems)
- **Influencer**: A verified social media content creator registered on the platform
- **Brand**: A business or individual registered to create campaigns and hire influencers
- **Deal**: A formal agreement between an influencer and brand for promotional content
- **Campaign**: An auction-based opportunity posted by brands where influencers can bid
- **Quote**: A structured proposal sent between influencers and brands during negotiation
- **Channel**: A social media account linked to an influencer profile (Instagram, YouTube, etc.)
- **Verification**: The AI-powered identity validation process requiring document submission
- **Commission**: The platform fee deducted from deal amounts (0.5% Pro, 1% Basic)
- **GMV**: Gross Merchandise Value - total transaction volume on the platform
- **Bypass**: Attempting to conduct transactions outside the platform to avoid fees
- **Admin**: Platform moderator with access to verification queues and dispute resolution
- **Super_Admin**: System administrator with full platform control and financial oversight
- **Pro_Plan**: Premium subscription (₹499/month) offering 0.5% commission and faster payouts
- **Basic_Plan**: Free tier offering 1% commission and standard payouts
- **Funded_Status**: Deal state indicating payment has been received and work can begin
- **Auto_Fetch**: Automated daily synchronization of social media analytics at 2 AM IST
- **Payout**: Transfer of earnings from platform to influencer bank account
- **Chat_System**: Real-time WebSocket-based messaging between users
- **Payment_Gateway**: PayU integration for processing transactions

## Requirements

### Requirement 1: User Authentication and Authorization

**User Story:** As a user, I want to securely register and log in to the platform, so that I can access role-specific features and protect my account.

#### Acceptance Criteria

1. WHEN a new user registers with email and password, THE System SHALL create an account and send an OTP verification code
2. WHEN a user enters a valid OTP within 10 minutes, THE System SHALL activate the account
3. WHEN a user logs in with valid credentials, THE System SHALL issue a JWT access token (15 minutes expiry) and refresh token (7 days expiry)
4. WHEN an access token expires, THE System SHALL accept a valid refresh token to issue a new access token
5. WHEN a user requests password reset, THE System SHALL send a reset link valid for 30 minutes
6. WHERE a user has Admin or Super_Admin role, THE System SHALL require 2FA authentication
7. WHEN a user fails login 5 times within 15 minutes, THE System SHALL temporarily lock the account for 30 minutes
8. THE System SHALL enforce password requirements: minimum 8 characters, one uppercase, one lowercase, one number, one special character

### Requirement 2: Influencer Profile Management

**User Story:** As an influencer, I want to create and manage my profile with linked social channels, so that brands can discover me and view my authentic metrics.

#### Acceptance Criteria

1. WHEN an influencer creates a profile, THE System SHALL require at least one social media channel link
2. WHEN an influencer links a social channel, THE Auto_Fetch SHALL retrieve follower count, engagement rate, and post frequency within 24 hours
3. WHILE an influencer has linked channels, THE Auto_Fetch SHALL synchronize metrics daily at 2 AM IST
4. THE System SHALL support channel linking for Instagram, YouTube, Telegram, LinkedIn, Twitter, and Facebook
5. WHEN channel metrics are updated, THE System SHALL recalculate the profile strength score (0-100 scale)
6. WHEN an influencer updates profile information, THE System SHALL validate all required fields before saving
7. THE System SHALL display verification status badge (Unverified, Pending, Verified) on influencer profiles
8. WHEN an influencer has zero active channels, THE System SHALL prevent profile from appearing in brand searches

### Requirement 3: AI-Powered Identity Verification

**User Story:** As an influencer, I want to verify my identity through an AI-powered process, so that I can access premium features and build trust with brands.

#### Acceptance Criteria

1. WHEN an influencer submits verification documents (government ID and selfie), THE System SHALL charge ₹199
2. WHEN documents are uploaded, THE Verification SHALL use Google Cloud Vision API for OCR extraction
3. WHEN face matching confidence is 99% or higher, THE System SHALL auto-approve verification within 2-5 minutes
4. WHEN face matching confidence is between 90-98%, THE System SHALL queue for manual Admin review
5. WHEN face matching confidence is below 90%, THE System SHALL auto-reject and refund ₹199
6. WHEN a document ID number matches an existing verified account, THE System SHALL reject as duplicate
7. WHEN verification is approved for the first time, THE System SHALL grant 2 months free Pro_Plan access
8. THE System SHALL store verification confidence score and processing timestamp for audit purposes

### Requirement 4: Subscription Plan Management

**User Story:** As an influencer, I want to choose between Basic and Pro plans, so that I can optimize my commission rates and payout speed based on my activity level.

#### Acceptance Criteria

1. THE System SHALL assign Basic_Plan (1% commission, 5-7 day payouts) to all new influencers by default
2. WHEN an influencer subscribes to Pro_Plan, THE System SHALL charge ₹499 monthly via PayU
3. WHEN Pro_Plan payment succeeds, THE System SHALL reduce commission to 0.5% and payout time to 2-3 days
4. WHEN Pro_Plan subscription expires, THE System SHALL revert influencer to Basic_Plan automatically
5. WHERE an influencer is on Basic_Plan, THE System SHALL limit active deals to 3 and active bids to 5
6. WHERE an influencer is on Pro_Plan, THE System SHALL allow unlimited active deals and bids
7. WHEN an influencer is a beta user, THE System SHALL grant 6 months free Pro_Plan access
8. WHEN Pro_Plan is active, THE System SHALL display "Pro" badge on influencer profile

### Requirement 5: Brand Account Management

**User Story:** As a brand, I want to register as either an individual or business account, so that I can access appropriate features for my organization type.

#### Acceptance Criteria

1. WHEN a brand registers as Individual account, THE System SHALL charge ₹399 one-time fee
2. WHEN a brand registers as Business account, THE System SHALL charge ₹799 one-time fee and require business documents
3. WHEN Business account documents are submitted, THE System SHALL queue for Admin verification
4. WHEN brand registration payment succeeds, THE System SHALL activate the account immediately
5. THE System SHALL allow brands to add multiple team members with role-based permissions
6. WHEN a brand adds products for verification, THE System SHALL charge ₹29, ₹49, or ₹99 based on product category
7. WHEN product verification is approved, THE System SHALL display verified badge on product listings
8. THE System SHALL maintain separate dashboards for Individual and Business accounts with appropriate features

### Requirement 6: Influencer Discovery and Search

**User Story:** As a brand, I want to search and filter influencers by multiple criteria, so that I can find the most suitable partners for my campaigns.

#### Acceptance Criteria

1. WHEN a brand searches influencers, THE System SHALL return only verified influencers by default
2. THE System SHALL support filtering by platform (Instagram, YouTube, Telegram, LinkedIn, Twitter, Facebook)
3. THE System SHALL support filtering by follower count ranges (1K-10K, 10K-50K, 50K-100K, 100K-500K, 500K+)
4. THE System SHALL support filtering by engagement rate ranges (0-2%, 2-5%, 5-10%, 10%+)
5. THE System SHALL support filtering by niche categories (Fashion, Tech, Food, Travel, etc.)
6. WHEN displaying search results, THE System SHALL show profile strength score, verification badge, and Pro badge
7. THE System SHALL rank search results by relevance score combining profile strength, engagement, and verification status
8. WHEN a brand views an influencer profile, THE System SHALL display complete channel analytics and past collaboration count

### Requirement 7: Campaign Creation and Auction System

**User Story:** As a brand, I want to create auction-based campaigns where influencers can bid, so that I can receive competitive proposals and choose the best fit.

#### Acceptance Criteria

1. WHEN a brand creates a campaign, THE System SHALL require campaign title, description, budget range, deadline, and target platforms
2. WHEN a campaign is published, THE System SHALL make it visible to all verified influencers matching the criteria
3. WHEN an influencer submits a bid, THE System SHALL include proposed price, deliverables, and timeline
4. WHERE an influencer is on Basic_Plan and has 5 active bids, THE System SHALL prevent new bid submissions
5. WHEN a brand reviews bids, THE System SHALL display influencer metrics, verification status, and past performance
6. WHEN a brand accepts a bid, THE System SHALL convert it to a Deal and notify the influencer
7. WHEN campaign deadline passes, THE System SHALL close bidding and archive the campaign
8. THE System SHALL allow brands to invite specific influencers to private campaigns

### Requirement 8: Real-Time Chat System

**User Story:** As a user, I want to communicate in real-time with potential partners, so that I can negotiate deal terms and share information efficiently.

#### Acceptance Criteria

1. WHEN a brand and influencer initiate chat, THE Chat_System SHALL establish a WebSocket connection
2. WHEN a user sends a message, THE Chat_System SHALL deliver it to the recipient within 1 second
3. THE Chat_System SHALL support text messages, file attachments (max 10MB), and quote objects
4. WHEN a user sends a quote, THE System SHALL structure it with price, deliverables, timeline, and terms
5. WHEN a quote is accepted in chat, THE System SHALL create a Deal in "Negotiation" state
6. WHILE chat is active, THE Chat_System SHALL display typing indicators and read receipts
7. WHEN a message contains bypass keywords (phone numbers, external payment mentions), THE System SHALL flag for AI review
8. THE Chat_System SHALL maintain message history for 2 years for audit purposes

### Requirement 9: Deal Lifecycle Management

**User Story:** As a user, I want deals to progress through clear states with automated transitions, so that both parties understand expectations and timelines.

#### Acceptance Criteria

1. WHEN a deal is created from accepted quote, THE System SHALL set state to "Negotiation"
2. WHEN both parties agree on final terms, THE System SHALL transition to "Awaiting Payment" and generate payment link
3. WHEN brand completes 100% upfront payment, THE System SHALL transition to "Funded" and notify influencer to begin work
4. WHEN influencer submits promotion proof (screenshots, links), THE System SHALL transition to "Under Review"
5. WHEN brand approves promotion within 48 hours, THE System SHALL transition to "Approved"
6. IF brand does not respond within 48 hours, THE System SHALL auto-approve and transition to "Approved"
7. WHEN deal reaches "Approved" state, THE System SHALL deduct commission (0.5% or 1%) and queue payout
8. WHEN payout is processed, THE System SHALL transition to "Completed" and close the deal

### Requirement 10: Payment Processing

**User Story:** As a brand, I want to pay for deals securely through multiple payment methods, so that I can fund collaborations conveniently.

#### Acceptance Criteria

1. WHEN a deal requires payment, THE Payment_Gateway SHALL generate a PayU payment link valid for 24 hours
2. THE Payment_Gateway SHALL support UPI, credit cards, debit cards, net banking, and digital wallets
3. WHEN payment succeeds, THE System SHALL record transaction ID, amount, timestamp, and payment method
4. WHEN payment fails, THE System SHALL notify the brand and allow retry for 48 hours
5. IF payment is not completed within 48 hours, THE System SHALL cancel the deal and notify both parties
6. THE System SHALL reconcile payments daily at 3 AM IST against PayU settlement reports
7. WHEN a deal is cancelled before "Funded" state, THE System SHALL process refund within 5-7 business days
8. THE System SHALL maintain transaction audit logs for 7 years for compliance

### Requirement 11: Payout Processing

**User Story:** As an influencer, I want to receive earnings in my bank account after deal completion, so that I can monetize my collaborations.

#### Acceptance Criteria

1. WHEN a deal reaches "Approved" state, THE System SHALL calculate net payout (deal amount minus commission)
2. WHERE influencer is on Basic_Plan, THE Payout SHALL be processed within 5-7 business days
3. WHERE influencer is on Pro_Plan, THE Payout SHALL be processed within 2-3 business days
4. WHEN payout is initiated, THE System SHALL transfer funds to influencer's registered bank account via NEFT/IMPS
5. WHEN payout succeeds, THE System SHALL send confirmation email with transaction reference
6. IF payout fails due to invalid bank details, THE System SHALL notify influencer to update information
7. THE System SHALL maintain minimum payout threshold of ₹500 (amounts below are held until threshold is met)
8. THE System SHALL generate payout reports monthly for influencer tax purposes

### Requirement 12: AI Fraud Detection System

**User Story:** As the platform, I want to detect and prevent bypass attempts automatically, so that I can protect revenue and maintain marketplace integrity.

#### Acceptance Criteria

1. WHEN a chat message is sent, THE System SHALL analyze it using NLP for bypass keywords (phone, WhatsApp, direct payment, etc.)
2. WHEN bypass keywords are detected, THE System SHALL assign risk score (0-100) based on context and user history
3. WHEN risk score exceeds 70, THE System SHALL flag the conversation for Admin review
4. WHEN risk score exceeds 90, THE System SHALL automatically warn both users and log the incident
5. WHEN a first bypass is confirmed, THE System SHALL suspend user for 30 days and charge ₹5,000 fine
6. WHEN a second bypass is confirmed, THE System SHALL suspend user for 90 days and charge ₹15,000 fine
7. WHEN a third bypass is confirmed, THE System SHALL permanently ban the user
8. THE System SHALL monitor post-deal content for undisclosed collaborations using image recognition and metadata analysis

### Requirement 13: Admin Panel Operations

**User Story:** As an admin, I want to manage verifications, disputes, and moderation tasks, so that I can maintain platform quality and resolve issues.

#### Acceptance Criteria

1. WHEN an admin logs in at /abcdef/admin/login, THE System SHALL require 2FA authentication
2. THE System SHALL display verification queues for influencer identity (90-98% confidence), brand business documents, and products
3. WHEN an admin reviews verification, THE System SHALL provide document images, extracted data, and confidence scores
4. WHEN an admin approves or rejects verification, THE System SHALL update user status and send notification within 1 minute
5. THE System SHALL display flagged chat conversations with risk scores and keyword highlights
6. WHEN an admin reviews a dispute, THE System SHALL show complete deal history, chat logs, and submitted evidence
7. THE System SHALL allow admins to manually adjust deal states, issue refunds, and apply penalties
8. THE System SHALL log all admin actions with timestamp, admin ID, and reason for audit trail

### Requirement 14: Super Admin System Control

**User Story:** As a super admin, I want complete platform oversight and configuration control, so that I can manage business operations and system settings.

#### Acceptance Criteria

1. WHEN a super admin logs in at /adasdadjashdkaj2131213/asdasidaodi23131/super-admin/login, THE System SHALL require 2FA authentication
2. THE System SHALL display financial dashboard with GMV, revenue, MRR, ARR, and commission breakdown
3. THE System SHALL allow super admin to modify commission rates, subscription fees, and verification prices
4. THE System SHALL allow super admin to override any admin decision or user action
5. THE System SHALL display complete audit logs filterable by user, action type, date range, and admin
6. THE System SHALL provide system analytics including user growth, deal volume, platform health metrics
7. THE System SHALL allow super admin to create, modify, and delete admin accounts with role assignments
8. THE System SHALL maintain immutable audit trail of all super admin actions for compliance

### Requirement 15: Public Website Content

**User Story:** As a visitor, I want to learn about the platform through informative pages, so that I can decide whether to register.

#### Acceptance Criteria

1. THE System SHALL serve a public website with Home, How It Works, Features, and Pricing pages
2. THE System SHALL provide separate landing pages for "For Influencers" and "For Brands" audiences
3. THE System SHALL display Beta Program details, Events calendar, and Success Stories
4. THE System SHALL maintain a blog with articles about influencer marketing and platform updates
5. THE System SHALL provide About, Contact, Careers, and Press & Media pages
6. THE System SHALL display legal pages: Terms of Service, Privacy Policy, Refund Policy, Cookie Policy, Community Guidelines
7. WHEN a visitor submits a contact form, THE System SHALL send email to support team within 5 minutes
8. THE System SHALL ensure all public pages load within 2 seconds on 4G connection

### Requirement 16: Channel Analytics Auto-Fetch

**User Story:** As the platform, I want to automatically synchronize influencer social media metrics daily, so that brands see accurate, up-to-date analytics.

#### Acceptance Criteria

1. WHEN the clock reaches 2:00 AM IST daily, THE Auto_Fetch SHALL begin synchronization for all linked channels
2. THE Auto_Fetch SHALL retrieve follower count, engagement rate, and post frequency from each platform API
3. WHEN API rate limits are reached, THE Auto_Fetch SHALL queue remaining channels for next available slot
4. WHEN API returns error, THE Auto_Fetch SHALL retry up to 3 times with exponential backoff
5. WHEN metrics are successfully fetched, THE System SHALL update influencer profile and recalculate profile strength
6. WHEN metrics show significant drop (>20% followers), THE System SHALL flag profile for Admin review
7. THE Auto_Fetch SHALL complete all synchronizations within 4 hours (by 6:00 AM IST)
8. THE System SHALL log all fetch attempts with status, timestamp, and error details for monitoring

### Requirement 17: Notification System

**User Story:** As a user, I want to receive timely notifications about important events, so that I can respond quickly to opportunities and requirements.

#### Acceptance Criteria

1. WHEN a deal state changes, THE System SHALL send email and in-app notification to both parties
2. WHEN a new bid is received on a campaign, THE System SHALL notify the brand within 1 minute
3. WHEN a payment is received, THE System SHALL send confirmation notification to both brand and influencer
4. WHEN verification is approved or rejected, THE System SHALL notify the user immediately
5. WHEN a chat message is received while user is offline, THE System SHALL send email notification after 5 minutes
6. WHEN payout is processed, THE System SHALL send confirmation with transaction details
7. THE System SHALL allow users to configure notification preferences (email, in-app, SMS)
8. WHEN a user is suspended or penalized, THE System SHALL send detailed notification with reason and appeal process

### Requirement 18: Dashboard Analytics

**User Story:** As a user, I want to view comprehensive analytics on my dashboard, so that I can track performance and make informed decisions.

#### Acceptance Criteria

1. WHEN an influencer views dashboard, THE System SHALL display total earnings, active deals, completed deals, and pending payouts
2. WHEN a brand views dashboard, THE System SHALL display total spent, active campaigns, completed deals, and ROI metrics
3. THE System SHALL provide date range filters (Last 7 days, Last 30 days, Last 90 days, Custom range)
4. THE System SHALL display earnings/spending trends as line charts with monthly breakdown
5. THE System SHALL show deal success rate, average deal value, and average completion time
6. WHERE user is an influencer, THE System SHALL display profile views, bid acceptance rate, and profile strength trend
7. WHERE user is a brand, THE System SHALL display campaign response rate, average bid count, and influencer retention rate
8. THE System SHALL allow exporting dashboard data as CSV or PDF reports

### Requirement 19: File Storage and Management

**User Story:** As the platform, I want to store user-uploaded files securely and efficiently, so that documents, images, and attachments are accessible and protected.

#### Acceptance Criteria

1. WHEN a user uploads a file, THE System SHALL validate file type against allowed extensions (jpg, png, pdf, doc, docx)
2. WHEN a file exceeds 10MB, THE System SHALL reject the upload and display error message
3. WHEN file validation passes, THE System SHALL upload to AWS S3 with unique filename and user-specific path
4. THE System SHALL generate signed URLs for file access with 1-hour expiration
5. WHEN a verification document is uploaded, THE System SHALL encrypt it at rest using AES-256
6. WHEN a user account is deleted, THE System SHALL mark associated files for deletion after 90-day retention period
7. THE System SHALL implement virus scanning on all uploaded files before storage
8. THE System SHALL maintain file metadata (upload timestamp, user ID, file size, content type) in database

### Requirement 20: Search Engine Optimization

**User Story:** As the platform, I want public pages to be optimized for search engines, so that we can attract organic traffic and grow user base.

#### Acceptance Criteria

1. THE System SHALL generate unique meta titles and descriptions for all public pages
2. THE System SHALL implement structured data markup (JSON-LD) for organization, articles, and FAQs
3. THE System SHALL generate XML sitemap automatically and update it when content changes
4. THE System SHALL implement canonical URLs to prevent duplicate content issues
5. THE System SHALL ensure all images have descriptive alt text for accessibility and SEO
6. THE System SHALL implement Open Graph and Twitter Card meta tags for social sharing
7. THE System SHALL serve pages with semantic HTML5 markup (header, nav, main, article, footer)
8. THE System SHALL achieve Lighthouse SEO score of 90+ on all public pages

### Requirement 21: Performance and Scalability

**User Story:** As the platform, I want the system to handle growing user base and transaction volume, so that performance remains consistent as we scale.

#### Acceptance Criteria

1. THE System SHALL serve API responses within 200ms for 95th percentile requests
2. THE System SHALL handle 1000 concurrent WebSocket connections without degradation
3. WHEN database queries exceed 100ms, THE System SHALL log slow query for optimization
4. THE System SHALL implement Redis caching for frequently accessed data (user profiles, channel metrics)
5. THE System SHALL use database connection pooling with minimum 10 and maximum 50 connections
6. THE System SHALL implement rate limiting: 100 requests per minute per user for API endpoints
7. THE System SHALL use CDN for static assets with 1-year cache headers
8. THE System SHALL implement horizontal scaling capability for backend services using load balancer

### Requirement 22: Security and Compliance

**User Story:** As the platform, I want to protect user data and comply with regulations, so that users trust the platform and we avoid legal issues.

#### Acceptance Criteria

1. THE System SHALL encrypt all data in transit using TLS 1.3
2. THE System SHALL hash passwords using bcrypt with salt rounds of 12
3. THE System SHALL implement CSRF protection on all state-changing endpoints
4. THE System SHALL sanitize all user inputs to prevent XSS and SQL injection attacks
5. THE System SHALL implement Content Security Policy headers to prevent XSS
6. THE System SHALL comply with PCI DSS requirements for payment data handling (no card storage)
7. THE System SHALL provide data export functionality for users to download their data (GDPR compliance)
8. THE System SHALL implement account deletion with 30-day grace period and complete data removal after 90 days

### Requirement 23: Error Handling and Monitoring

**User Story:** As the platform, I want to detect, log, and recover from errors gracefully, so that users have reliable experience and issues are resolved quickly.

#### Acceptance Criteria

1. WHEN an unhandled error occurs, THE System SHALL log error details (stack trace, user context, timestamp) to monitoring service
2. WHEN a critical error occurs, THE System SHALL send alert to on-call engineer within 1 minute
3. WHEN an API endpoint fails, THE System SHALL return appropriate HTTP status code and user-friendly error message
4. THE System SHALL implement circuit breaker pattern for external API calls (3 failures trigger 30-second cooldown)
5. WHEN database connection fails, THE System SHALL retry up to 3 times before returning error
6. THE System SHALL maintain 99.9% uptime SLA (max 43 minutes downtime per month)
7. THE System SHALL implement health check endpoints for load balancer monitoring
8. THE System SHALL provide error tracking dashboard showing error frequency, affected users, and resolution status

### Requirement 24: Beta Program Management

**User Story:** As the platform, I want to manage beta users with special benefits, so that we can reward early adopters and gather valuable feedback.

#### Acceptance Criteria

1. WHEN a user registers with beta invite code, THE System SHALL mark account as beta user
2. WHEN a beta user completes verification, THE System SHALL grant 6 months free Pro_Plan access
3. WHERE a user is a beta user, THE System SHALL display "Beta" badge on profile
4. WHEN beta users appear in search results, THE System SHALL rank them higher than non-beta users
5. THE System SHALL allow super admin to generate beta invite codes with usage limits
6. WHEN beta invite code reaches usage limit, THE System SHALL deactivate it automatically
7. THE System SHALL track beta user metrics separately (retention, deal volume, feedback submissions)
8. WHEN beta period ends (6 months), THE System SHALL notify user 7 days before Pro_Plan expiration

### Requirement 25: Mobile Responsiveness

**User Story:** As a user, I want to access the platform on mobile devices, so that I can manage deals and communicate on the go.

#### Acceptance Criteria

1. THE System SHALL render all pages responsively on screen widths from 320px to 2560px
2. THE System SHALL use mobile-first design approach with breakpoints at 640px, 768px, 1024px, 1280px
3. WHEN viewed on mobile, THE System SHALL display touch-friendly buttons (minimum 44x44px tap targets)
4. WHEN viewed on mobile, THE System SHALL collapse navigation into hamburger menu
5. THE System SHALL optimize images for mobile with responsive srcset attributes
6. THE System SHALL ensure forms are usable on mobile with appropriate input types and autocomplete
7. THE System SHALL achieve Lighthouse mobile performance score of 80+
8. THE System SHALL support Progressive Web App features (offline page, add to home screen)
