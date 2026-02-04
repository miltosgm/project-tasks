# ReviewsRealty MVP Features Definition
**Date:** 2026-02-04  
**Project:** ReviewsRealty  
**Status:** MVP Feature Set v1.0

## Product Vision
A verified, privacy-first review platform exclusively for real estate transactions. Focus: authentic reviews from real buyers/sellers, no incentivization, hyperlocal insights.

---

## Phase 1: MVP (Launch-Ready Features)

### 1. User Types & Roles

#### **Buyers/Sellers (Free Users)**
- Browse all reviews anonymously
- Search agents, properties, neighborhoods
- Submit reviews after verified transactions
- Upload photos with reviews
- Rate on 1-5 star scale

#### **Real Estate Agents (Freemium)**
- Free profile page with reviews
- Claim and verify their profile
- Read reviews about them
- Paid tier: Respond to reviews, enhanced profile

#### **Admins**
- Review moderation
- User management
- Verification approval

---

### 2. Core Review Features

#### **Review Submission**
- **Required fields:**
  - Transaction type (bought/sold)
  - Property address
  - Agent name
  - Star rating (1-5)
  - Written review (min 50 chars)
  - Date of transaction
  
- **Optional fields:**
  - Photos (max 5, 5MB each)
  - Specific property details (price range, property type)
  - Would recommend? (Yes/No)

#### **Verification System**
- Email verification required
- Transaction proof upload (closing docs, contract - admin reviews privately)
- Verified badge on approved reviews
- Unverified reviews shown with disclaimer until verified

#### **Review Display**
- Chronological + rating sort options
- Filter by: verified only, rating, date range, transaction type
- Agent aggregate score (average rating)
- Review count displayed
- Most helpful reviews (based on upvotes - phase 2)

---

### 3. Agent Profiles

#### **Free Agent Profile Includes:**
- Full name, agency affiliation
- Profile photo
- Aggregate rating (1-5 stars)
- Total reviews count
- All reviews displayed
- Contact info (phone, email)
- License number (verified)

#### **Paid Agent Profile Adds (Phase 2):**
- Ability to respond to reviews
- Featured badge
- Additional media (video intro, property portfolio)
- Analytics dashboard (profile views, review trends)
- Lead notifications (when someone views profile)

---

### 4. Property-Level Reviews

#### **Property Pages:**
- Address-based
- All reviews for that specific property (across different agents)
- Property history aggregation
- Neighborhood context (linked to neighborhood page)

#### **Data Shown:**
- Review count for this property
- Average rating
- All reviews chronologically
- Transaction timeline (e.g., "Sold 3 times in last 10 years")

---

### 5. Neighborhood Pages (Light Version in MVP)

#### **Basic Neighborhood Data:**
- Location (city, zip, neighborhood name)
- Aggregate rating from all reviews in area
- Review count
- Recent reviews from this neighborhood
- Link to agent activity in this area

#### **Phase 2 Adds:**
- School ratings integration
- Walkability scores
- Amenities mapping
- Price trends

---

### 6. Search & Discovery

#### **Search Types:**
1. **Agent Search:**
   - By name, agency, location
   - Sort by: rating, review count, recent activity
   
2. **Property Search:**
   - By address
   - Auto-suggest as you type
   
3. **Neighborhood Search:**
   - By city, zip, neighborhood name

#### **Filter Options:**
- Verified reviews only
- Rating (1-5 stars)
- Date range (last 30/90/365 days)
- Transaction type (buy/sell)

---

### 7. User Accounts

