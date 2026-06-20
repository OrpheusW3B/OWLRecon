# GoRecon

Advanced web reconnaissance and vulnerability scanning tool written in Go.

## Features

- **HTTP Header Analysis** — Extracts all headers, checks 12+ security headers (CSP, HSTS, X-Frame-Options, etc.), detects debug tokens, analyzes cookie security (Secure, HttpOnly, SameSite)
- **TLS/SSL Deep Analysis** — Certificate details, SHA-256 fingerprint, cipher suites, key exchange, SAN, expiry warnings, wildcard detection
- **Technology Fingerprinting** — Detects 40+ technologies (React, Vue, Angular, WordPress, Django, Rails, etc.) via header and body signatures with confidence scores
- **WAF Detection** — Identifies 15+ WAFs (Cloudflare, AWS WAF, Akamai, Imperva, Sucuri, etc.)
- **CORS Configuration Analysis** — Tests arbitrary origin reflection, null origin, credential access
- **DNS Enumeration** — A, AAAA, CNAME, MX, NS, TXT, SOA, SRV records
- **Subdomain Discovery** — Resolves 100 common subdomains concurrently
- **Port Scanning** — 70 common ports with service detection and banner grabbing (optional full 65535 port range with `-deep`)
- **Web Crawling** — Extracts links, scripts, stylesheets, images, iframes, meta refresh, data-* attributes
- **Form Analysis** — CSRF token detection, method analysis, input enumeration
- **JavaScript Analysis** — API endpoint extraction from JS source code, fetch/XHR patterns
- **Comment & Metadata Extraction** — HTML comments with sensitive keyword detection (password, todo, fixme, secret, etc.)
- **Secret & Credential Detection** — 25 regex patterns for AWS keys, Stripe, GitHub tokens, JWT, database URLs, private keys
- **Wayback Machine Lookup** — Queries archive.org CDX API for historical URLs
- **Vulnerability Scanning** — Reflected XSS (12 payloads), SQL Injection (12 payloads), LFI (8 payloads), Open Redirect (6 payloads), CRLF Injection (3 payloads)

## Output Formats

- **JSON** — Full structured data for automation and integration
- **HTML** — Dark-themed interactive report with severity filtering, distribution bar, CWE badges, and remediation advice

## Installation

```bash
git clone <repo-url>
cd gorecon
go build -o gorecon.exe
```

### Requirements

- Go 1.21+

## Usage

### Interactive TUI Mode

```bash
.\gorecon.exe https://target.com
```

### Headless Mode (faster, no interactive display)

```bash
.\gorecon.exe https://target.com -no-tui
```

### Deep Scan (all 65535 ports)

```bash
.\gorecon.exe https://target.com -deep
```

### Selective Scanning

```bash
.\gorecon.exe https://target.com -no-ports -no-dns -no-sub -no-wayback
```

### Custom Configuration

```bash
.\gorecon.exe https://target.com -threads 50 -rate 20 -timeout 60
```

### All Options

| Flag | Description | Default |
|------|-------------|---------|
| `-timeout <sec>` | Request timeout | 30 |
| `-threads <n>` | Concurrent workers | 20 |
| `-rate <ms>` | Rate limit per request | 50ms |
| `-deep` | Full port range (1-65535) | false |
| `-no-tui` | Headless mode | false |
| `-no-tls` | Skip TLS analysis | false |
| `-no-vuln` | Skip vulnerability scanning | false |
| `-no-crawl` | Skip web crawling | false |
| `-no-dns` | Skip DNS enumeration | false |
| `-no-ports` | Skip port scanning | false |
| `-no-waf` | Skip WAF detection | false |
| `-no-cors` | Skip CORS analysis | false |
| `-no-sub` | Skip subdomain discovery | false |
| `-no-wayback` | Skip Wayback Machine | false |
| `-no-secrets` | Skip secret detection | false |
| `-no-js` | Skip JavaScript analysis | false |

## Example Output

```
  SCANNING │ Target: https://target.com │ Elapsed: 19s │ Tasks: 14/14 │ Findings: 47

  ═══ SCAN COMPLETE ═══

    Status Code:    200
    Title:          Target Website
    Technologies:   8 detected
    Links:          142 found
    Forms:          3 found
    Endpoints:      89
    Emails:         2
    Subdomains:     5
    Open Ports:     4
    DNS Records:    6
    Secrets:        1
    Findings:       47

    CRITICAL:  1
    HIGH:      5
    MEDIUM:    12
    LOW:       18
    INFO:      11

  JSON report saved: report_target.com_20260620_201900.json
  HTML report saved: report_target.com_20260620_201900.html
```

## Project Structure

```
.
├── main.go       # Entry point, TUI, task orchestration
├── scanner.go    # All scanning modules (14+ scanners)
├── report.go     # JSON and HTML report generation
├── go.mod
└── README.md
```

## Architecture

All scanners run concurrently using goroutines with a configurable semaphore-based worker pool. Each scanner independently sends findings to the TUI for live display. Reports are generated after all scanners complete.

```
┌─────────────┐
│   main.go   │ ── Spawns goroutines per scanner
└──────┬──────┘
       │
       ├── scanHeaders()    ── Security headers, cookies
       ├── scanTLS()        ── Certificate analysis
       ├── scanTech()       ── Technology fingerprinting
       ├── scanWAF()        ── WAF detection
       ├── scanCORS()       ── CORS misconfiguration
       ├── scanDNS()        ── DNS records
       ├── scanSubdomains() ── Subdomain brute-force
       ├── scanPorts()      ── TCP port scanning
       ├── scanCrawl()      ── Link/endpoint extraction
       ├── scanForms()      ── Form analysis
       ├── scanJS()         ── JavaScript API discovery
       ├── scanComments()   ── HTML comment extraction
       ├── scanSecrets()    ── Credential/secret detection
       ├── scanWayback()    ── Archive.org lookup
       └── scanVulns()      ── XSS, SQLi, LFI, etc.
```

## Disclaimer

This tool is intended for authorized security testing and educational purposes only. Always obtain proper authorization before scanning any target you do not own or have explicit permission to test. The author assumes no responsibility for misuse of this tool.
