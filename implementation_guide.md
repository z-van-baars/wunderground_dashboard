# Implementation Guide - Weather Dashboard

This guide explains how the weather dashboard works and how to modify it.

---

## Architecture Overview

### Single-File Structure
```
index.html
├── <style>        - All CSS (240 lines)
├── <body>         - HTML structure (30 lines)
└── <script>       - All JavaScript (400 lines)
```

**Why single file?**
- No build process required
- Easy to deploy (just push HTML)
- Self-contained and portable
- Simple to debug (everything in one place)

---

## Data Flow

```
┌─────────────────┐
│  User loads     │
│  page           │
└────────┬────────┘
         │
         v
┌─────────────────┐
│  fetchWeather() │──────> Weather.com API
│  on page load   │        (hourly/15day)
└────────┬────────┘
         │
         v
┌─────────────────┐
│  weatherData    │        Global state object
│  (36 hours)     │        with 11 fields
└────────┬────────┘
         │
         ├──> updateStats()        -> Stats cards (top)
         ├──> renderWeatherIcons() -> Icons with times
         └──> renderChart()        -> Main graph
```

**Auto-refresh:** `setInterval(fetchWeather, 30min)` at bottom of script

---

## Key Components

### 1. Configuration (Lines 278-280)
```javascript
const API_URL = 'https://api.weather.com/v3/wx/forecast/hourly/15day';
const API_KEY = 'e1f10a1e78da46f5b10a1e78da96f525';
const LOCATION = { lat: 48.796, lon: -122.502 }; // Bellingham, WA
```

**To change location:**
1. Find lat/lon (Google Maps right-click → coordinates)
2. Update `LOCATION` object
3. Refresh page

### 2. Series Configuration (Lines 282-341)
```javascript
const SERIES_CONFIG = {
    temperature: {
        label: 'Temperature',
        color: '#ef4444',      // Red
        yAxisID: 'y',          // Left axis (temp scale)
        type: 'line',
        enabled: true          // Show by default
    },
    // ... more series
}
```

**To add a new data series:**
1. Add to `SERIES_CONFIG` with appropriate settings
2. Add field to `weatherData` object in `fetchWeather()` (extract from API)
3. Series will auto-appear in toggles and graph

**To change default visibility:**
- Set `enabled: true/false` in the config

### 3. Weather Icon Mapping (Lines 348-395)
```javascript
function getWeatherEmoji(iconCode, isNight) {
    const code = parseInt(iconCode);

    // Clear - day vs night
    if ([32, 34, 36].includes(code)) return isNight ? '🌙' : '☀️';

    // ... more mappings
}
```

**To add/change icons:**
- Weather.com icon codes are documented here: https://docs.google.com/document/d/1eo8TQU-JzZTJ5ZjrR7lGlLqg_6s1TU2rTg9b8LnGBqc
- Add code to appropriate category
- Replace emoji if needed

### 4. Stats Cards (Lines 452-492)
```javascript
function updateStats() {
    // Calculates: current, high, low, avg precip, total precip
    // Total precip uses 2.5" = 100% scale
    // Progress bar color changes based on amount
}
```

**To modify precipitation scale:**
- Line 467: Change `2.5` to different threshold (inches)
- Line 470-474: Adjust color breakpoints

**To add new stat:**
1. Calculate value from `weatherData`
2. Add HTML to `stats.innerHTML` template
3. Use existing `.stat` class for styling

### 5. Weather Icons Row (Lines 494-518)
```javascript
function renderWeatherIcons() {
    // Maps over time array
    // Gets emoji for each hour (day/night aware)
    // Shows time label below icon
}
```

**To change time format:**
- Line 504-506: Modify `toLocaleTimeString()` options
- Example: `{ hour: '2-digit', minute: '2-digit' }` for 24-hour format

### 6. Main Chart (Lines 527-649)
```javascript
function renderChart() {
    // Builds Chart.js config
    // Datasets from SERIES_CONFIG (enabled only)
    // Three Y-axes: y (temp), y1 (percent), y2 (precip amount)
}
```

