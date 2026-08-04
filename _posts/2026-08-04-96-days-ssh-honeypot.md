---
layout: post
title: "I Left an SSH Honeypot Online for 96 Days. The IP Count Was the Least Interesting Part"
date: 2026-08-04
categories: [security]
tags: [honeypot, cowrie, threat-intel, malware-analysis]
---

In late April, I spun up a small cloud instance, installed [Cowrie](https://github.com/cowrie/cowrie), exposed its SSH listener on port 2222, and left it online for 96 days.

Cowrie is a medium interaction SSH honeypot. It accepts connections and presents a convincing fake shell. It records commands, terminal activity, authentication attempts, and supported file transfers. It emulates the interaction, but it does not execute uploaded payloads on the host.

The final dataset contained 120,963 events in 71.6 MB of newline-delimited JSON. Those events represented 19,833 sessions from 3,668 external source IP addresses.

Calling those addresses 3,668 attackers would be easy. It would also be misleading. That number turned out to be one of the least useful measurements in the dataset.

## The shape of the traffic

Eighty-six percent of sessions never submitted a command. Many connected and disconnected before authentication. Others guessed a credential and immediately left. The median session lasted 2.6 seconds.

That is not a person carefully exploring a server. It is automated software working through an address range, and the sensor happened to be one address in its path.

Only 2,677 sessions produced command input. Of those, 2,653 planted the same SSH key. That is 99.1 percent of all sessions that produced commands.

Only 24 command sessions did not plant that key. Nine still ran the campaign's opening command. One ran the same hardware survey. Two were loopback tests. That left at most 12 external sessions with other command sequences.

The large numbers described repetition. The small remainder contained most of the behavioral variety.

![Daily Cowrie activity showing campaign onset and sensor degradation](/assets/images/blog/cowrie/cowrie-activity-timeline.png)

## The credential trap I almost fell into

Two credential pairs dominated the login attempts:

```text
2,475   root / 3245gs5662d34
2,472   345gs5662d34 / 345gs5662d34
```

These are unlikely to be attempts at plausible human passwords. Their fixed and unusual format suggests automated scanning with a hardcoded credential list.

My first instinct was to treat them as a fingerprint for the campaign that dominated the logs. I tested that assumption before writing it down.

The result was clear. The 4,947 sessions using this credential family produced zero command input. The campaign that planted the SSH key used 2,103 distinct passwords and never tried `3245gs5662d34`. The two populations did not overlap.

[TEHTRIS reported](https://tehtris.com/en/blog/honeypots-activity-of-the-week-50/) these strings in more than 30 percent of attempts during one week in December 2022. [SANS ISC later highlighted](https://isc.sans.edu/diary/31360) the same unusual credential family. An [Aalborg University paper](https://vbn.aau.dk/ws/files/573748244/sweetcam_honeypot_paper_1_.pdf) described it as possibly associated with a default credential for the Polycom CX600 IP phone.

That does not identify one botnet in this dataset. It identifies a credential family that has been sprayed across many unrelated sensors for years.

If a honeypot's top credential also appears near the top of public honeypot reports, it may describe the internet's noise floor more than the sensor's unique exposure.

## Fingerprinting the campaign properly

Credentials were weak attribution. HASSH provided a better pivot.

[HASSH was developed at Salesforce](https://engineering.salesforce.com/open-sourcing-hassh-abed3ae5044c/) as a fingerprinting method for SSH clients and servers. It hashes selected algorithm lists from the SSH key exchange in their offered order. Those choices help identify an SSH implementation and configuration even when the source address changes.

Across 17,535 sessions, the sensor recorded 44 distinct HASSH values. One fingerprint, `f555226df1963d1d3c09daf865abdc9a`, appeared in 10,876 sessions across 1,148 source addresses.

That fingerprint exactly matches the value documented by the researcher at [blog.port22.dk](https://blog.port22.dk/mdrfckrs-part-two/). The same reporting connects it to the `mdrfckr` key planting activity observed across tens of thousands of addresses.

The correlation in this dataset was strong, but it was asymmetric.

All 2,653 `mdrfckr` key plants used one of five libssh fingerprints. That gave the five fingerprints complete coverage of the observed key plants. It did not make them precise standalone indicators.

Across all five fingerprints, 2,653 of 14,529 sessions planted the key. That is 18.3 percent. For the primary fingerprint alone, 2,115 of 10,876 sessions planted it. That is 19.4 percent.

HASSH identifies a client implementation and configuration. It does not identify intent, a specific operator, or malware by itself. It is useful as a hunting pivot when combined with commands, credentials, and other behavior. It is a poor blocklist criterion on its own because legitimate software also uses libssh.


## What the dominant campaign did

The dominant campaign repeatedly used two commands. Its opening command appeared in 2,660 sessions. The SSH key plant appeared in 2,653 sessions. Both appeared together in 2,651 sessions.

A normalized version of the sequence is shown below. The public key has been shortened for readability.

```bash
cd ~; chattr -ia .ssh; lockr -ia .ssh
cd ~ && rm -rf .ssh && mkdir .ssh && echo "ssh-rsa AAAAB3NzaC1...== mdrfckr" >> .ssh/authorized_keys && chmod -R go= ~/.ssh && cd ~
```

The `chattr -ia` command removes immutable and append-only attributes from `.ssh`. The following `lockr` command is not a standard Linux utility, and Cowrie logged it as a failed command. It appears to be either a typo or an assumption specific to the campaign.

The second command deletes the entire `.ssh` directory, recreates it, and installs the attacker's key. This removes competing botnet keys, legitimate administrator keys, and other SSH configuration stored in that directory.

The key and command sequence match the `mdrfckr` activity documented by port22.dk. [Kaspersky associates](https://securelist.com/outlaw-botnet/116444/) `mdrfckr` authorized keys with Outlaw, also known as Dota. [Trend Micro has tracked](https://www.trendmicro.com/en_us/research/18/k/outlaw-group-distributes-botnet-for-cryptocurrency-mining-scanning-and-brute-force.html) Outlaw's SSH-based cryptomining activity since 2018.

This supports an Outlaw assessment for the dominant activity. The attribution comes from the combined key, commands, client fingerprints, and established reporting. It does not come from HASSH alone.

A total of 177 sessions also ran a longer discovery sequence. The commands collected core count, CPU model, memory, disk capacity, cron entries, logged in users, and active processes. Some sessions ended before the sequence completed.

The observed command set focused on resource discovery and persistence. It did not show attempts to search for business data, collect credentials from the fake filesystem, or move laterally. That behavior is consistent with a cryptomining campaign evaluating available hardware and checking for competition.

## A payload set consistent with RedTail

On July 30, one source uploaded seven files in 30.421 milliseconds:

```text
clean.sh
redtail.arm7  redtail.arm8  redtail.i686  redtail.riscv  redtail.x86_64
setup.sh
```

The filenames represent five architecture targets, including RISC-V. A [previous SANS honeypot analysis](https://isc.sans.edu/diary/31568) documented a similar RedTail delivery set with four architecture builds, `clean.sh`, and `setup.sh`.

The scripts in this collection also point toward RedTail. The setup logic uses `redtail` as its final filename fallback. The filenames, script roles, cleanup behavior, and architecture selection align with published RedTail activity.

That supports a working assessment of activity consistent with RedTail. It is not proof of attribution. Public repositories currently apply other labels, including Prometei and `wraith`, to some of the exact hashes. Malware family labels often conflict when tooling, scripts, or delivery infrastructure are shared. The observed behavior is more certain than the family name.

The two shell scripts still exposed the installation strategy without requiring the binaries to be executed.

### clean.sh protects the miner's access to the CPU

The script does not clean the system for the administrator. It removes competing persistence and processes so the new miner receives more resources.

Its cron cleanup is selective:

```bash
clean_file() {
  chattr -ia "$1"
  grep -vE 'wget|curl|/dev/tcp|/tmp|\.sh|nc|bash -i|sh -i|base64 -d' "$1" > /tmp/clean_file
  mv /tmp/clean_file "$1"
}
```

The function does not empty each file. It removes lines matching common malware persistence patterns and retains everything else. That is more restrained than deleting the entire crontab. It is consistent with keeping the host usable while removing competitors.

The script applies this filter across user crontabs, system crontabs, cron directories, `anacrontab`, the live table, and shell startup files. It also names one competitor directly with `systemctl disable c3pool_miner`.

Three suspicious binaries are truncated before their associated processes are killed:

```bash
echo > /bin/systemtd
echo > /bin/-bash
echo > /usr/bin/.sh
```

The names resemble common process hiding choices. `systemtd` is visually close to `systemd`. A process named `-bash` can resemble an ordinary login shell. The `.sh` filename is hidden from a plain `ls` listing.

### setup.sh anticipates common staging controls

The installer checks for `noexec` filesystems with `findmnt`. It falls back to `/proc/mounts` when `findmnt` is unavailable. Those paths are excluded from candidate staging locations.

It also prefers locations outside `/tmp`. The script searches the filesystem for directories owned by the current user. It excludes `/tmp` from that search and uses common temporary directories only as fallbacks.

The script tests more than simple writability. It attempts to create a 2 MB file with `dd`, then falls back to `truncate`. This can identify some quota and capacity problems before deployment. It does not guarantee enough room for every payload, but it is more robust than checking permissions alone.

The selected binary is copied to a randomized dot-prefixed filename. A detector that relies on one fixed filename would miss it.

The payload is then launched with the argument `ssh`. Public RedTail reporting describes SSH propagation capabilities, so the argument is consistent with enabling a spreader function. The scripts alone do not prove the binary's internal argument handling. That conclusion should remain an assessment until the binaries are unpacked and reverse engineered.

The random name generator tries `openssl`, then `/dev/urandom`, then Bash's `$RANDOM`. If all three fail, it uses the literal fallback `redtail`. On a minimal system, the installer names the payload after the family it appears to represent.

## The activity outside the shell

The sensor also recorded 257 `direct-tcpip` requests from 221 source addresses. These requests attempted to use SSH forwarding instead of submitting shell commands.

Of the 257 requests, 256 targeted `77.88.21.158:25`. The destination is associated with Yandex SMTP infrastructure, and port 25 is used for SMTP transport.

This activity is consistent with attempts to use a compromised SSH host as a TCP proxy for mail activity. The logs do not contain mail contents, so they do not prove open relay testing or successful spam delivery.

Setting `AllowTcpForwarding no` would reject these specific SSH forwarding requests. The [OpenSSH documentation](https://man.openbsd.org/sshd_config) notes that users with shell access can still install their own forwarding tools. Preventing unauthorized authentication remains the primary control.

This activity would be invisible in an analysis limited to command events.

## Two things I initially got wrong

I first flagged 177 sessions as a second payload campaign because Cowrie recorded a file download to `/etc/hosts.deny`. The hash was `01ba4719c80b6fe911b091a7c05124b64eeece964e09c058ef8f9805daca546b`.

That is the SHA-256 hash of a single newline character.

The command `echo > /etc/hosts.deny` created the file event. Cowrie recorded shell output redirection as a file download, which made an empty file look like a captured payload. A file hash should always be checked against trivial content before it is treated as a malware sample.

I also nearly placed `216.180.246.0/24` at the top of a worst offenders table. It was the busiest `/24` by connection count, with 281 sessions.

Those sessions produced no authentication attempts and no commands. The client version data contained a mixture of HTTP paths, TLS handshakes, and SSH banners. That pattern is consistent with multiprotocol internet measurement or survey scanning. It is not evidence of intrusion by itself.

Connection volume measured how often a source touched the sensor. It did not measure how far the source progressed.

## A collection caveat

A systemd misconfiguration degraded collection beginning July 15. Connections were still logged, but sessions rarely progressed to authentication.

The average connection volume fell from 256.1 per day before the degradation to 18.5 per day afterward. Command input nearly disappeared.

The cause was mundane. The unit passed a `-n` flag that Cowrie's shell wrapper understood. The console script inside the virtual environment did not accept the same flag. The names looked interchangeable, but their argument parsers were not.

The Outlaw activity ended in the logs on July 14, immediately before the degradation. I cannot conclude that the campaign stopped. The sensor may simply have stopped recording the stage where the campaign became visible.

Long-lived sensors need collection health monitoring. Raw volume is not enough. Authentication-to-connection ratios, command-to-connection ratios, expected event stages, and direct service checks can help distinguish a quiet day from a failing sensor.

The sensor worked well during its healthy window. The degraded period still captured a few useful sessions, including the payload set consistent with RedTail. Its limitations must remain part of the analysis.

## Takeaways

**A new address does not stay unremarkable for long.** The instance had no useful reputation, DNS presence, or inbound links. Automated login attempts still arrived within hours.

**Counting source IPs is not analysis.** The raw log contained 3,668 external source addresses. One repeated key plant accounted for 99.1 percent of sessions that produced commands. Source count alone did not reveal that concentration.

**Compromised hosts are contested territory.** The observed activity deleted competing SSH keys, cleared persistence, disabled rival miners, and searched for disguised processes. Resource theft creates competition between criminal operators.

**Some commodity malware anticipates common controls.** The installer in the RedTail-consistent session avoided `noexec` mounts, searched outside `/tmp`, tested candidate storage, and randomized its installed filename. Strong authentication would still have prevented initial access.

**Read the scripts before the binaries.** Two small shell scripts exposed the staging logic, competitor list, persistence surfaces, and family clues. The binaries still require deeper reverse engineering.

**Attribution should express confidence.** Filenames, script strings, public reporting, hashes, and behavior can support a family assessment. None should be treated as proof in isolation.

## Hardening that would have stopped the observed activity

Nothing in this dataset required a software vulnerability. Every successful command or forwarding session followed password authentication as root.

- Set `PasswordAuthentication no` when password-based SSH access is not required.
- Set `KbdInteractiveAuthentication no` when keyboard-interactive authentication is also unnecessary.
- Set `PermitRootLogin no`. Every successful authentication in this dataset used the root account.
- Monitor `authorized_keys` with file integrity monitoring. Both major command sequences modified it.
- Do not rely on `chattr +i` alone. Both sequences removed immutable attributes before replacing keys.
- Mount `/tmp`, `/var/tmp`, and `/dev/shm` as `noexec` as defense in depth. Understand that the observed installer searched for other locations.
- Set `AllowTcpForwarding no` unless forwarding is required. This would reject the 257 observed `direct-tcpip` requests.
- Alert on `systemtd`, `/bin/-bash`, `/usr/bin/.sh`, `c3pool_miner`, or crontabs that unexpectedly lose entries.



## Indicators of compromise

### File hashes

The following files were delivered in the session consistent with RedTail. The family assessment remains qualified because public classification currently conflicts.

| File | SHA-256 |
|---|---|
| `clean.sh` | `197c74408e15bd1168105f564f96aace4fd4819961b724630bf5a6be4878daf8` |
| `setup.sh` | `31d4181843b1ed10a7e7cb3f108f6d6c50a7a4452ee52ddacabe8ca77260615e` |
| `redtail.arm7` | `d70f917e35813a7ae323e6b2b539d6dbbfc3a3a6599f1fed93430b14ca08b141` |
| `redtail.arm8` | `d1cac82f44b54b0fd244a9e4122811e9ae108a197c7a65a20fd2e7552683e68e` |
| `redtail.i686` | `8e1a67a5c03b3cd818f046c7a1605afccc0ee5ce437a0d099881f1872b54bc70` |
| `redtail.riscv` | `3f3bf218089d1488617d37f8a5116bb2791eb39ce06a1b5bc9a4cdfe5e94dd39` |
| `redtail.x86_64` | `f0aa83bbbd2c75e2f71ec16029ee5fcfad59f3a8efa30a500b815f0f6c18d987` |

Hash `01ba4719c80b6fe911b091a7c05124b64eeece964e09c058ef8f9805daca546b` appeared 177 times as a file download. It is the SHA-256 of one newline character, not malware.

### HASSH fingerprints associated with the key plant

| HASSH | Client |
|---|---|
| `f555226df1963d1d3c09daf865abdc9a` | libssh 0.9.6 |
| `03a80b21afa810682a776a7d42e5e6fb` | libssh 0.11.1 |
| `af8223ac9914f509afdadfaf5f7ee94e` | libssh 0.12.0 |
| `713bd9cc935561939c02dad25af2d3de` | libssh 0.11.1 |
| `671ac49b8bd65b9e8ff02a3e690f0fd3` | libssh 0.9.6 |

Use these as hunting pivots, not blocklist entries. Every `mdrfckr` key plant used one of these fingerprints. Across all five values, 18.3 percent of sessions planted the key. The primary fingerprint alone had a 19.4 percent key plant rate.

### Network indicators

These indicators are time-bounded. Source addresses may be compromised intermediaries, proxies, or shared infrastructure rather than operator locations.

| Indicator | Context |
|---|---|
| `45.148.10.68` | Delivered the payload set consistent with RedTail on 2026-07-30 |
| `47.105.90.130` | Submitted a loader with multiple fallbacks on 2026-05-05 |
| `47.86.2.149:60122` | Payload host referenced by that loader |
| `8.152.5.15` | Attempted an inline ELF transfer on 2026-07-27 |
| `77.88.21.158:25` | Target of 256 SSH direct forwarding requests |
| `192.42.116.45` | Tor exit address that submitted container detection probes |
| `192.42.116.51` | Tor exit address that submitted container detection probes |
| `102.88.137.80` | Highest volume `mdrfckr` source, with 65 key plants |

`216.180.246.0/24` is intentionally excluded. It produced multiprotocol survey traffic without authentication or command activity.

### Behavioral indicators

| Indicator | Meaning |
|---|---|
| `ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEArDp4cun2lhr4KUhBGE7V...` | Outlaw-associated backdoor key |
| `...AsPKgAySVKPRK+oRw== mdrfckr` | Trailing bytes and comment from the same key |
| `chattr -ia .ssh; lockr -ia .ssh` | Attribute removal attempt before key replacement |
| `rm -rf .ssh && mkdir .ssh` | Deletes existing SSH keys and configuration |
| `echo "root:<12 random alnum>"|chpasswd|bash` | Sets a unique 12-character root password |
| `pkill -9 secure.sh` and `pkill -9 auth.sh` | Removes competing or unwanted processes |
| `echo > /etc/hosts.deny` | Clears legacy TCP Wrappers deny rules |
| `echo -e "\x61\x75\x74\x68\x5F\x6F\x6B\x0A"` | Completion marker that decodes to `auth_ok` |
| `./<random> ssh` | Execution pattern consistent with RedTail that likely enables SSH propagation |
| `ls /proc/1/` with `cat /proc/1/mounts` | Container detection |

### Suspected rival implant names from clean.sh

If these exist on a host, investigate the system for compromise. Their presence is not evidence of RedTail by itself.

| Name | Disguise or context |
|---|---|
| `/bin/systemtd` | Typosquat of `systemd` |
| `/bin/-bash` | Resembles a login shell process name |
| `/usr/bin/.sh` | Dotfile hidden from plain `ls` |
| `c3pool_miner` | C3Pool Monero mining service |
| `secure.sh`, `auth.sh` | Suspected competing staging scripts in `/tmp` |
