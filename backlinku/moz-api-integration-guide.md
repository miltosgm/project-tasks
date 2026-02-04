# BacklinkU - Moz API Integration Guide
**Date:** 2026-02-05  
**Project:** BacklinkU  
**Status:** Implementation-ready

---

## Overview

This guide shows how to integrate the BacklinkU MVP tool with the Moz Link API to replace simulated data with real backlink metrics.

**Current state:** MVP tool uses demo data  
**After integration:** Real Domain Authority, backlinks, referring domains, trust scores  
**Cost:** $990/year (Moz Pro plan) or free tier for testing  

---

## Step 1: Get Moz API Access

### Option A: Free Tier (for testing/demo)
1. Visit: https://moz.com/link-explorer
2. Sign up with free account
3. Get API key: https://moz.com/products/api/authentication
4. Daily limit: 100 requests/day (sufficient for MVP testing)
5. Cost: $0

### Option B: Pro Plan (production)
1. Subscribe to Moz API Pro: https://moz.com/products/api/pricing
2. Plan: **Advanced** (recommended for BacklinkU)
   - Cost: ~$2,500/month (4M rows/month)
   - Calls: ~4 calls per domain = ~1M domains analyzable
3. For small MVP: Start with free tier, upgrade when needed

---

## Step 2: Moz API Endpoints

### Core Endpoints for BacklinkU

**1. Get Domain Authority**
```
GET /v2/domain-authority
Parameters:
  - domain: example.com
  - auth: {your-moz-api-key}
Response:
  {
    "domain_authority": 68,
    "spam_score": 12,
    "linking_domains": 3200,
    "inbound_links": 12400
  }
```

**2. Get Link Data**
```
GET /v2/backlinks
Parameters:
  - domain: example.com
  - limit: 100
  - auth: {your-moz-api-key}
Response:
  {
    "links": [
      {
        "source_domain": "forbes.com",
        "source_url": "https://forbes.com/article",
        "target_url": "https://example.com/page",
        "anchor_text": "best crm",
        "link_authority": 45,
        "link_equity": 8.2
      }
      ...
    ],
    "total_backlinks": 12400
  }
```

**3. Get Referral Domains**
```
GET /v2/referring-domains
Parameters:
  - domain: example.com
  - auth: {your-moz-api-key}
Response:
  {
    "referring_domains": 3200,
    "domains": [
      {
        "domain": "forbes.com",
        "domain_authority": 87,
        "link_count": 45,
        "linking_c_blocks": 43
      }
      ...
    ]
  }
```

---

## Step 3: Backend Integration (Node.js Example)

### 3.1 Install Dependencies

```bash
npm install axios dotenv express
```

### 3.2 Create `.env` File

```
MOZ_API_KEY=YOUR_API_KEY_HERE
MOZ_API_ENDPOINT=https://api.moz.com/v2
NODE_ENV=production
```

### 3.3 Create API Service (`/api/backlink-service.js`)

