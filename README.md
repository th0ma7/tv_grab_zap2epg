# tv_grab_zap2epg - tvheadend XMLTV Grabber
North America (zap2epg using tvlistings.gracenote.com)

`tv_grab_zap2epg` will generate an `xmltv.xml` file for Canada/USA TV lineups by fetching channel list from tvheadend and downloading relevant entries from tvlistings.gracenote.com.

zap2epg was originally designed to be easily setup in Kodi for use as a grabber for tvheadend. This is a fork of the original code from edit4ever Python3 branch of script.module.zap2epg based on my PR https://github.com/edit4ever/script.module.zap2epg/pull/37 (much thanks for your great original work @edit4ever !!!)

## Key Features

- **Intelligent Channel Filtering**: Automatically fetches your channel list from TVH to reduce data downloaded and speed up the grab
- **Extended Program Details**: Optional downloading of extra detail information for programs with intelligent description enhancement
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
- Note: Generates additional HTTP requests per unique series

### Enhanced Description Format
When `xdesc=true`, descriptions are intelligently formatted with:
- Extended series descriptions when available
- Season/Episode information (S01E05)
- Original air dates
- Content ratings
- Program flags (NEW, LIVE, PREMIERE, etc.)

Example: `"A documentary about gaming history. • (2023) | S01E02 | Premiered: 2023-03-15 | NEW"`

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
├── conf/
└── log/
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
- WAF block detection and handling
- Channel matching details
- Description enhancement statistics
- Configuration processing details

### Cache Management
- Automatic cleanup of old guide data
- Intelligent cache reuse for extended details
- Manual cache clearing: `tv_grab_zap2epg --clear-cache`

### Error Handling
- Automatic retry with exponential backoff
- WAF protection with adaptive delays
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
- Verify internet connectivity for extended downloads

### Debug Logging
```bash
# Enable detailed logging
tv_grab_zap2epg --debug --days 1

# Check log file location (varies by system)
# Synology DSM7: /var/packages/tvheadend/var/epggrab/zap2epg/log/zap2epg.log
# Docker: ~/zap2epg/log/zap2epg.log
```

## Version History

### Version 4.0
- Intelligent TVheadend channel filtering (fixed)
- Enhanced description system with intelligent formatting
- Automated configuration management and cleanup
- Multi-platform auto-detection
- Optimized downloads with WAF protection
- Comprehensive debug logging and statistics
- Improved error handling and resilience
