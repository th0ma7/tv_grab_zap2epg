# tv_grab_zap2epg - tvheadend XMLTV Grabber
North America (zap2epg using tvlistings.gracenote.com)

`tv_grab_zap2epg` will generate an `xmltv.xml` file for Canada/USA TV lineups by fetching channel list from tvheadend and downloading relevant entries from tvlistings.gracenote.com.

zap2epg was originally designed to be easily setup in Kodi for use as a grabber for tvheadend. This is a fork of the original code from edit4ever Python3 branch of script.module.zap2epg based on my PR https://github.com/edit4ever/script.module.zap2epg/pull/37 (much thanks for your great original work @edit4ever !!!)

## Key Features

- **Intelligent Channel Filtering**: Automatically fetches your channel list from TVH to reduce data downloaded and speed up the grab
- **Simplified Cache Management**: Smart cache system that preserves existing data and only downloads what's needed
- **Intelligent Guide Refresh**: Refreshes first 48 hours while reusing cached data for later periods
- **Safe Backup System**: Automatic XMLTV backup before generation with smart retention management
- **Extended Program Details**: Optional downloading of extra detail information with optimized cache reuse
- **Smart Configuration Management**: Automatically cleans deprecated settings and manages configuration versions
- **Optimized Downloads**: WAF-protected downloads with adaptive delays and connection reuse
- **Multi-Platform Support**: Intelligent detection and configuration for Raspberry Pi, Synology NAS, and standard Linux systems
- **Enhanced Logging**: Debug mode with detailed statistics and intelligent error reporting

_Note that zap2epg is a proof of concept and is for personal experimentation only. It is not meant to be used in a commercial product and its use is your own responsibiility._