```javascript
const axios = require('axios');
require('dotenv').config();

const MOZ_ENDPOINT = process.env.MOZ_API_ENDPOINT;
const MOZ_KEY = process.env.MOZ_API_KEY;

class BacklinkService {
  constructor() {
    this.mozClient = axios.create({
      baseURL: MOZ_ENDPOINT,
      auth: {
        username: MOZ_KEY,
        password: '' // Moz uses empty password
      }
    });
  }

  /**
   * Get domain metrics from Moz
   * @param {string} domain - Domain to analyze (e.g., example.com)
   * @returns {object} Domain metrics
   */
  async getDomainMetrics(domain) {
    try {
      const response = await this.mozClient.get('/domain-authority', {
        params: { domain }
      });

      return {
        da: response.data.domain_authority || 0,
        spam: response.data.spam_score || 0,
        backlinks: response.data.inbound_links || 0,
        domains: response.data.linking_domains || 0,
        timestamp: new Date(),
        source: 'moz'
      };
    } catch (error) {
      console.error(`Moz API error for ${domain}:`, error.message);
      throw new Error(`Failed to fetch metrics for ${domain}`);
    }
  }

  /**
   * Get backlink details
   * @param {string} domain - Domain to analyze
   * @param {number} limit - Number of backlinks to return
   * @returns {array} Backlink details
   */
  async getBacklinks(domain, limit = 50) {
    try {
      const response = await this.mozClient.get('/backlinks', {
        params: { 
          domain,
          limit,
          sort: 'link_authority',
          ascending: false
        }
      });

      return response.data.links || [];
    } catch (error) {
      console.error(`Backlinks fetch error:`, error.message);
      throw new Error(`Failed to fetch backlinks for ${domain}`);
    }
  }

  /**
   * Get referring domains
   * @param {string} domain - Domain to analyze
   * @returns {array} Referring domains with authority
   */
  async getReferringDomains(domain) {
    try {
      const response = await this.mozClient.get('/referring-domains', {
        params: { domain }
      });

      return {
        total: response.data.referring_domains || 0,
        domains: response.data.domains || [],
        source: 'moz'
      };
    } catch (error) {
      console.error(`Referring domains error:`, error.message);
      throw new Error(`Failed to fetch referring domains for ${domain}`);
    }
  }

  /**
   * Calculate trust score (0-100) based on Moz data
   * @param {object} metrics - Domain metrics
   * @returns {number} Trust score
   */
  calculateTrustScore(metrics) {
    // Formula: 40% DA + 30% (links/100 capped at 100) + 30% (domains/100 capped at 100) - spam_score
    const daScore = (metrics.da / 100) * 40;
    const backlinksScore = Math.min((metrics.backlinks / 100), 100) * 0.30;
    const domainsScore = Math.min((metrics.domains / 100), 100) * 0.30;
    const spamPenalty = metrics.spam * 0.5;
    
    const trust = Math.max(0, Math.min(100, daScore + backlinksScore + domainsScore - spamPenalty));
    return Math.round(trust);
  }
}

module.exports = new BacklinkService();
```

### 3.4 Create API Endpoint (`/api/analyze.js`)

```javascript
const express = require('express');
const backlinker = require('./backlink-service');

const router = express.Router();

/**
 * POST /api/analyze
 * Body: { domain: "example.com" }
 * Returns: Full domain analysis with Moz data
 */
router.post('/analyze', async (req, res) => {
  const { domain } = req.body;

  if (!domain) {
    return res.status(400).json({ error: 'Domain required' });
  }

  try {
    // Get main metrics
    const metrics = await backlinker.getDomainMetrics(domain);
    const trust = backlinker.calculateTrustScore(metrics);

    // Get backlinks (top 10)
    const backlinks = await backlinker.getBacklinks(domain, 10);

    // Get referring domains
    const referringDomains = await backlinker.getReferringDomains(domain);

    return res.json({
      domain,
      da: metrics.da,
      backlinks: metrics.backlinks,
      domains: metrics.domains,
      trust,
      spam: metrics.spam,
      topBacklinks: backlinks,
      referringDomains: referringDomains.domains.slice(0, 10),
      timestamp: metrics.timestamp
    });
  } catch (error) {
    console.error('Analysis error:', error);
    return res.status(500).json({ 
      error: error.message 
    });
  }
});

module.exports = router;
```

---

## Step 4: Frontend Integration

### Update HTML to Call Real API

Replace the simulated `generateReminder()` in `backlink-checker-mvp.html`:

```javascript
async function analyzeDomain() {
  const input = document.getElementById('domain-input').value.trim().toLowerCase();
  
  if (!input) {
    showError('Please enter a domain name');
    return;
  }

  // Clean domain
  let domain = input
    .replace(/^https?:\/\//, '')
    .replace(/^www\./, '')
    .split('/')[0];

  showLoading();
  clearError();

  try {
    // Call backend API
    const response = await fetch('/api/analyze', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ domain })
    });

    if (!response.ok) {
      throw new Error('Domain analysis failed');
    }

    const data = await response.json();
    displayResults(domain, data);
  } catch (error) {
    hideLoading();
    showError(error.message);
  }
}
```

---

## Step 5: Caching Strategy (Optional but Recommended)

To avoid hitting API limits, cache results for 7 days:

