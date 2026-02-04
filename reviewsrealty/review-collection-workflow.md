# ReviewsRealty Review Collection Workflow
**Date:** 2026-02-04  
**Project:** ReviewsRealty  
**Purpose:** Define the complete journey of a review from submission to publication, including verification and admin processes.

---

## Overview

The review collection workflow has 4 key phases:
1. **Submission** - User fills out form and uploads proof
2. **Verification** - System validates email & transaction proof
3. **Admin Review** - Moderator approves/rejects
4. **Publication** - Review goes live on agent/property profile

Each phase has specific requirements, validations, and error handling.

---

## Phase 1: Submission (User-Facing)

### 1.1 Entry Points
- **Anonymous Browse → Submit Review** (most common)
  - User browses reviews on agent/property page
  - Clicks "Write a Review" button (CTA prominent)
  - Redirected to login/signup if not authenticated

- **Agent Search → Direct Submission**
  - User searches agent name
  - Views profile
  - Clicks "Review this agent"
  - Pre-fills agent name field

- **Property Search → Direct Submission**
  - User searches property address
  - Views property page
  - Clicks "Add your review"
  - Pre-fills property address field

### 1.2 Authentication Check
**Before showing form:**
- If user NOT logged in: Show login/signup modal
- If logged in: Proceed to form

**Sign-up Requirements:**
- Email (required)
- Password (required)
- First & Last Name (required)
- Phone (optional - for verification follow-up)

**Post-signup:** Auto-proceed to review form (better UX than forcing login page)

---

## Phase 2: Review Form (Data Collection)

### 2.1 Required Fields

| Field | Type | Validation | Notes |
|-------|------|-----------|-------|
| Transaction Type | Dropdown | Buyer / Seller (required) | Determines reviewer role |
| Property Address | Text + Autocomplete | Street, City, State, ZIP | Use Google Maps API for suggestions |
| Agent Name | Text + Autocomplete | Min 2 chars | Auto-suggest existing agents |
| Star Rating | Radio / 5-Star | 1-5 (required) | Visual star display |
| Written Review | Textarea | Min 50 chars, Max 2000 | Character counter |
| Transaction Date | Date Picker | Must be ≤ today | Can't review future transactions |

### 2.2 Optional Fields

| Field | Type | Notes |
|-------|------|-------|
| Property Type | Dropdown | House, Condo, Land, etc. - for categorization |
| Price Range | Dropdown | $0-250k, $250-500k, etc. - helps context |
| Would Recommend? | Yes/No | Simple conversion metric |
| Contact Email (Optional) | Email | For follow-up verification if needed |
| Photos | File Upload | Max 5 files, 5MB each, JPG/PNG only |

### 2.3 Form Validation (Client-Side)

Before allowing submission:

```
✓ Transaction type selected
✓ Property address entered (≥ 3 chars)
✓ Agent selected from dropdown (existing agent)
✓ Star rating selected
✓ Review text ≥ 50 chars
✓ Transaction date ≤ today
✓ Photos (if provided) are valid format & size
✓ No obvious spam patterns (ALL CAPS sentences, repeated words)
```

**Error UX:** Red border + tooltip with specific error. Don't let submit if any fail.

---

## Phase 3: Transaction Verification (Upload & Validation)

### 3.1 Proof Document Upload

**This is the critical step for platform credibility.**

After review form filled, prompt:
> "Verify your review is authentic by uploading transaction proof"
> "This keeps ReviewsRealty trustworthy. We'll keep it private."

**Accepted Documents:**
- Closing statement (PDF)
- Purchase contract (PDF)
- Deed recording (PDF)
- Screenshot of MLS listing (JPG/PNG)
- Email confirmation from agent/broker (screenshot)

**What NOT to accept:**
- Screenshots of listing searches
- Generic mortgage documents without property address
- Photos of contracts that are blurry/illegible

### 3.2 File Upload Requirements

