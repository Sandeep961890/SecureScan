SecureScan — Basic Web Vulnerability Scanner
An educational Python web vulnerability scanner with a Flask dashboard. It is intended for learning basic web-security concepts and should only be used on systems you own or are explicitly authorized to test.
Authorization notice: Do not scan third-party websites without permission. Use the local test page below to verify your setup.

Features
The original scanner is documented as supporting:
- Crawling up to 10 pages on the same domain
- HTML form detection
- Basic reflected XSS checks
- Error-based SQL injection probes
- Open redirect parameter checks
- Security-header checks
- Basic information-disclosure checks
- robots.txt checks
- Checks for directory listing on a small set of common paths
- HTML report output at reports/scan_report.html
This is a heuristic learning tool, not a replacement for professional security tools. It may produce false positives and false negatives.
Requirements
- Windows
- Python 3.8 or newer
- Project dependencies from requirements.txt
Check Python in PowerShell
Open PowerShell and run:
python --version
Get-Command python -ErrorAction SilentlyContinue
If python is not found, install Python from the official Python website and enable Add Python to PATH during installation. Close and reopen PowerShell after installation. These instructions use python rather than the Windows py launcher.
Install dependencies
Open PowerShell in the project folder:
Set-Location 'S:\basic-web-vulnerability-scanner-main'
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
If the project uses a virtual environment, activate it first. To create one if needed:
Set-Location 'S:\basic-web-vulnerability-scanner-main'
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
If PowerShell blocks virtual-environment activation, you can either use the project's existing environment or run Python from .venv\Scripts\python.exe directly. Avoid changing system-wide execution policy just for this project unless you understand the implications.
Run the original command-line scanner
The original scanner is documented as scanner.py:
Set-Location 'S:\basic-web-vulnerability-scanner-main'
python scanner.py http://127.0.0.1:8000/
Only run this after the local test server below is running and after confirming that this target is authorized. The scanner should print output in the terminal and, if implemented as documented, save its report to:
reports/scan_report.html
Open that HTML file in a browser to view the report.
Run the SecureScan Flask dashboard
The dashboard is documented as running at http://127.0.0.1:5000/. The server uses SECURESCAN_ALLOWED_HOSTS to restrict which targets can be scanned. Set environment variables in the same PowerShell window that starts Flask.
Option A — Scan a public host you are authorized to test
Replace staging.yourdomain.com with a hostname you own or have explicit permission to assess:
Set-Location 'S:\basic-web-vulnerability-scanner-main'
$env:SECURESCAN_ALLOWED_HOSTS = 'staging.yourdomain.com'
python app.py
Open this in your browser:
http://127.0.0.1:5000/
The target entered in the dashboard must match the server allowlist and the dashboard may ask you to confirm authorization.

Allowlist rules documented by this project:
- Entries are comma-separated.
- A hostname without a port allows only the default HTTP/HTTPS ports (80/443).
- A non-standard port must be included explicitly, such as staging.yourdomain.com:8443.
- A wildcard such as *.staging.yourdomain.com allows subdomains at label boundaries but not the apex hostname.
- Private and reserved IP addresses are blocked by default.
Option B — Test with a harmless local page you control
Use this option for local testing. It requires explicit loopback opt-in in the scanner. The code must support the documented SECURESCAN_ALLOW_LOOPBACK_TARGETS setting and exact IP:port allowlisting; if it does not, inspect and fix the validation code rather than bypassing it.

PowerShell window 1 — start the local test page
$testSite = Join-Path $env:TEMP 'securescan-local-test'
New-Item -ItemType Directory -Force -Path $testSite | Out-Null
'<!doctype html><html><body>SecureScan local test page</body></html>' |
    Set-Content -Encoding utf8 (Join-Path $testSite 'index.html')
Set-Location $testSite
python -m http.server 8000 --bind 127.0.0.1
Keep this window open. The test page should be available at:
http://127.0.0.1:8000/
PowerShell window 2 — start the SecureScan dashboard
Set-Location 'S:\basic-web-vulnerability-scanner-main'
$env:SECURESCAN_ALLOWED_HOSTS = '127.0.0.1:8000'
$env:SECURESCAN_ALLOW_LOOPBACK_TARGETS = '1'
python app.py
Keep this window open. In the browser, visit http://127.0.0.1:5000/, enter http://127.0.0.1:8000/ as the target, and confirm authorization if prompted.

Important: Do not enter http://127.0.0.1:5000/ as the target. Port 5000 is the SecureScan dashboard; port 8000 is the separate test page. If the request is rejected, the app's allowlist and loopback validation may use different rules than documented. Read the actual validation code and logs; do not disable the allowlist.

Stop both servers with Ctrl+C when finished.
Run automated tests
If pytest is installed and the project includes tests, run:
Set-Location 'S:\basic-web-vulnerability-scanner-main'
python -m pytest -q
If pytest is not installed and is listed as a development dependency, install it in the active environment:
python -m pip install pytest
python -m pytest -q
Do not treat an empty test suite as proof that the application works. Record actual pass/fail results.
Troubleshooting
python is not recognized
Check:
Get-Command python -ErrorAction SilentlyContinue
python --version
Install Python or correct PATH, then reopen PowerShell.
No module named flask or another missing module
Activate the correct environment and run:
python -m pip install -r requirements.txt
If Flask is imported by app.py, make sure Flask is listed in requirements.txt.
“This hostname and port are not in the server's authorized-host allowlist”
- Stop Flask with Ctrl+C.
- Set the environment variables in the same PowerShell window that launches Flask.
- Ensure the submitted URL matches the exact host and port in the allowlist.
- For the local example, use http://127.0.0.1:8000/, not port 5000.
- Confirm the application explicitly supports local loopback opt-in.
- Restart Flask after changing environment variables.
Scan returns no findings
A clean response from a simple HTML test page is expected. An empty findings list means only that the scanner's configured checks returned no findings; it does not prove that a website is secure.
Report file
The documented report path is reports/scan_report.html. The README indicates the dashboard does not expose a report-download API endpoint, so open the generated report locally unless the implementation has since been updated.
API behavior documented for the dashboard
The dashboard sends a POST request to /api/scan with a JSON body similar to:
{
  "url": "https://scan.example.org",
  "authorized": true
}
The response is normalized from the scanner's findings. Invalid input and disallowed targets return JSON errors; overlapping scans may return HTTP 409. The API is synchronous and does not provide live progress updates.
Limitations
- Detects only simple reflected XSS patterns.
- SQL injection checks rely on database error messages and can miss blind/time-based cases.
- Does not render JavaScript in a browser engine.
- Uses a limited set of test payloads.
- May produce false positives and false negatives.
- Severity and CVSS values in the original HTML report are fixed values, not contextual CVSS calculations.
- Not a substitute for professional security testing.