# tv_grab_zap2epg - tvheadend XMLTV Grabber
North America (zap2epg using tvlistings.zap2it.com)

`tv_grab_zap2epg` will generate an `xmltv.xml` file for Canada/USA TV lineups by fetching channel list from tvheadend and downloading relevant entries from tvlistings.zap2it.com.

zap2epg was originally designed to be easily setup in Kodi for use as a grabber for tvheadend.  This is a fork of the original code from edit4ever Python3 branch of script.module.zap2epg based on my PR https://github.com/edit4ever/script.module.zap2epg/pull/37 (much thanks for your great original work @edit4ever !!!)

It includes the ability to automatically fetch your channel list from TVH to reduce the amount of data downloaded and speed up the grab. It has an option for downloading extra detail information for programs. (Note: this option will generate an extra http request per episode) It also has an option to append the extra detail information to the description (plot) field, which makes displaying this information in the Kodi EPG easier on many skins.

_Note that zap2epg is a proof of concept and is for personal experimentation only. It is not meant to be used in a commercial product and its use is your own responsibiility._

## `tv_grab_zap2epg` capabilities
zap2epg TV guide grabber script provides `baseline` capabilities (ref: http://wiki.xmltv.org/index.php/XmltvCapabilities):
- `--quiet`: Suppress all progress information. When --quiet is used, the grabber shall only print error-messages to stderr.
- `--output FILENAME`: Redirect the xmltv output to the specified file. Otherwise output goes to stdout along with a copy under `epggrab/cache/xmltv.xml`.
- `--days X`: Supply data for X days, limited to 14.
- `--offset X`: Start with data for day today plus X days. The default is 0, today; 1 means start from tomorrow, etc.
- `--config-file FILENAME`: The grabber shall read all configuration data from the specified file.  Otherwise uses default under `epggrab/conf/zap2epg.xml`
It also provide the following "extra" capabilities:
- `--zip` or `--postal` or `--code`: Allow can be used to pass US Zip or Canadian Postal code to be used by the grabber.

## Configuration options (`zap2epg.xml`)
- `<setting id="zipcode">92101</setting>`: US Zip or Canada postal code
- `<setting id="lineupcode">lineupId</setting>`: 
- `<setting id="lineup">Local Over the Air Broadcast</setting>`: 
- `<setting id="device">-</setting>`: 
- `<setting id="days">1</setting>`: Number of TV guide days
- `<setting id="redays">1</setting>`: 
- `<setting id="slist"></setting>`: 
- `<setting id="stitle">false</setting>`: 
- `<setting id="xdetails">true</setting>`: Download extra Movie or Serie details
- `<setting id="xdesc">true</setting>`: Provide extra details to default TV show description
- `<setting id="epgenre">3</setting>`: 
- `<setting id="epicon">1</setting>`: 
- `<setting id="tvhoff">true</setting>`: true=fetch channel list from TVH server, false=do not fetch channel list
- `<setting id="usern"></setting>`: Username to access TVH server, anonymous if both `usern` + `passw` are empty
- `<setting id="passw"></setting>`: Password to access TVH server
- `<setting id="tvhurl">127.0.0.1</setting>`: IP address to TVH server
- `<setting id="tvhport">9981</setting>`: Port of TVH server
- `<setting id="tvhmatch">true</setting>`: 
- `<setting id="chmatch">true</setting>`: 

## TVheadEnd (TVH) auto-channel fetching
This feature is enabled by default using anonymous access.  You must have a `*` user defined in TVH with minimally:
* Change parameters: `Rights`
* Allowed networks: `127.0.0.1` or `0.0.0.0;::/0`
 using the following configuration:
* `<setting id="tvhoff">true</setting>`
* `<setting id="usern"></setting>`
* `<setting id="passw"></setting>`
* `<setting id="tvhurl">127.0.0.1</setting>`
* `<setting id="tvhport">9981</setting>`
* `<setting id="tvhmatch">true</setting>`
* `<setting id="chmatch">true</setting>`
It can be disabled by setting `<setting id="tvhoff">false</setting>`.  It can also be set to using another user account by filling in the `usern` and `passwd` fields.

## INSTALLATION: Synology DSM6/DSM7 TVH server:
Already included in the SynoCommunity tvheadend package for DSM6-7 since  v4.3.20210612-29
* https://synocommunity.com/package/tvheadend

**DSM7** file structure:
```
/var/packages/tvheadend/target/bin
└── tv_grab_zap2epg
/var/packages/tvheadend/var/epggrab
├── cache
│   ├── *.json
│   └── xmltv.xml
├── conf
│   └── zap2epg.xml
└── log 
    └── zap2epg.log
```
**DSM6** file structure:
```
/var/packages/tvheadend/target/bin
└── tv_grab_zap2epg
/var/packages/tvheadend/target/var/epggrab
├── cache
│   ├── *.json
│   └── xmltv.xml
├── conf
│   └── zap2epg.xml
└── log 
    └── zap2epg.log
```

### Manual Installation
1. Install `tv_grab_zap2epg` script addon in `/usr/local/bin` or `/var/packages/tvheadend/target/bin`
2. Install `zap2epg.xml` configuration file under tvheadend `epggrab/conf` directory such as `..var/epggrab/conf`
3. Manually adjust `zap2epg.xml` configuration file as needed or use capabilities (see description below)

### Testing
In order to test `tv_grab_zap2epg` EPG grabber under Synology DSM:
```
$ sudo su -s /bin/bash sc-tvheadend -c '~/bin/tv_grab_zap2epg --capabilities'
baseline
$ sudo su -s /bin/bash sc-tvheadend -c '~/bin/tv_grab_zap2epg --days 1 --postal J3B1M4 --quiet'
$ ls -la ~sc-tvheadend/var/epggrab/cache/xmltv.xml
-rw-rw-rw- 1 sc-tvheadend tvheadend 16320858 Jun 13 07:43 /var/packages/tvheadend/target/var/epggrab/cache/xmltv.xml
```

## INSTALLATION: Docker tvheadend setup:
Installation is somewhat different where it uses the `hts` user account to handle the directory structure such as:
```
/usr/bin
└── tv_grab_zap2epg
/home/hts/epggrab
├── cache
│   ├── *.json
│   └── xmltv.xml
├── conf
│   └── zap2epg.xml
└── log 
    └── zap2epg.log
```

### Manual Installation
Create the directory structure and adjust permissions:
```
# mkdir -p /home/hts/zap2epg/conf
# mkdir -p /home/hts/zap2epg/log
# mkdir -p /home/hts/zap2epg/cache
# chown -R hts:hts /home/hts/zap2epg
# chmod -R 0755 /home/hts/zap2epg
```
Copy the configuration and adjust permissions:
```
# cp zap2epg.xml /home/hts/zap2epg/conf
# chown hts:hts /home/hts/zap2epg/conf/zap2epg.xml
# chmod 644 /home/hts/zap2epg/conf/zap2epg.xml
```
Copy the script to /usr/bin and adjust permissions:
```
# tv_grab_zap2epg /usr/local/bin
# chmod 755 /usr/local/bin/tv_grab_zap2epg
```
