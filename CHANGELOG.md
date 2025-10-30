# Changelog - Atrium Gateway

All notable changes to the Atrium Gateway will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.6] - 2025-10-29

### 🎨 Added
- Global navigation header across all pages with pill-styled buttons
- Alert column on Sensors List page with red/OK indicators
- Counter metric logging and display on sensor dashboards
- Grouped metrics display (Core Metrics + Sensor Metrics sections)
- Smart color coding: green for healthy, red for unhealthy/alerting metrics
- Clickable metric cards that scroll to corresponding charts
- Mobile-responsive navigation (2-row layout on screens ≤768px)
- Password hash validation in deployment scripts

### 🔧 Changed
- Unified all navigation in global header (removed redundant page-specific buttons)
- Enhanced date/time display (larger, white monospace font)
- All input fields now have consistent dark theme styling (number, email, password, time, textarea)
- Removed panel gradients for cleaner appearance
- Removed battery voltage from dashboard display (still logged)
- Removed Online/Offline status indicator (shown via Last Heard color)
- Removed "Live" pill from Sensors List header
- All Cancel/secondary buttons now use consistent gray styling
- Dynamic buttons change from gray to orange when active

### 🐛 Fixed
- Bulk whitelist/blacklist operations (HTTP 400 error) - added JSON payload parsing
- Password authentication on fresh deployments - hash validation prevents silent failures

---

## [1.0.3] - 2025-10-24

### 🔒 Security (CRITICAL)
- Implemented JWT authentication enforcement on all API endpoints
- Created reusable JWT authentication subflow for Node-RED
- Updated React app to include Authorization headers in all requests
- Public endpoints: Only login and CORS preflight
- Fixed payload restoration in JWT subflow for POST endpoints

### nginx Architecture
- Migrated from Node-RED to nginx for serving React app
- React app now served on port 80 (production standard)
- API requests proxied through nginx to Node-RED (port 1880)
- Robustel console moved to port 8080
- Node-RED admin restricted to local access only (port 1880/admin)

### Rate Limiting
- Login endpoint: 5 requests/minute (prevents brute force attacks)
- General API: 100 requests/second (prevents abuse)
- Returns HTTP 429 when exceeded
- Separate limits for login vs general API calls

### SSH Key Authentication
- Added automatic SSH key setup to Mac/Linux deployment script
- Passwordless deployments after first run
- Matches existing Windows implementation

## 🎨 User Interface

### Date/Time Display
- Added live date/time banner to all pages
- Updates every second
- Shows: "Mon, Oct 28, 2024, 10:05:32 AM"
- Displays in user's browser timezone

### Sensor List Page
- Added sortable columns: Status, Sensor Type, Sensor Name, Location, Asset
- Case-insensitive alphabetical sorting
- Click column headers to toggle ascending/descending

### Gateway Config Page
- Added "Select All" checkbox for whitelisted devices
- Added timezone configuration dropdown (US + international zones)
- Added data retention period setting (days)
- Improved "Check for Updates" UX (no blocking alerts, auto-dismiss)
- Better error handling with actual backend messages

### Alert System
- Changed "ACTIVE" badge to "ALERT" (red) for clarity
- Added "OK" badge (green) when alert not active
- Applied to both sensor dashboards and alert management page
- Changed reset threshold from "+/-" offset to absolute value entry
  - More intuitive: "Reset at 24" vs "Reset at +/- 1"
- Column header: "Reset (±)" → "Reset At"

### Sensor Dashboard
- Added custom days input (1-365 days)
- Added custom date/time range picker with start/end date/time
- Date fields pre-filled with today's date
- Time fields default to 00:00 and 23:59
- Per-chart CSV export button (📥 CSV) on each chart
- Export includes timestamp, readable datetime, and value

### Alert History
- Implemented complete alert history logging system
- Real-time search/filter by device ID, sensor name, or metric
- Displays triggered and cleared events
- Color-coded badges (red/green)
- Clickable device IDs navigate to dashboards
- Auto-refreshes every 10 seconds

### SQL Query Tool (NEW)
- Complete SQL query interface for database inspection
- 10 built-in quick query buttons
- Save custom queries to localStorage
- Delete saved queries
- Export query results to CSV
- Terminal icon button on main page
- Safety warning banner
- Audit logging of all queries

## ⚙️ Configuration

### Timezone Management
- Configurable display timezone for email alerts
- Dropdown with US and international timezones
- Auto-loads on Node-RED boot
- Email alerts now show correct local time instead of GMT

### Data Retention
- Configurable retention period (default: 100 days)
- Automatic cleanup runs daily at 3 AM
- Deletes old readings, ingest logs, and alert history
- Runs VACUUM to reclaim disk space
- UI configuration in Gateway Config page

### Gateway Name
- Fixed automatic gateway name generation from MAC address
- Format: Gateway-XXXX (last 4 hex digits of MAC)
- Properly saved to database on installation
- No more random names like "Gateway-TIHS"

