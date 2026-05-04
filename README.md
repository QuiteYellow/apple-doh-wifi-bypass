# Apple DoH + Wi-Fi Bypass DNS Profile (iOS / macOS / iPadOS)

An Apple Configuration Profile (`.mobileconfig`) that gives you ad blocking and encrypted DNS on your iPhone, iPad, or Mac — no VPN needed.

If you run [Pi-hole](https://pi-hole.net/) or [AdGuard Home](https://adguard.com/adguard-home.html) on your home network, this profile keeps your devices using it while you're home and automatically falls back to a DoH server with ad blocking everywhere else.

- **At home**: your local Pi-hole or AdGuard Home handles DNS, so network-wide ad blocking works as normal
- **On cellular and unknown Wi-Fi**: DNS is routed through a DoH (DNS-over-HTTPS) server with ad blocking built in (e.g. [LibreDNS noads](https://libredns.gr/), [NextDNS](https://nextdns.io/), [Cloudflare for Families](https://developers.cloudflare.com/1.1.1.1/setup/)), encrypting queries from ISPs and public network operators
- **No VPN**: nothing is tunneled — DNS is controlled directly at the OS level

**Platforms:** iOS 14+, iPadOS 14+, macOS 11 Big Sur+

## Why this config is unique

Most DoH mobileconfig profiles take an all-or-nothing approach: encrypted DNS everywhere, or not at all. This Apple Configuration Profile uses Apple's `OnDemandRules` mechanism (borrowed from the VPN profile spec) inside a `com.apple.dnsSettings.managed` payload.

The key is SSID-scoped bypass: the `SSIDMatch` rule detects the trusted network by name and issues a `Disconnect` action *before* the catch-all `Connect` rules fire. The `SSIDMatch` rule must come first. Apple evaluates `OnDemandRules` top-to-bottom and stops at the first match.

Unlike split-DNS setups that route all traffic through a VPN to control DNS, this profile controls DNS resolution directly at the OS level without touching your traffic.

## Using the template

1. Copy `dns-doh-wifi-bypass.mobileconfig.example` to your desired filename (e.g. `my-dns.mobileconfig`)
2. Replace all placeholders:

| Placeholder | Description |
|---|---|
| `YOUR_DOH_SERVER_URL` | Your DoH server URL — see suggestions below |
| `YOUR_HOME_SSID` | Your trusted Wi-Fi network name |
| `YOUR_REVERSE_DOMAIN` | Reverse-DNS identifier for your payload (e.g. `locals.yourname`) |
| `YOUR_UUID_1` | A generated UUID for the DNS payload |
| `YOUR_UUID_2` | A generated UUID for the top-level profile payload |

The reverse-DNS identifier doesn't need to be a domain you own. `local.yourname` is fine, it's used as a unique identifier inside the profile.

Generate UUIDs with `uuidgen` on macOS/Linux.

### DoH servers with ad blocking

**US-based**

| Provider | DoH URL | Notes |
|---|---|---|
| [NextDNS](https://nextdns.io/) | `https://dns.nextdns.io/YOUR_ID` | Customizable blocklists, free tier, detailed logs dashboard |
| [Cloudflare for Families](https://developers.cloudflare.com/1.1.1.1/setup/) | `https://family.cloudflare-dns.com/dns-query` | Blocks malware + adult content; no account needed |
| [Control D](https://controld.com/free-dns) | `https://freedns.controld.com/p1` | Free ad-blocking preset, no account needed |

**EU-based (GDPR-friendly)**

| Provider | DoH URL | Notes |
|---|---|---|
| [Mullvad DNS](https://mullvad.net/en/help/dns-over-https-and-dns-over-tls/) | `https://adblock.doh.mullvad.net/dns-query` | Sweden, strict no-logs, no account needed |
| [dns0.eu](https://www.dns0.eu/) | `https://zero.dns0.eu/` | EU-operated, GDPR compliant, blocks ads + trackers |
| [LibreDNS](https://libredns.gr/) | `https://doh.libredns.gr/noads` | Greece, open source, no logs |
| [AdGuard DNS](https://adguard-dns.io/en/public-dns.html) | `https://dns.adguard-dns.com/dns-query` | Cyprus, blocks ads + trackers, no account needed |

3. Install it: AirDrop the `.mobileconfig` file to your iPhone, iPad, or Mac, or host it on a local web server and open the URL in Safari. Go to **Settings → General → VPN & Device Management** to approve it.

## Troubleshooting

If sites aren't loading, the DoH server is the most likely cause.

**Temporarily disable** (without removing): Settings → General → VPN & Device Management → DNS → switch to Automatic. Switch back to the profile name to re-enable.

**Remove entirely:**
- **iOS / iPadOS**: Settings → General → VPN & Device Management → tap the profile → Remove Profile
- **macOS**: System Settings → Privacy & Security → Profiles → select the profile → click the remove (–) button
