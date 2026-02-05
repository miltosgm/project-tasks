# ElevenLabs API Integration Guide

**Status:** Ready to implement  
**Timeline:** 1-2 hours setup + testing  
**Impact:** Converts Voice AI from simulated → production-ready with real voice generation

---

## 1. ElevenLabs Account Setup

### Free Tier (Starter)
- **Cost:** Free
- **Monthly quota:** 10,000 characters
- **Best for:** Development, testing, MVP validation
- **Signup:** https://elevenlabs.io/sign-up

### Pro Tier ($11/month)
- **Cost:** $11/month
- **Monthly quota:** 100,000 characters
- **Best for:** Production MVP (handles ~10K reminders/month)
- **Startup Grant:** ElevenLabs offers 50% discount for startups (apply at https://elevenlabs.io/startup-program)

### Business Tier (Custom)
- **Cost:** Contact sales
- **Monthly quota:** 1M+ characters
- **Best for:** Scaling beyond 50K+ calls/month
- **Features:** Dedicated API support, SLA, custom voice training

**Recommendation for MVP:** Start with Pro tier ($11/month) after applying for startup grant (should get 50% off = $5.50/month). Handles 100K characters = ~500-1000 reminder calls.

---

## 2. Getting Your API Key

1. Sign up at https://elevenlabs.io
2. Go to **Settings → API Keys**
3. Click **"Create new API key"**
4. Name it: `Voice AI Dental Reminders`
5. Copy the key (never share publicly!)
6. Store in `.env` file:
   ```
   ELEVENLABS_API_KEY=your_api_key_here
   ```

---

## 3. Voice ID Setup

ElevenLabs provides pre-built voices or you can create a custom voice clone.

### Option A: Use Built-in Voice (Recommended for MVP)
Pre-made professional voices ready to use:

| Voice ID | Voice Name | Best For |
|----------|-----------|----------|
| `EXAVITQu4MsSD4OA5j47` | Bella | Warm, friendly female |
| `EXAVITQu4MsSD4OA5j47` | Adam | Professional male |
| `pMsXgVXv3BLzUgSXRsSj` | Sam | Casual male |
| `9BWtsMINqrJLrRacOk9x` | Aria | Clear female |

Get full list at: https://api.elevenlabs.io/v1/voices

### Option B: Custom Voice Clone (Advanced)
Record 5-10 minute voice sample of yourself speaking naturally, then:
1. Go to **Voice Library → My Voices**
2. Click **"Create new voice"**
3. Upload samples (wav, mp3, m4a)
4. ElevenLabs trains custom voice (~10 minutes)
5. Use returned `voice_id` in API calls

**Recommendation:** Use built-in "Bella" or "Adam" for MVP. Custom voice clone after market validation.

---

## 4. API Endpoint Reference

### Text-to-Speech (Main Endpoint)

```
POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}

Content-Type: application/json

Body:
{
  "text": "Hi Sarah Johnson, this is a reminder call from Miami Dental Care...",
  "model_id": "eleven_monolingual_v1",
  "voice_settings": {
    "stability": 0.5,
    "similarity_boost": 0.75
  }
}

Headers:
xi-api-key: your_api_key_here

Response:
200 OK
Content-Type: audio/mpeg
(Audio file as binary MP3 data)
```

### Get Available Voices

```
GET https://api.elevenlabs.io/v1/voices

Headers:
xi-api-key: your_api_key_here

Response:
{
  "voices": [
    {
      "voice_id": "EXAVITQu4MsSD4OA5j47",
      "name": "Bella",
      "samples": [...],
      "category": "generated",
      "settings": {...}
    },
    ...
  ]
}
```

### Get User Subscription Info

```
GET https://api.elevenlabs.io/v1/user/subscription

Headers:
xi-api-key: your_api_key_here

Response:
{
  "character_count": 2150,
  "character_limit": 10000,
  "can_use_professional_voice_consistency": true,
  "can_use_streaming_api": true,
  "subscription_tier": "starter"
}
```

---

## 5. Backend Implementation (Node.js)

### Step 1: Install dependencies

```bash
npm install axios dotenv
```

### Step 2: Create ElevenLabs service

Create file: `services/elevenlabs-service.js`

```javascript
const axios = require('axios');

class ElevenLabsService {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.elevenlabs.io/v1';
    this.voiceId = 'EXAVITQu4MsSD4OA5j47'; // Bella voice
  }

  async textToSpeech(text, options = {}) {
    try {
      const voiceId = options.voiceId || this.voiceId;
      const stability = options.stability || 0.5;
      const similarityBoost = options.similarityBoost || 0.75;

      const response = await axios.post(
        `${this.baseUrl}/text-to-speech/${voiceId}`,
        {
          text,
          model_id: 'eleven_monolingual_v1',
          voice_settings: {
            stability,
            similarity_boost: similarityBoost
          }
        },
        {
          headers: {
            'xi-api-key': this.apiKey,
            'Content-Type': 'application/json'
          },
          responseType: 'arraybuffer'
        }
      );

      // Return audio as base64 for easy transmission
      return {
        success: true,
        audio: Buffer.from(response.data).toString('base64'),
        mimeType: 'audio/mpeg',
        textLength: text.length,
        characterCount: text.length
      };
    } catch (error) {
      return {
        success: false,
        error: error.response?.data?.detail || error.message,
        statusCode: error.response?.status
      };
    }
  }

  async getVoices() {
    try {
      const response = await axios.get(`${this.baseUrl}/voices`, {
        headers: {
          'xi-api-key': this.apiKey
        }
      });

      return {
        success: true,
        voices: response.data.voices
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }

  async getUserSubscription() {
    try {
      const response = await axios.get(`${this.baseUrl}/user/subscription`, {
        headers: {
          'xi-api-key': this.apiKey
        }
      });

      return {
        success: true,
        subscription: response.data
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }
}

module.exports = ElevenLabsService;
```

### Step 3: Create API route

Create file: `routes/voice-reminder-api.js`

```javascript
const express = require('express');
const ElevenLabsService = require('../services/elevenlabs-service');
const router = express.Router();

const elevenLabs = new ElevenLabsService(process.env.ELEVENLABS_API_KEY);

// Generate voice reminder
router.post('/api/voice-reminder', async (req, res) => {
  try {
    const {
      patientName,
      dentistName,
      practiceName,
      appointmentTime,
      appointmentType,
      voiceId = 'EXAVITQu4MsSD4OA5j47'
    } = req.body;

    // Validate inputs
    if (!patientName || !practiceName || !appointmentTime) {
      return res.status(400).json({
        success: false,
        error: 'Missing required fields: patientName, practiceName, appointmentTime'
      });
    }

    // Build reminder message
    const message = `Hi ${patientName}, this is a reminder call from ${practiceName}. You have an appointment with ${dentistName} for ${appointmentType.toLowerCase()} scheduled for ${appointmentTime}. Please press 1 to confirm, 2 to reschedule, or 3 to cancel. Thank you!`;

    // Generate speech
    const result = await elevenLabs.textToSpeech(message, { voiceId });

    if (!result.success) {
      return res.status(400).json({
        success: false,
        error: result.error
      });
    }

    res.json({
      success: true,
      audio: result.audio,
      message,
      characterCount: result.characterCount,
      mimeType: result.mimeType
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

// Get available voices
router.get('/api/voices', async (req, res) => {
  try {
    const result = await elevenLabs.getVoices();
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get user subscription info
router.get('/api/user-subscription', async (req, res) => {
  try {
    const result = await elevenLabs.getUserSubscription();
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

module.exports = router;
```

### Step 4: Add to Express server

```javascript
const express = require('express');
const voiceRoutes = require('./routes/voice-reminder-api');

const app = express();
app.use(express.json());
app.use(voiceRoutes);

app.listen(3000, () => {
  console.log('Voice API running on port 3000');
});
```

---

## 6. Frontend Integration

### Updated HTML/JS Demo (Real ElevenLabs)

```html
<!-- Inside your form -->
<div class="form-group">
  <label>API Key</label>
  <input type="password" id="api-key" placeholder="Enter your ElevenLabs API key">
</div>

<div class="form-group">
  <label>Voice</label>
  <select id="voice-select">
    <option value="EXAVITQu4MsSD4OA5j47">Bella (Female)</option>
    <option value="pNInY14gBaLJ2v4u18QG">Adam (Male)</option>
  </select>
</div>

<button onclick="generateReminderWithElevenLabs()">Generate Voice with ElevenLabs</button>
```

### JavaScript with ElevenLabs

```javascript
async function generateReminderWithElevenLabs() {
  const apiKey = document.getElementById('api-key').value;
  const patientName = document.getElementById('patient-name').value;
  const voiceId = document.getElementById('voice-select').value;

  if (!apiKey) {
    alert('Please enter your ElevenLabs API key');
    return;
  }

  // Reset log
  document.getElementById('log').innerHTML = '';
  document.getElementById('log').classList.remove('empty');

  addLog('VALIDATE', 'Checking inputs...');

  setTimeout(async () => {
    try {
      addLog('API_CALL', 'Sending request to ElevenLabs API...');

      const response = await fetch('https://api.elevenlabs.io/v1/text-to-speech/' + voiceId, {
        method: 'POST',
        headers: {
          'xi-api-key': apiKey,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          text: `Hi ${patientName}, this is a reminder call from Miami Dental Care...`,
          model_id: 'eleven_monolingual_v1',
          voice_settings: {
            stability: 0.5,
            similarity_boost: 0.75
          }
        })
      });

      if (!response.ok) {
        const error = await response.json();
        addLog('ERROR', error.detail || 'API request failed', true);
        return;
      }

      addLog('AUDIO_GEN', 'Audio generated successfully');

      const audioBlob = await response.blob();
      const audioUrl = URL.createObjectURL(audioBlob);

      // Store for playback
      lastAudioUrl = audioUrl;
      lastMessage = `Hi ${patientName}...`;

      addLog('READY', 'Voice message ready to play');
      updateStatusBadge('success', 'Message Ready');

    } catch (error) {
      addLog('ERROR', error.message, true);
    }
  }, 500);
}

// Updated play function
function playVoice() {
  if (!lastAudioUrl) {
    alert('Please generate a message first');
    return;
  }

  const audio = new Audio(lastAudioUrl);
  audio.play();

  addLog('PLAY', 'Playing ElevenLabs voice message...');
}
```

---

## 7. Rate Limiting & Cost Management

### Character Counting
ElevenLabs charges per character (including spaces).

**Example costs:**
- Single reminder: ~150 characters = 0.015 cents
- 1000 reminders/month: $0.15/month + API overhead

### Rate Limiting Strategy

```javascript
class RateLimiter {
  constructor(monthlyCharacters = 100000) {
    this.monthlyLimit = monthlyCharacters;
    this.used = 0;
    this.monthStart = new Date();
  }

  canGenerate(text) {
    const newTotal = this.used + text.length;
    if (newTotal > this.monthlyLimit) {
      return {
        allowed: false,
        message: `Would exceed monthly limit. Used: ${this.used}/${this.monthlyLimit}`
      };
    }
    return { allowed: true };
  }

  track(text) {
    this.used += text.length;
  }
}
```

### Daily Quota Tracking

Store in database:
```sql
CREATE TABLE usage_logs (
  id INT PRIMARY KEY,
  practice_id INT,
  date DATE,
  characters_used INT,
  api_calls INT,
  created_at TIMESTAMP
);
```

---

## 8. Error Handling

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API key | Check `.env` file, regenerate key |
| 400 Bad Request | Missing required fields | Validate JSON structure |
| 429 Too Many Requests | Rate limited | Implement queue system |
| 500 Server Error | ElevenLabs outage | Implement retry logic |

### Retry Logic

```javascript
async function textToSpeechWithRetry(text, voiceId, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await textToSpeech(text, voiceId);
    } catch (error) {
      if (i < maxRetries - 1) {
        // Exponential backoff: 1s, 2s, 4s
        await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
      } else {
        throw error;
      }
    }
  }
}
```

---

## 9. Testing Checklist

- [ ] ElevenLabs account created
- [ ] API key generated and stored in `.env`
- [ ] Voice ID selected (or custom voice trained)
- [ ] textToSpeech API call successful (test with curl)
- [ ] Backend service integration tested
- [ ] Frontend form accepts API key
- [ ] Audio playback works in browser
- [ ] Error handling tested (invalid key, missing fields)
- [ ] Rate limiting implemented and tested
- [ ] Character quota tracking implemented
- [ ] All voice options available in dropdown

---

## 10. Production Deployment Checklist

- [ ] API key stored securely (environment variable, not hardcoded)
- [ ] Frontend API key never exposed (use backend proxy)
- [ ] Rate limiting enforced on all endpoints
- [ ] Error messages don't expose sensitive data
- [ ] Audio generation logged for audits
- [ ] Monthly usage dashboard created for monitoring
- [ ] Backup voice provider ready (if ElevenLabs unavailable)
- [ ] SLA for voice generation (target: <2s latency)
- [ ] User authentication for API endpoints
- [ ] Usage limits per practice enforced

---

## Next Steps

1. **Sign up** for ElevenLabs free tier
2. **Get API key** from Settings
3. **Test locally** with provided service code
4. **Record custom voice** (optional, after MVP validation)
5. **Deploy to production** with all security measures
6. **Monitor costs** and optimize as needed

---

## Cost Projection (Annual)

| Scenario | Monthly Calls | Monthly Characters | Monthly Cost | Annual Cost |
|----------|---------------|-------------------|--------------|-------------|
| Tiny (5 practices) | 1,000 | 150,000 | $1.50 | $18 |
| Small (50 practices) | 10,000 | 1.5M | $15 | $180 |
| Growing (500 practices) | 100,000 | 15M | $150 | $1,800 |
| Scale (5,000 practices) | 1,000,000 | 150M | $1,500 | $18,000 |

*Assumes $100/month API tier for 1M+ characters. Adjust based on actual usage.*

---

## Support & Resources

- **Documentation:** https://docs.elevenlabs.io
- **API Reference:** https://api.elevenlabs.io
- **Community:** https://discord.com/invite/elevenlabs
- **Status:** https://status.elevenlabs.io