## 🚀 Deployment & Infrastructure

### Deployment Scripts - All Platforms
- Updated all 4 variants (Mac/Linux + Windows, Internal + Customer)
- Added automatic React app building (runs npm install + build if needed)
- Node.js >= 16 and npm >= 7 version checking
- Added JWT.json subflow to all deployments
- Updated nginx template in all deployment directories
- Fixed PowerShell syntax errors and CRLF line endings (Windows)
- Added remote-access certificate handling to deployments

### nginx Configuration
- nginx-template.conf included in all deployment packages
- Automated backup and deployment in install scripts
- Configuration test before applying
- Automatic rollback on failure
- Updated remote access setup scripts to port 80

### Database Schema
- Added alert_history table for logging all alert events
- Indexes for fast querying (device_id, timestamp, sensor_name)
- Migration script for existing gateways
- Updated all deployment database schemas

### Update System
- Improved subflow update logic (handles name-based replacement)
- Enhanced flow merge to prevent duplicate subflows
- Created v1.0.5 update package with all improvements

## 🐛 Bug Fixes

- Fixed AlertsPage 401 error (missing JWT token)
- Fixed gateway config save 400 error (read from req.body)
- Fixed sensor name prepending with sensor type ID
- Fixed React app API configuration (uses same origin, nginx proxies)
- Improved error messages from backend API
- Fixed deployment script platform version (1.0.5)

---

## [1.0.0] - 2025-10-15

### 🎉 Initial Production Release

#### Added
- **Authentication System**
  - Password-protected interface
  - Session management with 7-day expiration
  - User login/logout functionality
  
- **Sensor Monitoring**
  - Real-time sensor list with auto-refresh (5s)
  - Health status indicators (LED)
  - Online/offline detection based on report intervals
  - Sensor metadata (location, asset, custom names, install date)
  - Report interval configuration
  - Whitelist/blacklist device control
  
- **Dashboard Features**
  - Dynamic sensor dashboards with time-series charts
  - Multiple time ranges (1h, 6h, 24h, 7d, 30d)
  - Zoom and pan controls
  - Auto-refresh (30s)
  - Current value cards with min/max/avg statistics
  
- **Alert System**
  - Visual threshold configuration on charts
  - Draggable threshold lines
  - Hysteresis-based alerting (prevents spam)
  - Email notifications via SMTP
  - Per-alert recipient configuration (multi-recipient support)
  - Offline device alerts
  - Alert management page
  - Active/Inactive status tracking
  
- **Gateway Management**
  - Gateway configuration page
  - Custom gateway naming (auto-generated from MAC)
  - SMTP email configuration
  - New sensor handling (auto-whitelist/blacklist)
  - Device management with bulk operations
  - Gateway health dashboard
  - Storage usage bar (color-coded)
  - CPU, RAM, connectivity monitoring
  
- **Mobile Support**
  - Responsive design
  - Simplified sensor list for mobile (≤1024px)
  - Full-featured desktop view
  - Touch-friendly interface
  
- **Remote Access**
  - Remote access integration
  - Dynamic API base detection
  - Works on any gateway (MAC-agnostic)
  - Secure HTTPS access

#### Technical Features
- SQLite database with optimized schema
- Node-RED API flows
- RESTful API endpoints
- Client-side routing
- Dark theme UI
- Real-time data updates
- Efficient data querying with indexes

---

## Future Releases

### Planned for 1.1.0
- Alert history logging
- Data export (CSV)
- Sensor comparison views
- Custom date range picker
- Multiple user accounts with roles
- Password reset functionality

### Planned for 1.2.0
- SMS notifications
- Webhook integration
- Scheduled reports
- Data retention policies
- Automatic backups
- System diagnostics page

### Planned for 2.0.0
- Cloud integration (AWS/Azure)
- Mobile apps (iOS/Android)
- Advanced analytics
- Machine learning anomaly detection
- Multi-gateway management

---

## Version Format

`MAJOR.MINOR.PATCH`

- **MAJOR:** Breaking changes, major new features
- **MINOR:** New features, backward compatible
- **PATCH:** Bug fixes, minor improvements

---

## Migration Notes

### Upgrading to 1.0.6

**From any previous version:**
1. Automatic update available via Gateway Config
2. No data loss or configuration changes required
3. ~2-3 minutes downtime during update
4. Automatic rollback on failure

**Fresh Installation:**
- Follow [Quick Start Guide](documentation/QUICK_START.md)
- Use automated deployment scripts

---

## Support

For issues, feature requests, or questions:
- Check [User Guide](documentation/USER_GUIDE.md) for usage instructions
- Review [Quick Start Guide](documentation/QUICK_START.md) for setup
- Contact: [Your support contact information]

---

**[Unreleased changes will be listed here]**
