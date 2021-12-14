# Daily Job Scheduler (DJS)



- [DESCRIPTION](#description)
- [LICENSE](#license)
- [DISCLAIMER](#disclaimer)
- [WARNING](#warning)
- [DONATIONS](#donations)
- [PREREQUISITES](#prerequisites)
- [CONFIGURATION (/data/adb/vr25/djs-data/config.txt)](#configuration-dataadbvr25djs-dataconfigtxt)
- [USAGE](#usage)
  - [Terminal Commands](#terminal-commands)
- [NOTES/TIPS FOR FRONT-END DEVELOPERS](#notestips-for-front-end-developers)
  - [Basics](#basics)
  - [Initializing DJS](#initializing-djs)
- [FREQUENTLY ASKED QUESTIONS (FAQ)](#frequently-asked-questions-faq)
- [NOOB FRIENDLY](#noob-friendly)
  - [Prequisites](#prequisites)
  - [Steps](#steps)
  - [Extras](#extras)
- [LINKS](#links)
- [LATEST CHANGES](#latest-changes)


---
## DESCRIPTION

DJS runs scripts and/or commands on boot or at set HH:MM (e.g, 23:59).
Any root solution is supported.
The installation is always "systemless", whether or not the system is rooted with Magisk.


---
## LICENSE

Copyright (C) 2019-2021, VR25

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see <https://www.gnu.org/licenses/>.


---
## DISCLAIMER

Always read/reread this reference prior to installing/upgrading this software.

While no cats have been harmed, the author assumes no responsibility for anything that might break due to the use/misuse of it.

To prevent fraud, do NOT mirror any link associated with this project; do NOT share builds (tarballs/zips)! Share official links instead.


---
## WARNING

The author assumes no responsibility under anything that might break due to the use/misuse of this software.
By choosing to use/misuse it, you agree to do so at your own risk!


---
## DONATIONS

Please, support the project with donations ([links](#links) at the bottom).
As the project gets bigger and more popular, the need for coffee goes up as well.


---
## PREREQUISITES

- Android or Android based OS
- Any root solution (e.g., [Magisk](https://github.com/topjohnwu/Magisk/))
- [Busybox\*](https://github.com/search?o=desc&q=busybox+android&s=updated&type=Repositories/) (only if not rooted with Magisk)
- Non-Magisk users can enable djs auto-start by running /data/adb/vr25/djs/service.sh, a copy of, or a link to it - with init.d or an app that emulates it.
- Terminal emulator
- Text editor (optional)

\* A busybox binary can simply be placed in /data/adb/vr25/bin/.
Permissions (0700) are set automatically, as needed.
Precedence: /data/adb/vr25/bin/busybox > Magisk's busybox > system's busybox

Other executables or static binaries can also be placed in /data/adb/vr25/bin/ (with proper permissions) instead of being installed system-wide.


---
## CONFIGURATION (/data/adb/vr25/djs-data/config.txt)

```
# This is a comment line.
# All lines that do not match a schedule instruction are ignored, regardless of the '#'.

# This is used to determine whether this file should be patched. Do NOT modify!
versionCode=201908180

# Schedule Examples

# Run on boot
#boot touch /data/I-was-born-on-boot; : --delete
# ": --delete" is optional; it means "delete the schedule after execution" - effectively turning it into a one-time boot schedule.

# Apply Advanced Charging Controller night settings at 22:00
#2200 acc pc=45 rc=44 mcc=500000 mcv=3920

# Restore regular ACC settings at 6:00 (morning)
#0600 acc pc=75 rc=70 mcc= mcv=; : --boot
# ": --boot" is optional; it makes this schedule run on boot as well (e.g., as part of a fail-safe plan).
```

---
## USAGE


If you feel uncomfortable with the command line, use a `text editor` to modify `/data/adb/vr25/djs-data/config.txt`.
Changes to this file take effect almost instantly, and without a [daemon](https://en.wikipedia.org/wiki/Daemon_(computing)) restart.


### Terminal Commands

```
Config Management

Usage: djsc|djs-config OPTION ARGS

-a|--append 'LINE'
  e.g., djsc -a 2200 reboot -p

-d|--delete ['regex'] (deletes all matching lines)
  e.g., djsc --delete 2200

-e|--edit [cmd] (fallback cmd: nano -l|vim|vi)
  e.g., djs-config --edit vim

-l|--list ['regex'] (fallback regex: ".", matches all lines)
  e.g., djsc -l '^boot'

-L|--log [cmd] (fallback cmd: tail -F)
  e.g., djsc -L cat > /sdcard/djsd.log


Daemon Management

Start/restart: djsd|djs-daemon
Stop: djs.|djs-stop
Status: djs,|djs-status


Print Version Code (integer)

djsv
djs-version


Notes
- All commands return either 0 (success/running) or 1 (failure/stopped).
- djs-status prints the daemon's process ID (PID).
- Special shell characters (e.g., "|", ";", "&") must be quoted or escaped. For the sake of simplicity and consistency, single-quote all arguments as a whole (e.g., djsc -a '2200 reboot -p').
- PATH starts with /data/adb/vr25/bin:/dev/.vr25/busybox.
This means schedules don't require additional busybox setup.
The first directory holds user executables.
```

---
## NOTES/TIPS FOR FRONT-END DEVELOPERS


### Basics

DJS does not require Magisk.
Any root solution is fine.

Use `/dev/.vr25/djs/*` over regular commands.
These are guaranteed to be readily available after installation/upgrades.

It may be best to use long options over short equivalents - e.g., `/dev/.vr25/djs/djs-config --list`, instead of `/dev/.vr25/djs/djsc -l`.
This makes code more readable (less cryptic).

Include provided descriptions of DJS features/settings in your app(s).
Provide additional information (trusted) where appropriate.
Explain settings/concepts as clearly and with as few words as possible.


### Initializing DJS

DJS is automatically initialized after installation/upgrades.
It needs to be initialized on boot, too.
If it's installed as a Magisk module, this is done by Magisk itself.
Otherwise, the front-end should handle it as follows:
```
on boot_completed receiver and main activity
  if file /dev/.acca/started does NOT exist
    create it
      mkdir -p /dev/.acca
      touch /dev/.acca/started
    if accd is NOT running
      launch it
        /data/adb/vr25/djs/service.sh
    else
      do nothing
  else
    do nothing
```
`/dev/` is volatile - meaning, a reboot/shutdown clears `/dev/.acca/` and its contents.
That's exactly what we want.
Of course, `/dev/.acca/started` is just an example.
One can use any random path (e.g., `.myapp/initialized`), as long as it's under `/dev/` and does not conflict with preexisting data.
**WARNING**: do not play with preexisting /dev/ data!
Doing so may result in data loss and/or other undesired outcome.


---
## FREQUENTLY ASKED QUESTIONS (FAQ)


> How do I report issues?

Open issues on GitHub or contact the developer on Telegram/XDA (linked below).
Always provide as much information as possible.


> Where do I find daemon logs?

`/dev/.vr25/djs/djsd.log`


---
## NOOB FRIENDLY

### Prequisites
- Magisk Rooted
- Root File Explorer & Text Editor
- Your Script to be executed
- Confidence
### Steps
1. Open the config.txt file at /data/adb/vr25/djs-data/
2. Move to end of the text and create a new line
3. Start with writing "boot" (without the quotations) if you want the script to executed on boot.
4. Or Start with writing the time at which you want the script to run in HH:MM format. Ex. "21:30".
5. Now continue in the same line with a space and paste your script. You're done.
### Extras
6. It is possible to Add "; : --delete" to the script if you want the script to run only for first boot.
7. In case of time based command you can Add "; : --boot" to the script if you want the script to run also at boot.


---
## LINKS

- [Donate - Airtm, username: ivandro863auzqg](https://app.airtm.com/send-or-request/send)
- [Donate - Liberapay](https://liberapay.com/vr25)
- [Donate - Patreon](https://patreon.com/vr25)
- [Donate - PayPal or Credit/Debit Card](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=iprj25@gmail.com&lc=US&item_name=VR25+is+creating+free+and+open+source+software.+Donate+to+suppport+their+work.&no_note=0&cn=&currency_code=USD&bn=PP-DonationsBF:btn_donateCC_LG.gif:NonHosted)
- [Facebook Page](https://fb.me/vr25xda)
- [Telegram Channel](https://t.me/vr25_xda)
- [Telegram Profile](https://t.me/vr25xda)
- [Upstream Repository](https://github.com/VR-25/djs)


---
## LATEST CHANGES

**v2021.10.30 (202110300)**
- General fixes & optimizations

**v2021.11.3 (202111030)**
- Fixed installation issues
- Improved support for the current Magisk canary.

**v2021.12.14 (202112140)**
- "Noob friendly" section by @n-ce;
- Updated build script;
- Updated links in the README.