| Requirement | Details |
|------------|---------|
| **File Size** | Max 10MB per document |
| **Format** | PDF, JPG, PNG only |
| **Minimum Documents** | 1 required (some reviewers won't have docs - see 3.3) |
| **Maximum Documents** | 3 max (to prevent spam uploads) |
| **Required Text** | Document must contain: property address, agent name, or date (auto-check via OCR/regex) |

### 3.3 Fallback for Missing Documents

**If reviewer can't upload proof:**
- Allow submission with "Unverified" status
- Add note: "Awaiting verification"
- Send email asking for alternative proof
- Admin manually reviews and contacts reviewer
- If no response in 7 days → auto-delete review

**Rationale:** Some legitimate reviewers may not have access to docs, but verification email + admin review will catch most fakes.

### 3.4 Document Processing (Backend)

1. **Upload received** → Send to S3/Cloudflare R2
2. **Virus scan** → ClamAV or equivalent
3. **OCR pass** → Extract text to verify address/agent/date present
4. **Flag for admin** → If OCR fails or text mismatch → manual review required
5. **Store securely** → Encrypt at rest, never shown publicly

---

## Phase 4: Pre-Submission Summary

Before final submit, show reviewer:

```
╔═══════════════════════════════════════════════════════════════╗
║  REVIEW SUMMARY - Please confirm before submitting            ║
╠═══════════════════════════════════════════════════════════════╣
║  Agent: John Smith, Miami Realty                              ║
║  Property: 123 Main St, Miami, FL 33139                       ║
║  Your Rating: ⭐⭐⭐⭐⭐ (5 stars)                              ║
║  Review: "John was professional and responsive throughout..." ║
║  Transaction: Buyer (Jan 2023)                                ║
║  Proof: closing_statement.pdf (uploaded)                      ║
║                                                               ║
║  ☐ I confirm this is an authentic review of my transaction   ║
║  ☐ I understand fake reviews may result in account removal   ║
║                                                               ║
║           [SUBMIT] [CANCEL]                                   ║
╚═══════════════════════════════════════════════════════════════╝
```

**Confirmation checkbox** = Legal protection + intent confirmation.

---

## Phase 5: Submission & Queuing

### 5.1 Review Status Flow

```
SUBMITTED (User confirms)
    ↓
PENDING_VERIFICATION (System running checks)
    ↓
    ├─→ EMAIL_VERIFICATION (Email needs confirmation) → PENDING_ADMIN
    ├─→ DOCUMENT_REVIEW (Admin checks proof) → APPROVED or REJECTED
    └─→ SPAM_FLAGGED (Spam score > threshold) → ADMIN_REVIEW
    
PENDING_ADMIN (Waiting for moderator approval)
    ↓
APPROVED ✓ → PUBLISHED (Live on site)
    ↓
REJECTED ✗ → Notify user + archive
```

### 5.2 Immediate Feedback to Reviewer

**Success Screen:**
```
✓ Review Submitted!
  
Your review is under verification. We'll email you within 24 hours
when it's approved.

In the meantime:
→ Continue browsing
→ Share ReviewsRealty with others
→ Check the FAQ
```

**Email Sent To Reviewer:**
```
Subject: Review Submitted - Verification in Progress [ReviewsRealty]

Hi [First Name],

Thanks for sharing your honest review! We're verifying the details
to keep ReviewsRealty trustworthy.

Status: PENDING_VERIFICATION
Review ID: REV-2026-02-04-12345

We'll notify you within 24 hours once approved.

Have questions? Reply to this email.

— The ReviewsRealty Team
```

---

## Phase 6: Verification Layer (Backend - Automated)

### 6.1 Email Verification

**Trigger:** Within 1 minute of submission

- Send verification email with 6-digit code
- Code valid for 24 hours
- Reviewer must click link or enter code
- Until verified: Review stays in PENDING_VERIFICATION

**Email Template:**
```
Subject: Confirm Your ReviewsRealty Account [6-digit code]

Verify your email to activate your review:

[VERIFY EMAIL BUTTON] or enter code: 123456

This code expires in 24 hours.
```

### 6.2 Spam & Fraud Detection

**Automated checks (real-time):**

| Check | Threshold | Action |
|-------|-----------|--------|
| Review length | < 50 chars | REJECT (already validated client-side) |
| All CAPS words | > 30% | FLAG for admin review |
| Repeated characters | "heeelllpppp" | FLAG as likely spam |
| Blacklisted domains | Known spam IPs | REJECT + block IP |
| Same user > 3 reviews/day | Per user | RATE_LIMIT (queue for manual review) |
| Duplicate review | Same address + agent + 90% text match | FLAG for review |

### 6.3 Agent/Property Validation

| Check | Validation |
|-------|-----------|
| **Agent Exists** | Query database; if not found, create stub profile for new agents |
| **Property Exists** | Geocode address; if new, create property page |
| **Address Valid** | Use Google Maps API; reject if address invalid |
| **Agent-Property Link** | Cross-check: did this agent work this property? (Optional - can be fuzzy for MVP) |

### 6.4 Document Verification (If Uploaded)

**OCR Processing:**
- Extract text from uploaded document
- Check if contains: property address, agent name, or transaction date
- If OCR confidence < 70% → FLAG for manual admin review
- If no matching text found → FLAG for manual admin review

**Not checking for:** Authenticity of document signature (too hard to automate). Admin will verify.

---

## Phase 7: Admin Review & Moderation

### 7.1 Admin Dashboard

**For each pending review, admin sees:**

```
┌─────────────────────────────────────────────────┐
│ PENDING REVIEW #REV-2026-02-04-12345           │
├─────────────────────────────────────────────────┤
│ Reviewer: Sarah Johnson                         │
│ Status: EMAIL_VERIFIED ✓ | DOCUMENT: ⚠ REVIEW │
│ Risk Score: 23% (LOW)                           │
│                                                 │
│ REVIEW DATA:                                    │
│ ─────────────────────────────────────────────── │
│ Agent: John Smith, Miami Realty                │
│ Property: 123 Main St, Miami, FL 33139         │
│ Rating: ⭐⭐⭐⭐⭐ (5 stars)                      │
│ Transaction: Buyer (Jan 2023)                  │
│ Submitted: 2026-02-04 14:30 UTC                │
│                                                 │
│ REVIEW TEXT:                                    │
│ "John was incredibly responsive and helped    │
│  me navigate a complex offer. Highly           │
│  recommend for anyone serious about buying."  │
│                                                 │
│ ATTACHED DOCUMENTS:                             │
│ • closing_statement.pdf                        │
│   └─ OCR: "123 Main St" ✓, "John Smith" ✓    │
│                                                 │
│ [APPROVE] [REJECT] [REQUEST MORE INFO]        │
│ [VIEW REVIEWER HISTORY]                        │
└─────────────────────────────────────────────────┘
```

### 7.2 Approval Criteria

**Admin should APPROVE if:**
- ✓ Email verified
- ✓ Document present & OCR extracts key data
- ✓ Review text is coherent (50+ chars, English)
- ✓ Rating matches sentiment (5 stars + positive text = good)
- ✓ No obvious spam flags
- ✓ Agent/property exist in system

**Admin should REJECT if:**
- ✗ Spam content (all CAPS, random text)
- ✗ Fake agent name (e.g., "John Fake Realtor")
- ✗ Impossible timeline (review dated before purchase)
- ✗ Abusive/harassing language
- ✗ Obvious fraud (same person submitting identical reviews)
- ✗ Document unreadable or clearly fake

**Admin should REQUEST MORE INFO if:**
- ⚠️ Document OCR failed but text looks legitimate
- ⚠️ Address in review doesn't match document
- ⚠️ Ambiguous agent name (multiple matches in system)

### 7.3 Admin Actions

| Action | Result | Notification |
|--------|--------|--------------|
| **APPROVE** | Review published, goes live immediately | Email: "Your review is now live!" |
| **REJECT** | Review archived, not published | Email: "Your review was declined. Here's why..." |
| **REQUEST INFO** | Sends email asking for clarification | "We need more info to verify your review..." |
| **HOLD** | Remains pending, manual escalation | Admin reviews later |

### 7.4 Rejection Reasons (Template)

When rejecting, admin selects reason (shown to reviewer):

```
Your review was not approved because:

☐ The review text appears to be spam or auto-generated
☐ The document provided doesn't match the review content
☐ The transaction details don't align with the review
☐ The review violates our community guidelines
☐ We detected duplicate reviews from your account

You can edit and resubmit, or contact support@reviewsrealty.com
```

---

## Phase 8: Publication (Review Goes Live)

### 8.1 Review Status Changed to APPROVED

**Automatic actions:**
1. Set review.status = "APPROVED"
2. Set review.published_at = now()
3. Set review.verified_badge = true (email + document verified)
4. Update agent aggregate rating (refresh cache)
5. Update property aggregate rating (refresh cache)
6. Send email to reviewer: "Your review is now live!"

### 8.2 Where Review Appears

**Location 1: Agent Profile Page**
```
John Smith, Miami Realty
⭐ 4.6 (28 verified reviews)

RECENT REVIEWS:
┌─────────────────────┐
│ ⭐⭐⭐⭐⭐ (5 stars) │
│ Sarah Johnson - Buyer (Jan 2023)
│ Property: 123 Main St, Miami, FL
│ "John was incredibly responsive..." │
│ Verified ✓ | 14 days ago      │
└─────────────────────┘
```

**Location 2: Property Page**
```
123 Main St, Miami, FL 33139
⭐ 4.8 (3 reviews for this property)

ALL REVIEWS FOR THIS PROPERTY:
┌─────────────────────┐
│ Agent: John Smith │
│ ⭐⭐⭐⭐⭐ (5 stars) │
│ Buyer - "John was incredibly..."  │
│ Verified ✓ | 14 days ago      │
└─────────────────────┘
```

### 8.3 Rating Aggregation

**Agent Rating Formula:**
```
agent_rating = SUM(review.rating) / COUNT(verified_reviews_only)
```

**Important:** Only verified reviews count toward aggregate rating.

**Unverified reviews:**
- Still shown on profile with "Pending" badge
- Don't affect aggregate rating
- Have disclaimer: "This review hasn't been verified yet"

---

## Phase 9: Post-Publication Moderation

### 9.1 Ongoing Checks

**Auto-flagging:**
- If review gets 3+ "Inappropriate" flags → Admin review
- If agent claims review is false → Admin contacts reviewer, escalates

**Manual Review:**
- Spot-check 5% of published reviews daily
- Look for patterns (same user, similar language)
- Remove fake reviews if detected (+ warn user)

### 9.2 Review Flags & Reporting

**Users can flag reviews as:**
- Spam
- Inappropriate language
- Doesn't match description
- Possibly false

**3+ flags → Admin review → Possible removal**

---

## Timeline & SLAs

| Phase | Target Duration | Escalation |
|-------|-----------------|-----------|
| Submission → Email Verification | 1-5 minutes | Auto-verify or manual within 24h |
| Email Verified → Admin Review | < 24 hours | Escalate to manager if > 48h |
| Admin Review → Published | < 2 hours (assume 24h for scale) | Daily batch review |
| **Total: Submission → Published** | **< 48 hours** | **SLA: 72 hours max** |

---

## Error Handling & Edge Cases

### E1: Document Upload Fails
**User Experience:**
- Show error: "File upload failed. Try again."
- Offer alternative: "Email proof to verify@reviewsrealty.com"
- Allow submission with "Unverified" status

### E2: Email Not Verified in 24h
- Auto-reminder email: "Confirm your email to activate your review"
- If still not verified in 7 days: Archive review + notify user

### E3: Same User Submits Duplicate Review
- System detects: Same user + same agent + similar text
- Options:
  - Auto-reject as duplicate
  - Merge reviews (edit + resubmit)
  - Allow if transaction details different

### E4: Agent Claims Review is Fake
- Admin contacts reviewer with proof request
- If reviewer can't provide → Review marked "Disputed"
- If confirmed fraud → Review removed + user banned

### E5: Property Address Not Found
- User can manually enter property details
- Admin verifies & geocodes address later
- Review still publishes, property page created on-the-fly

---

## Security & Compliance

### Data Protection
- ✓ Transaction documents encrypted at rest (AES-256)
- ✓ Never shown to public
- ✓ Only accessible to admin via secure login
- ✓ Deleted after 90 days (unless user requests longer)

### GDPR Compliance
- ✓ User can request data export (review + metadata)
- ✓ User can delete account + reviews
- ✓ Right to be forgotten implemented
- ✓ Privacy policy clear on document handling

### CCPA Compliance
- ✓ California residents can opt-out of data sales (N/A - not selling data)
- ✓ Can view/delete personal info
- ✓ Clear disclosures in signup

---

## Success Metrics & Monitoring

### Key Metrics to Track

| Metric | Target | How to Measure |
|--------|--------|----------------|
| **Reviews Published/Day** | 10+ | Count approved reviews |
| **Avg Verification Time** | < 12h | published_at - submitted_at |
| **Approval Rate** | > 85% | approved / submitted |
| **Bounce Rate (Form Start → Submit)** | < 20% | tracking pixels |
| **Spam Detection Rate** | > 95% | flag_count / total_submitted |
| **User Satisfaction** | 4.5+ (NPS) | Email survey post-publish |

### Monitoring Dashboard (For Team)
- Real-time pending review count
- Average time-to-approval per moderator
- Spam detection rate
- Document rejection rate (OCR failures)
- Reviewer return rate (users who submit 2+ reviews)

---

## Implementation Checklist

### Before Launch
- [ ] Database schema for reviews, documents, verification logs
- [ ] Email service integration (SendGrid for verification + notifications)
- [ ] S3/R2 bucket for document storage (encrypted)
- [ ] OCR library integration (AWS Textract or similar)
- [ ] Admin dashboard built out
- [ ] Spam detection rules configured
- [ ] Moderation SLA tracker
- [ ] Email templates (10+ templates needed)
- [ ] Legal review (ToS updated re: verification, privacy)

### Post-Launch (Week 1)
- [ ] Daily admin review of flagged content
- [ ] Monitor spam detection accuracy
- [ ] Collect user feedback on form UX
- [ ] Iterate on rejection messaging if complaints

---

## Future Enhancements (Phase 2)

1. **Video Reviews** - Allow users to submit 30-sec video reviews
2. **Review Editing** - Let users edit reviews (with admin approval)
3. **Reviewer Verification UI** - Show "Verified Buyer" badge on profile
4. **Review Filters** - "Show only verified reviews from Q4 2023"
5. **Comparison Tool** - "See how this agent compares to others in Miami"

---

*Workflow defined by Clawdy - 2026-02-04*
*Ready for frontend & backend implementation*