```javascript
// cache-service.js
const fs = require('fs');
const path = require('path');

class CacheService {
  constructor(cacheDir = './cache') {
    this.cacheDir = cacheDir;
    if (!fs.existsSync(cacheDir)) {
      fs.mkdirSync(cacheDir, { recursive: true });
    }
  }

  getCacheFile(domain) {
    return path.join(this.cacheDir, `${domain}.json`);
  }

  isCacheValid(cacheFile, maxAge = 7 * 24 * 60 * 60 * 1000) {
    // 7 days default
    if (!fs.existsSync(cacheFile)) return false;
    
    const stat = fs.statSync(cacheFile);
    return Date.now() - stat.mtimeMs < maxAge;
  }

  get(domain) {
    const file = this.getCacheFile(domain);
    if (this.isCacheValid(file)) {
      return JSON.parse(fs.readFileSync(file, 'utf-8'));
    }
    return null;
  }

  set(domain, data) {
    const file = this.getCacheFile(domain);
    fs.writeFileSync(file, JSON.stringify(data, null, 2));
  }
}

module.exports = new CacheService();
```

Update API endpoint to use cache:

```javascript
const cache = require('./cache-service');

router.post('/analyze', async (req, res) => {
  const { domain } = req.body;
  
  // Check cache first
  const cached = cache.get(domain);
  if (cached) {
    return res.json({ ...cached, cached: true });
  }

  // ... rest of API logic
  
  // Save to cache
  cache.set(domain, results);
  return res.json(results);
});
```

---

## Step 6: Error Handling & Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

// Limit: 10 requests per minute per IP
const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  message: 'Too many requests, please try again later'
});

app.post('/api/analyze', limiter, async (req, res) => {
  // ...
});
```

---

## Step 7: Deployment Checklist

- [ ] Get Moz API key
- [ ] Set up `.env` file with credentials
- [ ] Install Node dependencies
- [ ] Test API locally with sample domains
- [ ] Implement caching strategy
- [ ] Add rate limiting
- [ ] Deploy to hosting (Vercel, Heroku, AWS)
- [ ] Update frontend to call real API
- [ ] Test production domain analysis
- [ ] Monitor API usage vs. plan limits
- [ ] Set up error logging/alerts

---

## Cost Analysis

**Free Tier:**
- Cost: $0/month
- Limit: 100 requests/day = 3 domains/day
- Use case: Personal testing, demo site

**Moz Pro (Advanced):**
- Cost: $2,500/month
- Limit: 4M rows/month
- Domains analyzable: ~1M (assuming 4 calls/domain)
- Cost per domain: $0.0025

**Pricing for BacklinkU Users:**
- Free tier: 3 free analyses/day
- Premium: $19-99/month for unlimited
- At $49/month, need ~20 premium users to break even at scale

---

## Testing

### Test with Free API Tier

```bash
curl -X POST http://localhost:3000/api/analyze \
  -H "Content-Type: application/json" \
  -d '{"domain":"example.com"}'
```

### Expected Response

```json
{
  "domain": "example.com",
  "da": 68,
  "backlinks": 12400,
  "domains": 3200,
  "trust": 75,
  "spam": 12,
  "topBacklinks": [...],
  "referringDomains": [...],
  "timestamp": "2026-02-05T10:30:00Z"
}
```

---

## Next Steps

1. ✅ Get Moz API key (free tier or paid plan)
2. ✅ Set up Node backend with integration code above
3. ✅ Update frontend to call real API
4. ✅ Test with 5-10 sample domains
5. ✅ Deploy to production
6. ✅ Monitor API usage & costs
7. ✅ Add domain-specific insights (top backlinks, competitor analysis)
8. ✅ Launch premium tier ($49-99/month)

---

## Resources

- Moz API Docs: https://moz.com/products/api
- Moz API Authentication: https://moz.com/products/api/authentication
- Rate limits: https://moz.com/help/guides/moz-api/getting-started/rate-limits
- Sample code: https://github.com/seomoz/moz-api-examples

---

*Integration guide ready for implementation - Clawdy, Feb 5, 2026*
