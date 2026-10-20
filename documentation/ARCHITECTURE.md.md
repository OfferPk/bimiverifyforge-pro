\# BIMIVerifyForge Pro: Architectural Specification & Brand Assertion Modeling

\#\# Executive Summary  
Brand Indicators for Message Identification (BIMI \- RFC 8463\) enables email-sending organizations that have deployed strong DMARC (RFC 7489\) protection to display their official verified brand logotypes alongside authenticated outbound messages in supporting email client applications (Google Gmail, Apple Mail on iOS/macOS, Yahoo\! Mail, and Fastmail).

In Gmail, qualifying BIMI senders with an active Verified Mark Certificate (VMC) are awarded the coveted \*\*blue verified checkmark\*\*, increasing email open rates by 21%–39% and establishing instant sender authenticity.

However, over 90% of BIMI deployment attempts fail or are rejected by mailbox providers:  
1\. \*\*The SVG Tiny-PS Profile Failure\*\*: Standard web SVGs exported from Figma, Illustrator, or Sketch fail mailbox security parsers. BIMI requires strict adherence to the SVG Tiny 1.2 Portable/Secure (Tiny-PS) profile, mandating a square 1:1 aspect ratio, \`version="1.2"\`, \`baseProfile="tiny-ps"\`, \`\<title\>\` tags, zero scripts, and zero raster \`\<image\>\` tags.  
2\. \*\*DMARC Enforcement Prerequisite\*\*: Mailbox providers discard BIMI assertions if the sending domain lacks an enforced DMARC policy (\`p=reject\` or \`p=quarantine; pct=100;\`).  
3\. \*\*VMC Certificate Chain Complexity\*\*: Integrating X.509 Verified Mark Certificates (issued by DigiCert or Entrust) with embedded logotypes (\`id-pe-logotype\` / RFC 3709\) and active trademark serial numbers requires rigorous cryptographic validation.

\*\*BIMIVerifyForge Pro\*\* provides an offline SVG Tiny-PS sanitizer, VMC validator, 10-point Zero-Trust auditor, inbox avatar simulator, and multi-provider DNS synthesizer (\*\*Cloudflare\*\*, \*\*AWS Route 53\*\*, \*\*BIND\*\*, \*\*Terraform\*\*).

\---

\#\# Technical Fundamentals: BIMI & SVG Tiny-PS Profile

\#\#\# 1\. SVG Tiny-PS Profile Constraints (AuthIndicators Specification)  
\- \`version="1.2"\` and \`baseProfile="tiny-ps"\`.  
\- Must have a square 1:1 aspect ratio (\`viewBox="0 0 N N"\` where width \== height).  
\- Strictly forbidden elements: \`\<script\>\`, \`\<style\>\`, \`\<foreignObject\>\`, \`\<iframe\>\`, \`\<audio\>\`, \`\<video\>\`, \`\<animate\>\`, and \`\<image\>\`.  
\- Strictly forbidden attributes: Event handlers (\`onclick\`, \`onload\`, \`onerror\`), external \`xlink:href\` URLs.  
\- Mandatory \`\<title\>\` tag naming the brand.  
\- File size must be \`\<= 32 KB\`.

\#\#\# 2\. Verified Mark Certificate (VMC) Architecture  
\- X.509 certificate issued by an approved CA (DigiCert or Entrust).  
\- Binds a registered trademark logotype (verified with national trademark offices like USPTO, EUIPO, or UK IPO) directly to the domain.  
\- Embedded as an SVG gz/base64 payload inside the \`id-pe-logotype\` certificate extension (RFC 3709).  
\- Mandatory for Google Gmail (blue checkmark) and Apple Mail.

\---

\#\# 10-Point Zero-Trust BIMI Scorecard Matrix

| ID | Check Title | Weight | Criteria |  
|---|---|---|---|  
| 1 | \*\*SVG Tiny-PS Profile Conformance\*\* | 15 pts | Root \`\<svg\>\` declares \`version="1.2"\` and \`baseProfile="tiny-ps"\`. |  
| 2 | \*\*Square 1:1 Aspect Ratio\*\* | 15 pts | \`viewBox\` width equals height for clean avatar clipping. |  
| 3 | \*\*Zero Forbidden Elements\*\* | 10 pts | Zero scripts, foreignObjects, or embedded raster images. |  
| 4 | \*\*Mandatory \`\<title\>\` Brand Tag\*\* | 10 pts | Accessible \`\<title\>\` tag naming brand present inside \`\<svg\>\`. |  
| 5 | \*\*DMARC Enforcement Prerequisite\*\* | 10 pts | Enforced DMARC policy (\`p=reject\` or \`p=quarantine; pct=100;\`). |  
| 6 | \*\*Verified Mark Certificate (VMC)\*\* | 10 pts | Valid VMC from DigiCert/Entrust with active trademark. |  
| 7 | \*\*SVG File Size Optimization\*\* | 10 pts | SVG payload \`\<= 32 KB\` threshold. |  
| 8 | \*\*HTTPS Asset Hosting Security\*\* | 10 pts | Assets hosted exclusively over secure TLS endpoints. |  
| 9 | \*\*Multi-Mailbox Compatibility\*\* | 5 pts | Certified across Gmail, Apple Mail, Yahoo\!, and Fastmail. |  
| 10| \*\*BIMI DNS Record Formatting\*\* | 5 pts | Valid \`default.\_bimi\` TXT record with \`v=BIMI1; l=...; a=...;\`. |

\---

\#\# Compliance Tiers  
\- \*\*Grade A+ (95–100%)\*\*: Zero-Trust Certified, VMC Verified, Gmail Blue Checkmark Ready.  
\- \*\*Grade A (85–94%)\*\*: Standard BIMI Certified (Yahoo/Fastmail Ready; VMC optional).  
\- \*\*Grade B (75–84%)\*\*: Compliant with Minor SVG Optimization Suggestions.  
\- \*\*Grade C (65–74%)\*\*: Elevated Rejection Risk.  
\- \*\*Grade F (\< 60%)\*\*: Critical Failure (DMARC \`p=none\` or Dirty Non-Compliant SVG).  
