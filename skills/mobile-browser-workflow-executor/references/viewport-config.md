# Viewport Configuration

## Mobile Viewport Specifications

| Device | Width | Height | DPR |
|--------|-------|--------|-----|
| iPhone 14 Pro (default) | 393 | 852 | 3 |
| iPhone SE | 375 | 667 | 2 |
| Pixel 7 | 412 | 915 | 2.625 |

## CSS Device Frame Mockup

Use this CSS to wrap screenshots in realistic iPhone frame for reports:

```css
.device-frame {
  position: relative;
  width: 393px;
  margin: 0 auto;
  border: 12px solid #1a1a1a;
  border-radius: 40px;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
  background: #1a1a1a;
}

.device-frame::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 50%;
  transform: translateX(-50%);
  width: 120px;
  height: 30px;
  background: #1a1a1a;
  border-radius: 20px;
  z-index: 10;
}

.device-frame img {
  width: 100%;
  display: block;
  border-radius: 28px;
}

.device-comparison {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(393px, 1fr));
  gap: 40px;
  margin: 40px 0;
}

.device-label {
  text-align: center;
  margin-top: 16px;
  font-size: 14px;
  color: #8E8E93;
}
```

## Session State Structure

```json
{
  "session_id": "mbwe-20260208-143022",
  "created_at": "2026-02-08T14:30:22Z",
  "mode": "audit",
  "engine": "playwright",
  "current_phase": 3,
  "workflows": [
    {
      "name": "Product Search Flow",
      "status": "completed",
      "findings": 2,
      "screenshots": ["before-001.png", "after-001.png"]
    },
    {
      "name": "Checkout Process",
      "status": "in_progress",
      "current_step": 3,
      "findings": 0
    }
  ],
  "total_findings": 2,
  "last_updated": "2026-02-08T14:45:10Z"
}
```