#### **For Reviewers (Buyers/Sellers):**
- Email + password login
- Optional: Google/Facebook OAuth
- Profile: Name, email (private)
- Review history (reviews they've submitted)
- Draft reviews (save incomplete)

#### **For Agents:**
- Email + password login
- Email verification
- Profile setup wizard
- License verification process
- Free tier by default

---

### 8. Technical Requirements

#### **Frontend:**
- Responsive design (mobile-first)
- Fast load times (<2s)
- Accessible (WCAG 2.1 AA)
- Clean, modern UI (similar to Airbnb reviews aesthetic)

#### **Backend:**
- PostgreSQL or MongoDB for data
- Photo storage (S3 or Cloudflare R2)
- Email service (SendGrid, Mailgun)
- Admin panel for moderation

#### **Security:**
- HTTPS only
- Data encryption at rest
- PII protection (don't expose reviewer emails publicly)
- GDPR compliance (EU data privacy)
- CCPA compliance (California)

#### **Performance:**
- CDN for images
- Lazy loading
- Caching for popular pages
- Server-side rendering (SSR) for SEO

---

## Phase 2: Post-MVP Enhancements

### Additional Features:
- ✅ Agent response to reviews
- ✅ Video reviews
- ✅ "Helpful" voting on reviews
- ✅ Review reporting (flag inappropriate content)
- ✅ Advanced neighborhood data (schools, amenities)
- ✅ Comparison tools (agent A vs agent B)
- ✅ Email alerts for new reviews (agents)
- ✅ API for third-party integrations

### Monetization (Phase 2):
- **Agent Premium:** $29/month or $290/year
  - Respond to reviews
  - Featured profile badge
  - Analytics dashboard
  - Priority placement in search
  
- **Agency Plans:** $99/month for 5 agents
  - All premium features per agent
  - Agency-level dashboard
  - Co-branded pages

---

## Technical Stack Recommendation

### Option A: Fast MVP (No-Code/Low-Code)
- **Frontend:** Webflow or Framer
- **Backend:** Airtable + Zapier + Memberstack
- **Pros:** Launch in 2-4 weeks
- **Cons:** Limited scalability, customization constraints

### Option B: Custom Development
- **Frontend:** Next.js (React) + Tailwind CSS
- **Backend:** Node.js + Express or Supabase
- **Database:** PostgreSQL (via Supabase)
- **Auth:** Supabase Auth or Clerk
- **Storage:** Cloudflare R2 or Supabase Storage
- **Hosting:** Vercel (frontend) + Supabase (backend)
- **Pros:** Full control, scalable, modern stack
- **Cons:** 8-12 weeks development time

### Recommended: **Option B** (Custom Development)
Why: Verification workflows, photo uploads, and review moderation need custom logic. No-code tools will be limiting long-term.

---

## User Flow Examples

### Flow 1: Buyer Leaves Review
1. Visit reviewsrealty.com
2. Click "Leave a Review"
3. Sign up / Log in
4. Enter property address
5. Select agent from dropdown (or add new agent)
6. Fill review form (rating, text, photos)
7. Upload transaction proof
8. Submit → "Under review" status
9. Admin verifies → Review goes live with "Verified" badge
10. Email confirmation sent

### Flow 2: Agent Claims Profile
1. Visit reviewsrealty.com
2. Search for their name
3. Click "Is this you? Claim profile"
4. Sign up / Log in
5. Upload license verification
6. Admin approves
7. Profile marked as "Verified Agent"
8. Agent can now view all reviews, access free tier features

### Flow 3: Home Shopper Researches Agent
1. Visit reviewsrealty.com
2. Search "John Smith, Realtor, Miami"
3. View agent profile
4. Read reviews (sort by recent/rating)
5. Filter: Verified only, last 90 days
6. See 4.7⭐ average, 28 reviews
7. Click agent's contact info
8. Reach out directly (no lead sharing from platform)

---

## Success Metrics (KPIs)

### Launch Goals (First 90 Days):
- 100+ verified reviews
- 50+ claimed agent profiles
- 500+ unique visitors/month
- 20+ organic agent signups

### Growth Metrics:
- Monthly active reviewers
- Verified review completion rate
- Agent profile claim rate
- Search traffic (organic)
- Time on site
- Review-to-agent-contact conversion

---

## Risks & Mitigations

### Risk 1: Low Initial Review Volume
**Mitigation:** 
- Seed platform with initial reviews (reach out to recent buyers in target markets)
- Partner with 2-3 small agencies for pilot
- Offer early adopter incentives (not payment, but recognition)

### Risk 2: Fake Reviews
**Mitigation:**
- Require transaction verification
- Manual admin review for first 500 reviews
- Ban users who submit fake reviews
- Clear "Verified" vs "Pending" badges

### Risk 3: Agent Pushback
**Mitigation:**
- Emphasize authenticity and consumer trust
- Offer free tier with all core features
- Show success stories from transparent agents

---

## Next Steps

1. **Wireframes:** Design key screens (homepage, agent profile, review form, search results)
2. **Tech Setup:** Set up Supabase project, create database schema
3. **Landing Page:** Deploy marketing site with waitlist
4. **Alpha Testing:** 10-20 users + 5 agents in closed beta
5. **Launch:** Public beta with target market (pick 1 city to start)

---

## Launch Timeline Estimate

- **Week 1-2:** Database schema, auth setup, basic frontend
- **Week 3-4:** Review submission flow, photo uploads
- **Week 5-6:** Agent profiles, search functionality
- **Week 7-8:** Admin panel, verification workflows
- **Week 9-10:** Testing, bug fixes, polish
- **Week 11-12:** Beta launch, collect feedback

**Total:** ~12 weeks to MVP launch

---

*Defined by Clawdy - 2026-02-04*
