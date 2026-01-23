# Complete Playwright PDF Generation Guide with SSL

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding Playwright Architecture](#understanding-playwright-architecture)
3. [SSL Certificate Management](#ssl-certificate-management)
4. [Docker Environment Setup](#docker-environment-setup)
5. [Python Implementation](#python-implementation)
6. [Security Best Practices](#security-best-practices)
7. [Production Deployment](#production-deployment)

---


Introduction

What is Playwright?

Playwright is a browser automation framework developed by Microsoft that allows you to control Chromium, Firefox, and WebKit browsers programmatically. Unlike traditional PDF generation libraries, Playwright uses a real browser engine, which means:

Perfect rendering: Exactly what users see in their browser

Full CSS support: Including modern features like Grid, Flexbox, animations

JavaScript execution: Dynamic content renders correctly

Font rendering: System fonts and web fonts work seamlessly


Why Playwright for PDF Generation?

Traditional PDF libraries (like WeasyPrint, ReportLab) have limitations:

❌ Poor CSS support (especially modern CSS)

❌ No JavaScript execution

❌ Font rendering issues

❌ Layout inconsistencies

Playwright solves these by using Chrome's native PDF engine:

✅ 100% accurate rendering

✅ Full modern web standards support

✅ Print-quality output

✅ Production-ready stability



Understanding Playwright Architecture

How Playwright Works Internally

```
                     Your Python Application

  ┌──────────────────────────────────────────────────────┐
  │         Playwright Python Library                     │
  │  (playwright.async_api.async_playwright)             │
  └────────────────┬─────────────────────────────────────┘

                   │ WebSocket Communication

  ┌────────────────▼─────────────────────────────────────┐
  │      Playwright Server (Node.js Process)             │
  │  - Manages browser lifecycle                         │
  │  - Handles protocol translation                      │
  │  - Coordinates multiple browser instances            │
  └────────────────┬─────────────────────────────────────┘
└───────────────────┼──────────────────────────────────────────┘
                    │
                    │ Chrome DevTools Protocol (CDP)
                    │
    ┌───────────────▼─────────────────┐
    │   Chrome Browser Process        │
    │                                  │
    │  ┌────────────────────────────┐ │
    │  │   Rendering Engine         │ │
    │  │   - Blink (HTML/CSS)      │ │
    │  │   - V8 (JavaScript)       │ │
    │  │   - Skia (Graphics)       │ │
    │  └────────────────────────────┘ │
    │                                  │
    │  ┌────────────────────────────┐ │
    │  │   Network Stack            │ │
    │  │   - HTTPS/TLS Handling    │ │
    │  │   - Certificate Validation│ │
    │  └────────────────────────────┘ │
    │                                  │
    │  ┌────────────────────────────┐ │
    │  │   PDF Generation Engine    │ │
    │  │   - Skia PDF Backend      │ │
    │  │   - Print Layout          │ │
    │  └────────────────────────────┘ │
    └──────────────────────────────────┘
```

Key Components Explained

1. Playwright Python Library

```python
from playwright.async_api import async_playwright

async with async_playwright() as p:
    # p is the Playwright instance
    # Manages connection to Playwright server
```

What happens internally:

Spawns a Node.js process running Playwright server
Establishes WebSocket connection for communication
Provides Python API that translates to browser commands


2. Browser Launch Process

```python
browser = await p.chromium.launch(
    channel="chrome",
    headless=True,
    args=[...]
)
```

What happens internally:

Playwright locates Chrome binary on system
Launches Chrome with specified command-line arguments
Connects via Chrome DevTools Protocol (CDP)
Returns browser handle for control


3. Browser Context

```python
context = await browser.new_context(
    viewport={"width": 1920, "height": 1080},
    ignore_https_errors=False
)
```

What is a Context?

Isolated browser session (like Incognito mode)
Has its own cookies, cache, local storage
Multiple contexts = multiple isolated sessions in one browser

Why use contexts instead of multiple browsers?

Performance: Contexts share browser process
Resource efficiency: Less memory per context
Isolation: Each context is completely independent


4. Page Object

```python
page = await context.new_page()
```

**What is a Page?**
- Represents a single browser tab
- Can load HTML, execute JavaScript, capture screenshots
- Provides full control over the page lifecycle

---

## SSL Certificate Management

### Understanding SSL/TLS in Browsers

When Chrome loads `https://example.com`:

```
1. TCP Connection
   └─> Client: "Hello, let's use TLS"

2. TLS Handshake
   └─> Server: "Here's my certificate"

3. Certificate Validation
   ├─> Check: Is certificate signed by trusted CA?
   ├─> Check: Is domain name correct?
   ├─> Check: Is certificate expired?
   └─> Check: Is certificate revoked?

4. If Valid:
   └─> Encrypted communication established

5. If Invalid:
   └─> net::ERR_CERT_AUTHORITY_INVALID
```

### The Custom CA Challenge

**Scenario**: Your company uses internal S3 (Yotta) with custom CA certificate

**Problem**: Chrome doesn't trust custom CAs by default

```
Browser tries to load: https://nm1ecs.yotta.com:9021/image.png

Chrome checks certificate:
├─> Issued by: "Yotta CA"
├─> Chrome's trust store: ❌ "Yotta CA" not found
└─> Result: net::ERR_CERT_AUTHORITY_INVALID
```

Solution: Multi-Layer Certificate Trust

We need to install the certificate in THREE places because Chrome checks multiple certificate stores:

Layer 1: System Certificate Store

```dockerfile
# Copy custom CA certificate
COPY yotta-ca.pem /usr/local/share/ca-certificates/yotta-ca.crt

# Add to system trust store
RUN update-ca-certificates
```

What this does:

Copies certificate to /usr/local/share/ca-certificates/
update-ca-certificates reads all .crt files
Adds them to /etc/ssl/certs/ca-certificates.crt
Python libraries (boto3, requests) now trust this CA

Verification:

```bash
# Check if certificate was added
ls -la /etc/ssl/certs/ | grep yotta

# Verify system trust
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt yotta-server.crt
```

Layer 2: Chrome NSS Database

Chrome uses Network Security Services (NSS) for certificate management:

```dockerfile
# Create Chrome's certificate database
RUN mkdir -p /root/.pki/nssdb && \
    certutil -d sql:/root/.pki/nssdb -N --empty-password && \
    certutil -d sql:/root/.pki/nssdb -A -t "C,," -n "Yotta CA" -i /usr/local/share/ca-certificates/yotta-ca.crt
```

Breaking down the command:

```bash
# Create NSS database directory
mkdir -p /root/.pki/nssdb

# Initialize new NSS database
certutil -d sql:/root/.pki/nssdb -N --empty-password
# -d: database path (sql: prefix for SQLite format)
# -N: create new database
# --empty-password: no password protection

# Add certificate to database
certutil -d sql:/root/.pki/nssdb -A -t "C,," -n "Yotta CA" -i yotta-ca.crt
# -A: add certificate
# -t "C,,": trust flags (C = trusted CA)
# -n: nickname for certificate
# -i: input file
```

Trust flags explained:

C,, = Trust for SSL/TLS certificates
First position: SSL trust
Second position: Email trust
Third position: Object signing trust

Why Chrome needs this:

Chrome doesn't use system certificate store on Linux. It maintains its own database in ~/.pki/nssdb.

Verification:

```bash
# List certificates in NSS database
certutil -d sql:/root/.pki/nssdb -L

# Should show:
# Certificate Nickname         Trust Attributes
# Yotta CA                     C,,
```

Layer 3: Environment Variables

```python
os.environ['SSL_CERT_FILE'] = '/etc/ssl/certs/ca-certificates.crt'
os.environ['SSL_CERT_DIR'] = '/etc/ssl/certs'
os.environ['REQUESTS_CA_BUNDLE'] = '/etc/ssl/certs/ca-certificates.crt'
os.environ['AWS_CA_BUNDLE'] = '/etc/ssl/certs/ca-certificates.crt'
```

**What each variable does:**

| Variable | Used By | Purpose |
|----------|---------|---------|
| `SSL_CERT_FILE` | Python SSL module | Points to CA bundle file |
| `SSL_CERT_DIR` | Python SSL module | Directory containing CA certs |
| `REQUESTS_CA_BUNDLE` | requests library | CA bundle for HTTP requests |
| `AWS_CA_BUNDLE` | boto3/botocore | CA bundle for AWS S3 uploads |

**Why needed:**
- Python libraries don't automatically find custom CAs
- These variables tell Python where to look for certificates
- Critical for S3 uploads to work with custom CA

### Certificate Installation Flow

```
Docker Build Time:
├─> COPY yotta-ca.pem → /usr/local/share/ca-certificates/
├─> RUN update-ca-certificates → adds to system trust
├─> RUN certutil → adds to Chrome NSS database
└─> ENV HOME=/root → Chrome will find NSS database

Container Runtime:
├─> Python starts
├─> Sets environment variables (SSL_CERT_FILE, etc.)
├─> boto3 reads AWS_CA_BUNDLE → trusts Yotta for S3
├─> Playwright launches Chrome
├─> Chrome reads /root/.pki/nssdb → trusts Yotta
└─> HTTPS requests succeed ✅
```

## Docker Environment Setup

Complete Dockerfile Explained

```dockerfile
# Use Python 3.11 slim as base image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# ============================================================
# SECTION 1: Install System Dependencies
# ============================================================
RUN apt-get update && apt-get install -y \
    # Core utilities
    wget \
    gnupg \
    curl \
    ca-certificates \
    # Font support for PDF rendering
    fonts-liberation \
    fonts-noto-color-emoji \
    # Chrome dependencies (sound)
    libasound2 \
    # Chrome dependencies (accessibility)
    libatk-bridge2.0-0 \
    libatk1.0-0 \
    libatspi2.0-0 \
    # Chrome dependencies (printing)
    libcups2 \
    # Chrome dependencies (system integration)
    libdbus-1-3 \
    # Chrome dependencies (graphics)
    libdrm2 \
    libgbm1 \
    libglib2.0-0 \
    libgtk-3-0 \
    # Chrome dependencies (security)
    libnspr4 \
    libnss3 \
    libnss3-tools \
    # Chrome dependencies (display)
    libwayland-client0 \
    libxcomposite1 \
    libxdamage1 \
    libxfixes3 \
    libxkbcommon0 \
    libxrandr2 \
    libx11-6 \
    libxcb1 \
    libxext6 \
    xdg-utils \
    # Cleanup
    && rm -rf /var/lib/apt/lists/*
```

Why each dependency is needed:

PackagePurposeWhat breaks without itlibnss3Certificate handlingSSL verification failslibnss3-toolscertutil commandCan't add custom CAlibgtk-3-0UI toolkitChrome can't renderlibgbm1Graphics bufferHeadless rendering failsfonts-liberationStandard fontsText appears as boxeslibcups2Printing supportPDF generation fails

```dockerfile
# ============================================================
# SECTION 2: Install Custom SSL Certificate
# ============================================================
COPY yotta-ca.pem /usr/local/share/ca-certificates/yotta-ca.crt
RUN update-ca-certificates
```

Certificate format matters:

File MUST have .crt extension (not .pem)
Must be in /usr/local/share/ca-certificates/
Must be PEM format (base64 encoded)

```dockerfile
# ============================================================
# SECTION 3: Install Google Chrome
# ============================================================
RUN wget -q -O /tmp/google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb \
    && apt-get update \
    && apt-get install -y /tmp/google-chrome.deb \
    && rm /tmp/google-chrome.deb \
    && rm -rf /var/lib/apt/lists/*
```

Why not use Playwright's bundled Chromium?

System Chrome is pre-installed (no playwright install)
Better compatibility with system libraries
Easier to update (via apt)
Chrome is optimized for production

```dockerfile
# ============================================================
# SECTION 4: Python Dependencies
# ============================================================
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

**Key dependencies:**

```
playwright==1.40.0
fastapi==0.104.1
uvicorn[standard]==0.24.0
boto3==1.29.7
python-dotenv==1.0.0
```

```dockerfile
# ============================================================
# SECTION 5: Application Code
# ============================================================
COPY . .
```

```dockerfile
# ============================================================
# SECTION 6: Environment Configuration
# ============================================================
ENV PYTHONUNBUFFERED=1
ENV CHROME_BIN=/usr/bin/google-chrome
```

What these do:

PYTHONUNBUFFERED=1: Print logs immediately (no buffering)
CHROME_BIN: Tells Playwright where Chrome is located

```dockerfile
# ============================================================
# SECTION 7: Chrome NSS Database Setup
# ============================================================
RUN mkdir -p /root/.pki/nssdb && \
    certutil -d sql:/root/.pki/nssdb -N --empty-password && \
    certutil -d sql:/root/.pki/nssdb -A -t "C,," -n "Yotta CA" -i /usr/local/share/ca-certificates/yotta-ca.crt && \
    chmod -R 755 /root/.pki

ENV HOME=/root
```

Critical: ENV HOME=/root ensures Chrome looks in /root/.pki/nssdb for certificates.

```dockerfile
# ============================================================
# SECTION 8: Service Configuration
# ============================================================
EXPOSE 8004

HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8004/health || exit 1

CMD ["uvicorn", "pdf_service:app", "--host", "0.0.0.0", "--port", "8004"]
```

Health check explained:

--interval=30s: Check every 30 seconds
--timeout=10s: Wait 10 seconds for response
--start-period=40s: Wait 40 seconds before first check
--retries=3: Mark unhealthy after 3 failures



## Python Implementation

Complete Code Structure

```python
from playwright.async_api import async_playwright
import asyncio

class PDFConverter:
    async def convert_to_pdf(
        self, 
        html_content: str, 
        output_path: str, 
        options: Dict[str, Any]
    ) -> bytes:
        """
        Convert HTML to PDF using Playwright with proper SSL validation.
        
        Architecture:
        1. Launch Chrome with minimal flags
        2. Create isolated context with SSL validation
        3. Load HTML content
        4. Wait for resources (images, fonts)
        5. Generate PDF bytes
        6. Cleanup resources
        
        Returns:
            PDF as bytes (ready for S3 upload or download)
        """
```

Step-by-Step Implementation

Step 1: Initialize Playwright

```python
browser = None
context = None
page = None

try:
    async with async_playwright() as p:
        # Playwright instance created
        # Manages browser lifecycle
```

What happens here:

Spawns a Node.js process running Playwright server
Establishes WebSocket connection for communication
p object is your control interface

Step 2: Launch Browser

```python
browser = await p.chromium.launch(
    channel="chrome",  # Use system Chrome (not Chromium)
    headless=True,     # No GUI (server mode)
    args=[
        # Docker requirements
        "--no-sandbox",
        "--disable-setuid-sandbox",
        "--disable-dev-shm-usage",
        
        # Performance
        "--disable-gpu",
        "--disable-background-networking",
        "--disable-sync",
        "--disable-translate",
        "--disable-extensions",
        
        # Process isolation (CRITICAL)
        "--single-process",
        
        # Tracking
        f"--user-agent=PDFService-{uuid.uuid4()}",
    ]
)
```

**Flag breakdown:**

| Flag | Why Needed | What Breaks Without It |
|------|-----------|----------------------|
| `--no-sandbox` | Docker requires root | Chrome won't start |
| `--disable-setuid-sandbox` | No SUID in containers | Security error |
| `--disable-dev-shm-usage` | Limited `/dev/shm` | Memory allocation fails |
| `--disable-gpu` | No GPU in headless | Uses CPU rendering |
| `--single-process` | Process isolation | State leaks between requests |

**Why `--single-process` is critical:**

```
Without --single-process:
Request 1: Browser process #1234
           └─> Caches certificate validation
Request 2: Reuses process #1234
           └─> Uses cached state (may be invalid)
           └─> Intermittent failures ❌

With --single-process:
Request 1: Fresh process #1234
           └─> Validates certificates fresh
Request 2: Fresh process #5678
           └─> Independent validation
           └─> Consistent behavior ✅
```

Step 3: Create Browser Context

```python
context = await browser.new_context(
    viewport={"width": 1920, "height": 1080},
    bypass_csp=True,
    ignore_https_errors=False,  # ✅ Validate certificates
    java_script_enabled=True,
    accept_downloads=False,
)
```

Context options explained:

```python
viewport={"width": 1920, "height": 1080}
# Sets virtual screen size for rendering
# Affects: @media queries, responsive layouts
# PDF will render as if viewed on 1920x1080 screen

bypass_csp=True
# Bypasses Content Security Policy
# Why: HTML content might have restrictive CSP
# Safe because: We control the HTML content

ignore_https_errors=False  # ⚠️ MOST IMPORTANT
# False = Validate all SSL certificates (secure)
# True = Skip validation (insecure, never use in production)
# Our setup: False works because we installed CA properly

java_script_enabled=True
# Allows JavaScript execution
# Why: Dynamic content, charts, animations

accept_downloads=False
# Prevents file downloads during rendering
# Why: We only want PDF output, no side effects
```

Step 4: Create Page and Load Content

```python
page = await context.new_page()

# Set media type to screen (not print)
await page.emulate_media(media="screen")

# Load HTML content
await page.set_content(html_content, wait_until="networkidle")
```

Media emulation:

```css
/* CSS can target different media */
@media screen {
    .header { background: blue; }
}

@media print {
    .header { background: gray; }
}
```

By using media="screen", we render with screen styles (colors, backgrounds).

Wait strategies:

```python
wait_until="networkidle"
# Waits until:
# - No more than 0-2 network connections for 500ms
# - All resources loaded (images, fonts, CSS)
```

Other options:

"load": Wait for load event only (fast, but may miss resources)
"domcontentloaded": Wait for DOM only (fastest, but no resources)
"networkidle": Wait for all network activity (slowest, most reliable) ✅

Step 5: Ensure Print Fidelity

```python
await page.add_style_tag(
    content="""
        html, body, * {
            -webkit-print-color-adjust: exact !important;
            print-color-adjust: exact !important;
        }
    """
)
```

What this does:

Forces browsers to preserve colors when printing
Without this: Backgrounds might be white in PDF
!important: Overrides any existing styles

Step 6: Wait for Resources

```python
await page.wait_for_load_state("networkidle")
await asyncio.sleep(1)  # Extra safety margin
```

Why double wait?

wait_for_load_state: Ensures network quiet
asyncio.sleep(1): Extra time for:

Font rendering to complete
JavaScript animations to settle
Layout recalculations to finish



Step 7: Generate PDF

```python
pdf_bytes = await page.pdf(
    path=None,  # Return bytes (not save to file)
    format="A4",
    landscape=True,
    print_background=True,
    prefer_css_page_size=True,
    margin={
        "top": "0.5in",
        "bottom": "0.5in",
        "left": "0.5in",
        "right": "0.5in",
    },
)
```

PDF options explained:

```python
path=None
# None = return bytes
# "file.pdf" = save to disk

format="A4"
# Paper size: A4, Letter, Legal, Tabloid
# Dimensions: 8.27 × 11.69 inches

landscape=True
# Orientation: horizontal
# False = portrait (vertical)

print_background=True
# Include CSS backgrounds and colors
# False = white backgrounds (printer-friendly)

prefer_css_page_size=True
# Respect @page CSS rules
# Example:
# @page { size: A4 landscape; margin: 0; }

margin={...}
# Physical margins on paper
# Can use: "in", "cm", "mm", "px"
```

Step 8: Cleanup Resources

```python
await page.close()
await asyncio.sleep(0.3)

await context.close()
await asyncio.sleep(0.3)

await browser.close()
await asyncio.sleep(0.5)
```

**Why delays between cleanup?**
- Ensures Chrome fully releases resources
- Prevents "connection refused" errors on next request
- Small performance cost for reliability ✅

### Resource Lifecycle

```
┌─────────────────────────────────────────┐
│  async with async_playwright() as p:    │  Playwright connects
│    │                                     │
│    ├─> browser = p.chromium.launch()    │  Chrome starts
│    │     │                               │
│    │     ├─> context = new_context()    │  Isolated session
│    │     │     │                         │
│    │     │     ├─> page = new_page()    │  Tab created
│    │     │     │     │                   │
│    │     │     │     ├─> page.pdf()     │  Generate PDF
│    │     │     │     │                   │
│    │     │     │     └─> page.close()   │  Close tab
│    │     │     │                         │
│    │     │     └─> context.close()      │  End session
│    │     │                               │
│    │     └─> browser.close()            │  Quit Chrome
│    │                                     │
│    └─> Playwright disconnects           │  Cleanup
└─────────────────────────────────────────┘
```

## Security Best Practices

1. Proper Certificate Validation

```python
# ✅ SECURE (Production)
context = await browser.new_context(
    ignore_https_errors=False
)

# ❌ INSECURE (Never use in production)
context = await browser.new_context(
    ignore_https_errors=True
)
```

**Why validation matters:**

```
With validation (ignore_https_errors=False):
├─> Only trusted certificates accepted
├─> Man-in-the-middle attacks prevented
├─> Expired certificates rejected
└─> Security maintained ✅

Without validation (ignore_https_errors=True):
├─> ANY certificate accepted
├─> Man-in-the-middle attacks possible
├─> Expired certificates accepted
└─> Security compromised ❌
```

2. Process Isolation

```python
# ✅ SECURE (Each request isolated)
args=["--single-process"]

# ❌ RISKY (Process reuse)
args=[]  # Default: multi-process with reuse
```

Security benefit:

User A's data can't leak to User B
Certificate validation state is fresh
Memory isolation between requests

3. Resource Limits

```python
# Set timeout for PDF generation
try:
    pdf_bytes = await asyncio.wait_for(
        page.pdf(...),
        timeout=30.0  # 30 second limit
    )
except asyncio.TimeoutError:
    # Handle timeout gracefully
```

Why timeouts matter:

Prevents infinite loops in HTML/JavaScript
Protects against denial-of-service
Ensures predictable resource usage

4. Input Validation

```python
async def generate_pdf(request: PDFRequest):
    # Validate HTML size
    if len(request.htmlContent) > 10_000_000:  # 10MB
        raise HTTPException(400, "HTML too large")
    
    # Validate options
    allowed_formats = ["A4", "Letter", "Legal"]
    if request.options.get("format") not in allowed_formats:
        raise HTTPException(400, "Invalid format")
```

5. Memory Management

```python
# Always cleanup
finally:
    page = None
    context = None
    browser = None
    gc.collect()  # Force garbage collection
```

Memory leak prevention:

Clear all references
Force garbage collection
Monitor memory usage in production

6. Logging Without Secrets

```python
# ✅ SAFE
print(f"Generated PDF: {len(pdf_bytes)} bytes")
print(f"Project ID: {request.projectId}")

# ❌ DANGEROUS
print(f"HTML content: {request.htmlContent}")  # May contain PII
print(f"S3 Key: {AWS_SECRET_ACCESS_KEY}")     # Exposes secrets
```

## Production Deployment

Kubernetes Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pdf-service
spec:
  replicas: 3  # 3 pods for high availability
  selector:
    matchLabels:
      app: pdf-service
  template:
    metadata:
      labels:
        app: pdf-service
    spec:
      containers:
      - name: pdf-service
        image: pdf-service:1.0.0
        ports:
        - containerPort: 8004
        
        # Resource limits (CRITICAL)
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        
        # Health checks
        livenessProbe:
          httpGet:
            path: /health
            port: 8004
          initialDelaySeconds: 30
          periodSeconds: 10
        
        readinessProbe:
          httpGet:
            path: /health
            port: 8004
          initialDelaySeconds: 10
          periodSeconds: 5
        
        # Environment variables
        env:
        - name: YOTTA_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: s3-credentials
              key: access-key
        - name: YOTTA_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: s3-credentials
              key: secret-key
        
        # Mount custom CA certificate
        volumeMounts:
        - name: ca-cert
          mountPath: /usr/local/share/ca-certificates/yotta-ca.crt
          subPath: yotta-ca.crt
      
      volumes:
      - name: ca-cert
        configMap:
          name: yotta-ca-cert
```

Monitoring and Observability

```python
import time
from prometheus_client import Counter, Histogram

# Metrics
pdf_generation_total = Counter(
    'pdf_generation_total', 
    'Total PDF generations',
    ['status']
)

pdf_generation_duration = Histogram(
    'pdf_generation_duration_seconds',
    'PDF generation duration'
)

async def convert_to_pdf(self, html_content: str, ...) -> bytes:
    start_time = time.time()
    
    try:
        # ... PDF generation code ...
        
        pdf_generation_total.labels(status='success').inc()
        return pdf_bytes
        
    except Exception as e:
        pdf_generation_total.labels(status='failure').inc()
        raise
        
    finally:
        duration = time.time() - start_time
        pdf_generation_duration.observe(duration)
```

### Scaling Considerations

**Horizontal Scaling:**

```
1 pod  = ~10 PDFs/minute
3 pods = ~30 PDFs/minute
10 pods = ~100 PDFs/minute
```

**Vertical Scaling:**

```
Memory: 1GB = handle 5 concurrent PDFs
Memory: 2GB = handle 10 concurrent PDFs
Memory: 4GB = handle 20 concurrent PDFs
```

**Resource Formula:**

```
pods_needed = (pdfs_per_minute / 10) * 1.5  # 1.5x safety factor
memory_per_pod = (concurrent_pdfs / 5) * 1GB
```

Performance Optimization

```python
# 1. Reuse Playwright instance (advanced)
_playwright_instance = None

async def get_playwright():
    global _playwright_instance
    if _playwright_instance is None:
        _playwright_instance = await async_playwright().start()
    return _playwright_instance

# 2. Connection pooling for S3
s3_client = boto3.client(
    's3',
    config=Config(
        max_pool_connections=50  # Allow 50 concurrent uploads
    )
)
```

3. Async S3 uploads

```python
import aioboto3
async def upload_pdf_async(pdf_bytes: bytes, s3_key: str):
session = aioboto3.Session()
async with session.client('s3') as s3:
await s3.put_object(
Bucket=S3_BUCKET,
Key=s3_key,
Body=pdf_bytes
)
```

---

## Conclusion

This guide covered:

1. ✅ Playwright architecture and internals
2. ✅ SSL certificate management (3-layer approach)
3. ✅ Docker environment configuration
4. ✅ Production-grade Python implementation
5. ✅ Security best practices
6. ✅ Deployment and scaling strategies

**Key Takeaways:**

- Playwright uses real Chrome = perfect rendering
- SSL requires system certs + NSS database + environment variables
- Process isolation (`--single-process`) prevents state leakage
- Proper cleanup prevents memory leaks
- Security through certificate validation, not bypassing

---