## `tv_grab_zap2epg` capabilities
zap2epg TV guide grabber script provides `baseline` capabilities (ref: http://wiki.xmltv.org/index.php/XmltvCapabilities):
- `--quiet`: Suppress all progress information. When --quiet is used, the grabber shall only print error-messages to stderr.
- `--output FILENAME`: Redirect the xmltv output to the specified file. Otherwise output goes to stdout along with a copy under `cache/xmltv.xml`.
- `--days X`: Supply data for X days, limited to 14.
- `--offset X`: Start with data for day today plus X days. The default is 0, today; 1 means start from tomorrow, etc.
- `--config-file FILENAME`: The grabber shall read all configuration data from the specified file. Otherwise uses default under `conf/zap2epg.xml`
- `--basedir DIRECTORY`: Specify base directory for configuration, cache, and logs. Auto-detected if not specified.
- `--zip` or `--postal` or `--code`: Pass US Zip code or Canadian Postal code to be used by the grabber.
- `--debug`: Enable debug logging with detailed download statistics and verbose output.

## Configuration options (`zap2epg.xml`)

### Required Settings
- `<setting id="zipcode">92101</setting>`: US Zip code (5 digits) or Canada postal code (A1A 1A1 format)

### Basic Settings  
- `<setting id="lineupcode">lineupId</setting>`: Lineup identifier (use 'lineupId' for auto-detection)
- `<setting id="lineup">Local Over the Air Broadcast</setting>`: Human-readable lineup description
- `<setting id="device">-</setting>`: Device identifier (use '-' for default)
- `<setting id="days">1</setting>`: Number of TV guide days to download (1-14)
- `<setting id="redays">1</setting>`: Number of days to keep in cache before cleanup

### Station Filtering
- `<setting id="slist"></setting>`: Comma-separated list of specific station IDs (leave empty to use TVH channel filtering)
- `<setting id="stitle">false</setting>`: Safe filename processing for episode titles (true/false)

### Extended Details
- `<setting id="xdetails">true</setting>`: Download extended program details (series descriptions, cast, etc.)
- `<setting id="xdesc">true</setting>`: Use extended descriptions in XMLTV output (automatically enables xdetails)

### Display Options
- `<setting id="epgenre">3</setting>`: Genre handling mode (0=none, 1=primary only, 2=EIT categories, 3=all genres)
- `<setting id="epicon">1</setting>`: Program icons (0=none, 1=series+episode, 2=episode only)

### TVheadEnd Integration
- `<setting id="tvhoff">true</setting>`: Enable automatic channel list fetching from TVH server
- `<setting id="usern"></setting>`: Username for TVH server (leave empty for anonymous access)
- `<setting id="passw"></setting>`: Password for TVH server
- `<setting id="tvhurl">127.0.0.1</setting>`: IP address of TVH server
- `<setting id="tvhport">9981</setting>`: Port of TVH server (usually 9981)
- `<setting id="tvhmatch">true</setting>`: Use TVH channels as station filter
- `<setting id="chmatch">true</setting>`: Match channel numbers with call signs for sub-channels

## TVheadEnd (TVH) Auto-Channel Filtering

**This is the recommended configuration** and is enabled by default. The script will:

1. Connect to your TVH server to fetch the list of enabled channels
2. Filter the TV guide download to only include those channels
3. Match channel numbers with TVH configuration for proper identification

### TVH User Permissions
You must have a `*` user defined in TVH with minimally:
* **Change parameters**: `Rights`
* **Allowed networks**: `127.0.0.1` or `0.0.0.0;::/0`

### Configuration for TVH Integration
```xml
<setting id="tvhoff">true</setting>         <!-- Enable TVH integration -->
<setting id="usern"></setting>              <!-- Anonymous access -->
<setting id="passw"></setting>              <!-- Anonymous access -->
<setting id="tvhurl">192.168.1.100</setting> <!-- Your TVH server IP -->
<setting id="tvhport">9981</setting>        <!-- TVH web port -->
<setting id="tvhmatch">true</setting>       <!-- Use TVH channels as filter -->
<setting id="chmatch">true</setting>        <!-- Match sub-channels -->
```

### Example Log Output with TVH Filtering
```
2025/07/30 11:33:14 32 Tvheadend channels found...
2025/07/30 11:33:14 Tvheadend channel filtering enabled: 32 channels will be used as filter
2025/07/30 11:33:25 32 Stations and 2156 Episodes written to xmltv.xml file.
```

To disable TVH integration, set: `<setting id="tvhoff">false</setting>`

## Extended Program Details

The script offers two levels of program information:

### Basic Mode (`xdetails=false`, `xdesc=false`)
- Uses only standard TV guide descriptions
- Fastest download with minimal API requests

### Extended Mode (`xdetails=true` and/or `xdesc=true`)
- Downloads additional series information, cast details, and enhanced descriptions
- `xdetails=true`: Downloads extended data but uses basic descriptions
- `xdesc=true`: Uses enhanced descriptions with intelligent formatting (automatically enables xdetails)
- **Intelligent Cache Reuse**: Only downloads details for new series, reuses existing cached data
- Note: First run may require downloading 1000+ series details, subsequent runs are much faster

### Enhanced Description Format
When `xdesc=true`, descriptions are intelligently formatted with:
- Extended series descriptions when available
- Season/Episode information (S01E05)
- Original air dates
- Content ratings
- Program flags (NEW, LIVE, PREMIERE, etc.)

Example: `"A documentary about gaming history. • (2023) | S01E02 | Premiered: 2023-03-15 | NEW"`

### Extended Details Cache Efficiency
The script intelligently manages series details cache:
```
Extended details processing completed:
  Total unique series: 1137
  Downloads attempted: 1           ← Only new series
  Successful downloads: 1
  Unique series from cache: 1136   ← Reused existing
  Cache efficiency: 99.9% (1136/1137 unique series reused)
```

## Simplified Cache Management System

The script features an intelligent cache management system that optimizes performance and storage:

### Guide Cache (3-hour blocks)
- **Smart Refresh Window**: Refreshes first 48 hours of guide data on each run
- **Cache Reuse**: Reuses cached blocks outside refresh window to minimize downloads
- **Safe Updates**: Creates backup before refreshing, restores if download fails
- **Automatic Cleanup**: Removes blocks outside current guide period

### XMLTV Backups
- **Always Backup**: Creates timestamped backup before generating new XMLTV
- **Smart Retention**: Keeps backups for guide duration (days parameter)
- **Automatic Cleanup**: Removes old backups beyond retention period

### Series Details Cache
- **Preserve Existing**: Never deletes cached series details unless no longer needed
- **Intelligent Cleanup**: Only removes details for series not in current guide
- **Optimal Reuse**: Dramatically reduces download time on subsequent runs

### Cache Statistics Example
```
=== Cache Cleanup ===
Guide cache cleanup: 12 removed, 24 kept, 2 invalid blocks removed
Show cache cleanup: 45 removed, 1092 kept
XMLTV cleanup: 3 old backups removed (retention: 7 days)

Guide download completed:
  Blocks: 56 total (8 downloaded, 48 cached, 0 failed)
  Cache efficiency: 85.7% reused
  Success rate: 100.0%
```

## System Auto-Detection

The script automatically detects your system type and configures appropriate directories:

### Raspberry Pi (Kodi/LibreELEC)
```
~/script.module.zap2epg/epggrab/  (if exists)
~/zap2epg/                       (fallback)
```

### Synology NAS
**DSM7:**
```
/var/packages/tvheadend/var/epggrab/zap2epg/
├── cache/
│   ├── xmltv.xml
│   ├── xmltv.xml.20250731_143022 (backup)
│   ├── 2025073100.json.gz (guide blocks)
│   └── SH01234567.json (series details)
├── conf/
│   └── zap2epg.xml
└── log/
    └── zap2epg.log
```

**DSM6:**
```
/var/packages/tvheadend/target/var/epggrab/zap2epg/
├── cache/
├── conf/
└── log/
```

### Standard Linux/Docker
```
~/zap2epg/
├── cache/
├── conf/
└── log/
```

## Configuration Management

The script automatically:
- **Upgrades configuration** from version 2 to version 3
- **Removes deprecated settings**: `useragent`, `desc01-desc20` (old custom formatting)
- **Creates backups** only when modifications are needed
- **Preserves your settings** while cleaning obsolete parameters

### Migration Example
Your old configuration with deprecated settings:
```xml
<settings version="2">
  <setting id="zipcode">92101</setting>
  <setting id="useragent">Mozilla/5.0...</setting>  <!-- Removed -->
  <setting id="desc01">10</setting>                  <!-- Removed -->
  <!-- ... other desc02-desc20 removed ... -->
</settings>
```

Becomes:
```xml
<settings version="3">
  <setting id="zipcode">92101</setting>
  <!-- Clean, modern configuration -->
</settings>
```

## INSTALLATION

### Synology DSM6/DSM7 TVH server
Already included in the SynoCommunity tvheadend package for DSM6-7 since v4.3.20210612-29
* https://synocommunity.com/package/tvheadend

### Manual Installation (All Systems)
1. Copy `tv_grab_zap2epg` script to appropriate location:
   - Synology: `/var/packages/tvheadend/target/bin/`
   - Docker: `/usr/local/bin/`
   - Standard Linux: `/usr/local/bin/`

2. Make executable: `chmod 755 tv_grab_zap2epg`

3. Create configuration file or let the script auto-generate it on first run

## Testing & Usage

### Basic Testing
```bash
# Check capabilities
tv_grab_zap2epg --capabilities

# Test with debug logging
tv_grab_zap2epg --days 1 --postal J3B1M4 --debug

# Quiet operation
tv_grab_zap2epg --days 1 --postal J3B1M4 --quiet
```

### Synology Testing
```bash
sudo su -s /bin/bash sc-tvheadend -c '~/bin/tv_grab_zap2epg --capabilities'
sudo su -s /bin/bash sc-tvheadend -c '~/bin/tv_grab_zap2epg --days 1 --postal J3B1M4 --debug'
```

### Docker Testing
```bash
# Inside container
su - hts -c "tv_grab_zap2epg --days 1 --zip 90210 --debug"
```

## Advanced Features

### Debug Mode
Enable with `--debug` for detailed information:
- Download statistics and timing
- Cache efficiency metrics
- WAF block detection and handling
- Channel matching details
- Description enhancement statistics
- Configuration processing details

### Performance Optimization
- **First Run**: May take longer as series details are downloaded and cached
- **Subsequent Runs**: Much faster due to intelligent cache reuse
- **Refresh Strategy**: Only refreshes recent guide data (48 hours) while preserving older cached blocks
- **Network Efficiency**: Minimizes API requests through smart caching

### Error Handling
- Automatic retry with exponential backoff
- WAF protection with adaptive delays
- Safe backup/restore for failed downloads
- Graceful degradation when TVH is unavailable
- Comprehensive error logging

## Troubleshooting

### Common Issues

**No channels found:**
- Check TVH server accessibility (`tvhurl` and `tvhport`)
- Verify TVH user permissions
- Enable debug mode to see connection details

**Too many/few channels:**
- Review `tvhmatch` and `chmatch` settings
- Check TVH channel configuration
- Use debug mode to see channel matching logic

**Missing descriptions:**
- Enable extended details with `xdesc=true`
- Check download statistics in logs
- First run will download many series details (normal)
- Verify internet connectivity for extended downloads

**Slow performance:**
- First run with extended details takes longer (normal)
- Check cache efficiency in logs
- Subsequent runs should be much faster
- Consider reducing `days` parameter for testing

### Performance Tips
1. **Extended Details**: First run downloads 1000+ series details but subsequent runs reuse cache
2. **Cache Management**: Script automatically maintains optimal cache size
3. **Network Issues**: Script handles temporary network problems gracefully
4. **Debug Mode**: Use `--debug` to monitor cache efficiency and download statistics

### Debug Logging
```bash
# Enable detailed logging
tv_grab_zap2epg --debug --days 1

# Check log file location (varies by system)
# Synology DSM7: /var/packages/tvheadend/var/epggrab/zap2epg/log/zap2epg.log
# Docker: ~/zap2epg/log/zap2epg.log

# Monitor real-time with tail
tail -f ~/zap2epg/log/zap2epg.log
```

### Example Debug Output
```
=== Cache Cleanup ===
Initial cache cleanup completed (show cache will be cleaned after parsing episodes)

Starting optimized guide download
  Refresh window: first 48 hours will be re-downloaded
  Guide duration: 56 blocks (168 hours)

Refreshing block: 2025-07-31 21:00-00:00 [REFRESH]
  Refresh success: 2025073121.json.gz (124756 bytes)
Using cached: 2025-08-01 00:00-03:00
...

Guide download completed:
  Blocks: 56 total (8 downloaded, 48 cached, 0 failed)
  Cache efficiency: 85.7% reused
  Success rate: 100.0%

=== Show Cache Cleanup ===
Found 1137 active series in current schedule
Show cache cleanup: 45 removed, 1092 kept

Extended details processing completed:
  Total unique series: 1137
  Downloads attempted: 45
  Successful downloads: 45
  Unique series from cache: 1092
  Cache efficiency: 96.0% (1092/1137 unique series reused)
```

## Version History

### Version 4.0
- **Simplified Cache Management**: Intelligent cache system that preserves existing data
- **Smart Guide Refresh**: Refreshes first 48 hours while reusing cached data
- **Automatic XMLTV Backup**: Safe backup system with smart retention
- **Optimized Series Details**: Dramatically improved cache reuse for extended details
- **Enhanced Statistics**: Detailed cache efficiency and performance metrics
- **Intelligent TVheadend channel filtering** (fixed)
- **Enhanced description system** with intelligent formatting
- **Automated configuration management** and cleanup
- **Multi-platform auto-detection**
- **Optimized downloads** with WAF protection
- **Comprehensive debug logging** and statistics
- **Improved error handling** and resilience