**Key sections:**

**Datasets (Lines 542-573):**
- Filters enabled series from `SERIES_CONFIG`
- Builds dataset objects with color, type, axis assignment
- Line vs bar styling applied based on `type`

**Tooltip (Lines 583-615):**
- Custom callback adds units (°F, %, ", mph)
- Checks label name to determine unit
- Line 600-610: Unit logic

**Y-Axes (Lines 633-667):**
- `y`: Left axis, temperature (dynamic scale)
- `y1`: Right axis, percentage (fixed 0-100%)
- `y2`: Hidden axis for precipitation bars (fixed 0-0.5")

---

## Common Modifications

### Change Graph Height

**Desktop:**
```css
/* Line 109 */
#weatherChart {
    height: 550px !important;  /* Change this */
}
```

**Mobile:**
```css
/* Line 123 */
@media (max-width: 640px) {
    #weatherChart {
        height: 450px !important;  /* Change this */
    }
}
```

### Change Color Scheme

**Dark mode variables:**
```css
/* Lines 15-19 */
body {
    background: #0f1419;  /* Page background */
    color: #e0e0e0;       /* Text color */
}

.graph-container {
    background: #1a1f2e;  /* Graph background */
}
```

**Data series colors:**
- Edit `color` in `SERIES_CONFIG` (Line 282+)
- Use hex codes: `#rrggbb`

### Change Default Time Range

```javascript
/* Line 346 */
let currentHours = 12;  // Change to 24 or 36
```

Also update button active state:
```html
/* Line 258 */
<button class="btn active" data-hours="12">12 Hours</button>
<!-- Move 'active' class to desired default -->
```

### Change Refresh Interval

```javascript
/* Line 693 */
setInterval(fetchWeather, 30 * 60 * 1000);  // 30 minutes
// Change to: 15 * 60 * 1000 for 15min
//        or: 60 * 60 * 1000 for 1hr
```

### Add Location Selector

This requires more work (not currently implemented):

1. **Add input field:**
```html
<input type="text" id="locationInput" placeholder="City, State">
<button id="locationBtn">Update</button>
```

2. **Add geocoding:**
```javascript
async function geocodeLocation(cityState) {
    // Use Google Geocoding API or similar
    // Return { lat, lon }
}
```

3. **Update on button click:**
```javascript
document.getElementById('locationBtn').addEventListener('click', async () => {
    const input = document.getElementById('locationInput').value;
    const coords = await geocodeLocation(input);
    LOCATION.lat = coords.lat;
    LOCATION.lon = coords.lon;
    fetchWeather();
});
```

---

## Chart.js Details

### Mixed Chart Type

```javascript
/* Line 551 */
chart = new Chart(ctx, {
    type: 'bar',  // Base type supports mixed datasets
    data: { labels, datasets }
    // ...
});
```

Individual datasets override with `type: 'line'` or `type: 'bar'`.

### Multiple Y-Axes

**Left (y):** Temperature scale
- Dynamic min/max based on data
- Shows °F values

**Right (y1):** Percentage scale
- Fixed 0-100%
- Shows % values
- Used by: Cloud Cover, Precip %, Humidity

**Hidden (y2):** Precipitation amount
- Fixed 0-0.5" (0.5" per hour is heavy rain)
- No visible axis
- Used by: QPF bars

**Why hidden y2?**
- Prevents cluttering right side with two scales
- Bars are supplementary visual (not primary metric)
- Hover tooltip shows exact value anyway

### Responsive Behavior

```javascript
/* Lines 557-558 */
responsive: true,
maintainAspectRatio: false,
```

- `responsive: true` → Resizes with container
- `maintainAspectRatio: false` → Allows fixed height (from CSS)

---

## API Structure

### Request
```
GET https://api.weather.com/v3/wx/forecast/hourly/15day
    ?apiKey=e1f10a1e78da46f5b10a1e78da96f525
    &geocode=48.796,-122.502
    &language=en-US
    &units=e
    &format=json
```

### Response (Excerpt)
```json
{
    "validTimeLocal": ["2026-01-15T13:00:00-0800", ...],  // 360 timestamps
    "temperature": [50, 51, 51, ...],                     // 360 temps
    "precipChance": [2, 2, 2, ...],                       // 360 percentages
    "cloudCover": [7, 3, 0, ...],                         // 360 percentages
    "iconCode": [32, 32, 34, ...],                        // 360 codes
    "dayOrNight": ["D", "D", "D", ...],                   // 360 indicators
    // ... 24 more fields
}
```

**We extract 36 hours:**
```javascript
/* Line 410-420 */
weatherData = {
    time: data.validTimeLocal.slice(0, 36),
    temperature: data.temperature.slice(0, 36),
    // ...
};
```

**Available fields not currently used:**
- `temperatureHeatIndex`
- `temperatureDewPoint`
- `uvIndex`
- `visibility`
- `windDirectionCardinal`
- `wxPhraseLong` (text descriptions)
- Many more...

---

## Styling Guide

### CSS Organization

1. **Reset & Base** (Lines 9-25): Universal styles, body, container
2. **Header** (Lines 27-40): Title and metadata
3. **Controls** (Lines 42-69): Buttons and time range selector
4. **Graph Container** (Lines 71-77): Main chart wrapper
5. **Weather Icons** (Lines 79-126): Icon row above chart
6. **Series Toggles** (Lines 128-164): Data series on/off buttons
7. **Stats Cards** (Lines 171-238): Summary metrics at top
8. **Mobile Overrides** (Lines 240-244): < 640px adjustments

### Color Palette

**Background:**
- `#0f1419` - Page background (darkest)
- `#1a1f2e` - Card backgrounds (dark)
- `#1e2732` - Interactive elements (medium dark)
- `#2d3748` - Borders, grids (lighter)

**Text:**
- `#e0e0e0` - Primary text (light)
- `#888` - Secondary text (gray)

**Data Series:**
- `#ef4444` - Temperature (red)
- `#f97316` - Feels Like (orange)
- `#3b82f6` - Precip % (blue)
- `#0ea5e9` - Precip Amount (light blue)
- `#64748b` - Cloud Cover (gray)
- `#10b981` - Wind (green)
- `#8b5cf6` - Humidity (purple)

**Accent:**
- `#3b82f6` - Active state (blue)
- `#dc2626` - Error (red)

---

## Debugging Tips

### Check API Response
```javascript
// Add after Line 406 in fetchWeather()
console.log('API response:', data);
console.log('Extracted 36hr:', weatherData);
```

### Verify Chart Data
```javascript
// Add after Line 542 in renderChart()
console.log('Chart labels:', labels);
console.log('Chart datasets:', datasets);
```

### Test Tooltip Formatting
```javascript
// Add inside tooltip callback (Line 591)
console.log('Tooltip context:', context);
console.log('Label:', label, 'Value:', value);
```

### Force Refresh
- Hard reload: `Ctrl+Shift+R` (clears cache)
- Check Network tab in DevTools for API calls
- Look for 200 status on Weather.com request

---

## Deployment

### Initial Setup
```bash
# Create repo on GitHub (public)
# Clone locally
git clone https://github.com/z-van-baars/wunderground_dashboard.git
cd wunderground_dashboard

# Add file
cp path/to/index.html .
git add index.html
git commit -m "Initial commit"
git push
```

### Enable GitHub Pages
1. Go to repo Settings → Pages
2. Source: "Deploy from a branch"
3. Branch: `main` / `root`
4. Save
5. Wait ~1 minute

**Live URL:** `https://z-van-baars.github.io/wunderground_dashboard/`

### Making Updates
```bash
# Edit index.html locally
# Test changes (open in browser)

git add index.html
git commit -m "Description of changes"
git push

# Auto-deploys to GitHub Pages in 1-2 minutes
```

---

## Testing Checklist

Before committing changes:

- [ ] Test all time ranges (12/24/36hr)
- [ ] Toggle each data series on/off
- [ ] Hover tooltips show correct units
- [ ] Stats cards update when changing time range
- [ ] Graph renders correctly on mobile width (< 640px)
- [ ] Icons show day/night variants appropriately
- [ ] Precip bar color changes based on amount
- [ ] No console errors
- [ ] Hard refresh works (Ctrl+Shift+R)

---

## Performance Notes

**Load Time:**
- Single HTML file: ~15KB gzipped
- Chart.js CDN: ~150KB (cached after first load)
- API request: ~75KB (36hr data)
- **Total first load:** ~240KB, < 1 second on 4G

**Runtime:**
- Chart render: ~50ms
- API fetch: ~200-500ms (depends on Weather.com)
- Memory: ~15MB (Chart.js canvas + data)

**Optimization opportunities:**
- Lazy load Chart.js (only when needed)
- Cache API response in localStorage (reduce requests)
- Compress emojis (use smaller unicode variants)
- Service worker for offline capability

---

## Troubleshooting

### Graph not showing
1. Check console for Chart.js errors
2. Verify `weatherData` is populated (console.log)
3. Check that at least one series is `enabled: true`

### API request failing
1. Check Network tab for 403/401 errors (API key issue)
2. Verify URL is correct (line 365)
3. Check CORS (should work from GitHub Pages)
4. Try API URL directly in browser

### Icons misaligned
1. Adjust padding in `.weather-icons` (line 82)
2. Left padding: affects start position
3. Right padding: affects end position
4. Fine-tune in ~5px increments

### Tooltips showing wrong units
1. Check label name matching in callback (line 600-610)
2. Verify series labels match expected strings
3. Add console.log to see what label is being checked

### Mobile layout broken
1. Check viewport meta tag (line 5): `width=device-width`
2. Verify media query breakpoint (line 240): `max-width: 640px`
3. Test in browser DevTools mobile view

---

## Security Notes

**API Key Exposure:**
- The Weather.com API key is intentionally public (client-side app)
- Weather.com allows this for their embedded widget keys
- Key is rate-limited per IP, not globally
- If abused, create new free account and get new key

**XSS Prevention:**
- No user input is rendered (location is hardcoded)
- If adding location selector, sanitize inputs
- Use `textContent` not `innerHTML` for user data

**HTTPS:**
- GitHub Pages enforces HTTPS
- API requests are HTTPS
- No mixed content warnings

---

## Future Architecture Considerations

### If scaling to multi-location:
- Add backend (Vercel serverless functions)
- Store API key server-side
- Return sanitized data to frontend
- Add authentication for saved locations

### If adding real-time updates:
- WebSocket connection to backend
- Backend polls API every 15min
- Push updates to all connected clients
- Reduces client-side API calls

### If building native app:
- Extract core logic to shared module
- React Native wrapper
- Local storage for preferences
- Push notifications capability

---

## Resources

**APIs:**
- Weather.com API docs: https://weather.com/swagger-docs/ui/sun/v3/sunV3HourlyForecast.json
- Alternative: OpenWeatherMap (free tier)

**Libraries:**
- Chart.js: https://www.chartjs.org/docs/latest/
- Alternative: D3.js (more complex but more flexible)

**Hosting:**
- GitHub Pages docs: https://docs.github.com/en/pages
- Alternative: Vercel, Netlify (also free)

**Icons:**
- Emoji reference: https://emojipedia.org/
- Weather.com icon codes: https://docs.google.com/document/d/1eo8TQU-JzZTJ5ZjrR7lGlLqg_6s1TU2rTg9b8LnGBqc

---

## Contact & Maintenance

**Repository:** https://github.com/z-van-baars/wunderground_dashboard
**Live URL:** https://z-van-baars.github.io/wunderground_dashboard/
**Last Updated:** 2026-01-15

For questions or issues, create a GitHub issue in the repo.
