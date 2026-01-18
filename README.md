# 36-Hour Weather Dashboard

A mobile-optimized, interactive weather forecast dashboard with location switching. Built with vanilla JavaScript and Chart.js, deployed on GitHub Pages.

**🌐 Live Site:** https://z-van-baars.github.io/wunderground_dashboard/

---

## Features

- **Location switching**: Enter any city to see its forecast, with shareable URLs and localStorage persistence
- **Interactive time ranges**: 12, 24, or 36 hour views
- **Multiple data series**: Temperature, feels-like, precipitation (chance & amount), cloud cover, wind, humidity
- **Enhanced visualizations**:
  - Shaded cloud cover area
  - Hourly weather icons with day/night variants (🌙 vs ☀️)
  - Total precipitation tracker with visual scale
  - Fixed 0-100% scale for instant interpretation
- **Mobile-first design**: Optimized for quick weather checks on phones
- **Auto-refresh**: Updates every 30 minutes

---

## Quick Start

### View Live
Just visit: **https://z-van-baars.github.io/wunderground_dashboard/**

Default location is Seattle, WA - use the location input to change to any city.

### Change Location
1. Enter a city name (e.g., "Portland, OR" or "Denver, CO")
2. Click "Change Location" or press Enter
3. Your choice is saved and the URL updates for sharing

### Share a Specific Location
After changing location, copy the URL - it includes parameters like:
```
https://z-van-baars.github.io/wunderground_dashboard/?lat=45.523&lon=-122.676&name=Portland
```
Anyone opening this link will see Portland's weather.

### Embed in Notion
1. In Notion, type `/embed`
2. Paste: `https://z-van-baars.github.io/wunderground_dashboard/` (or URL with specific location)
3. Resize to fill page width

### Add to Mobile Home Screen
1. Open the URL on your phone
2. iOS: Share → Add to Home Screen
3. Android: Menu → Add to Home Screen

---

## Screenshots

**Desktop View:**
- Wide graph with all data series
- Toggleable metrics for custom views
- Summary stats at top

**Mobile View:**
- Taller graph optimized for portrait
- Compact axis labels
- Touch-friendly toggles

---

## Documentation

- **[Implementation Guide](implementation_guide.md)** - Technical details, modification recipes, debugging tips
- **[Postmortem](postmortem.md)** - Project retrospective, learnings, future enhancements

---

## Tech Stack

- **Frontend**: Vanilla JavaScript (no framework)
- **Charts**: Chart.js 4.4.1
- **Data**: Weather.com API (hourly forecasts)
- **Hosting**: GitHub Pages (auto-deploy on push)
- **Architecture**: Single HTML file (no build process)

---

## Local Development

```bash
# Clone the repo
git clone https://github.com/z-van-baars/wunderground_dashboard.git
cd wunderground_dashboard

# Open in browser
open index.html  # macOS
start index.html # Windows
xdg-open index.html # Linux

# Make changes to index.html
# Refresh browser to see updates

# Deploy
git add index.html
git commit -m "Description of changes"
git push
# Live site updates in 1-2 minutes
```

---

## Customization

### Change Default Location
The default location (shown when no URL params or localStorage) can be changed in code:
```javascript
// Line 323 in index.html
let LOCATION = { lat: 47.606, lon: -122.332 }; // Seattle - change to your lat/lon
let currentLocationName = 'Seattle, WA'; // Update display name too
```

Or just use the UI - your browser will remember your choice via localStorage.

### Add/Remove Data Series
```javascript
// Lines 282-341 in index.html
const SERIES_CONFIG = {
    yourSeries: {
        label: 'Your Label',
        color: '#hexcode',
        yAxisID: 'y', // or 'y1' for percentage
        type: 'line', // or 'bar'
        enabled: true // show by default
    }
}
```

See [Implementation Guide](implementation_guide.md) for more recipes.

---

## Contributing

This is a personal project, but feel free to fork and customize for your own use!

**Ideas for future enhancements:**
- Multiple saved locations (favorites list)
- 7-day daily forecast summary
- Dark/light mode toggle
- Severe weather alerts
- Historical comparisons
- PWA with offline caching

See [Postmortem](postmortem.md) for full list of future enhancements.

---

## License

MIT License - use freely, attribution appreciated but not required.

---

## Credits

**Built by:** Zac Van Baars
**Data source:** Wunderground.com API
**Chart library:** Chart.js
**Hosting:** GitHub Pages

---

## Questions?

Create an issue in this repo or check the [Implementation Guide](implementation_guide.md) for troubleshooting tips.

---

*Last updated: 2026-01-15*
