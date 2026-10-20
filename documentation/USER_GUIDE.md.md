\# BIMIVerifyForge Pro: User Guide & NetDevOps / BrandOps Integration

\#\# Overview  
BIMIVerifyForge Pro operates both as a 100% client-side web application and as a standalone Python CLI tool to sanitize standard vector SVGs into SVG Tiny-PS format, audit BIMI readiness, simulate mailbox inbox presentations, and generate production DNS records.

\---

\#\# 1\. Web Studio Quickstart  
1\. Open \`source-code/index.html\` in any modern web browser.  
2\. Select an architecture preset:  
   \- \*\*Apex FinBank\*\*: Full VMC certificate with Gmail Blue Checkmark readiness (Grade A+).  
   \- \*\*CloudPulse SaaS\*\*: Self-hosted SVG for Yahoo\! & Fastmail certification (Grade A).  
   \- \*\*Legacy Corp\*\*: Non-compliant SVG with DMARC \`p=none\` (Grade F).  
3\. If using your own logo, paste the raw SVG into the editor and click \*\*1-Click Sanitize\*\* to automatically format it to SVG Tiny-PS standards.  
4\. Click \*\*Audit BIMI\*\* to evaluate the 10-point scorecard.  
5\. Review the \*\*Live Avatar Previews\*\* to inspect how your brand appears across Gmail, Apple Mail, and Yahoo\! Mail.  
6\. Select your DNS provider (\*\*Cloudflare\*\*, \*\*AWS Route 53\*\*, \*\*BIND Zonefile\*\*, or \*\*Terraform HCL\*\*) and click \*\*Copy DNS Records\*\*.

\---

\#\# 2\. Standalone Python CLI Tool

Requires Python 3.8+ with zero external package dependencies.

\#\#\# Command Reference

\`\`\`bash  
\# Audit a BIMI configuration preset  
python3 build/bimiverify-cli.py audit \-p presets/enterprise-bank-apex-vmc.json

\# Audit with strict CI/CD gating (exits 1 on failures or score \< 90\)  
python3 build/bimiverify-cli.py audit \-p presets/enterprise-bank-apex-vmc.json \--strict

\# Sanitize a dirty/standard SVG file into strict SVG Tiny-PS  
python3 build/bimiverify-cli.py sanitize \-i raw-logo.svg \-o logo-tiny-ps.svg \-b "Your Brand"  
\`\`\`

\---

\#\# 3\. NetDevOps / BrandOps CI/CD Pipeline Integration

Integrate BIMIVerifyForge Pro into design and domain deployment pipelines to prevent broken brand logos from reaching production:

\`\`\`yaml  
\# .github/workflows/bimi-audit.yml  
name: BIMI & Brand Asset Verification  
on: \[push, pull\_request\]

jobs:  
  bimi-verify:  
    runs-on: ubuntu-latest  
    steps:  
      \- uses: actions/checkout@v4  
      \- name: Set up Python  
        uses: actions/setup-python@v5  
        with:  
          python-version: '3.11'  
      \- name: Run BIMIVerifyForge Strict Audit  
        run: |  
          python3 build/bimiverify-cli.py audit \\  
            \-p presets/enterprise-bank-apex-vmc.json \\  
            \--strict  
\`\`\`  
