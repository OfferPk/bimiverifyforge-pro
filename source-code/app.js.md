/\*\*  
 \* BIMIVerifyForge Pro \- Web Application Orchestrator  
 \*/

(function () {  
  'use strict';

  const PRESETS \= {  
    apex: {  
      name: "Apex FinBank Core BIMI & VMC Certificate (Tier-1 Zero-Trust)",  
      domain: "apexfinbank.com",  
      dmarcPolicy: {  
        p: "reject",  
        sp: "reject",  
        pct: 100  
      },  
      bimiConfig: {  
        selector: "default",  
        logoUrl: "https\://apexfinbank.com/bimi/apex-logo.svg",  
        vmcUrl: "https\://apexfinbank.com/bimi/apex-vmc.pem",  
        vmcCertificate: {  
          issuer: "DigiCert Verified Mark RSA4096 SHA256 2026 CA1",  
          subject: "Apex Financial Services Corp",  
          trademarkOffice: "US Patent and Trademark Office",  
          trademarkRegNumber: "US PTO \#88912345",  
          validFrom: "2026-01-01T00:00:00Z",  
          validTo: "2027-01-01T00:00:00Z"  
        }  
      },  
      rawSvg: \`\<svg xmlns="http\://www\.w3.org/2000/svg" version="1.2" baseProfile="tiny-ps" viewBox="0 0 500 500" width="500" height="500"\>  
  \<title\>Apex Financial\</title\>  
  \<circle cx="250" cy="250" r="240" fill="\#0c1126" stroke="\#6366f1" stroke-width="12"/\>  
  \<polygon points="250,70 390,380 110,380" fill="\#06b6d4"/\>  
  \<polygon points="250,150 340,350 160,350" fill="\#0c1126"/\>  
\</svg\>\`  
    },  
    cloudpulse: {  
      name: "CloudPulse SaaS BIMI (Self-Hosted SVG & Yahoo/Fastmail Certified)",  
      domain: "cloudpulse.io",  
      dmarcPolicy: {  
        p: "quarantine",  
        sp: "quarantine",  
        pct: 100  
      },  
      bimiConfig: {  
        selector: "default",  
        logoUrl: "https\://cloudpulse.io/bimi/logo.svg",  
        vmcUrl: "",  
        vmcCertificate: null  
      },  
      rawSvg: \`\<svg xmlns="http\://www\.w3.org/2000/svg" version="1.2" baseProfile="tiny-ps" viewBox="0 0 512 512" width="512" height="512"\>  
  \<title\>CloudPulse\</title\>  
  \<rect width="512" height="512" rx="100" fill="\#0a0f24"/\>  
  \<path d="M120 280 L200 180 L280 340 L340 240 L400 300" fill="none" stroke="\#10b981" stroke-width="28" stroke-linecap="round" stroke-linejoin="round"/\>  
\</svg\>\`  
    },  
    legacy: {  
      name: "Legacy Corp Non-Compliant SVG (Forbidden Tags & DMARC p=none)",  
      domain: "legacycorp.com",  
      dmarcPolicy: {  
        p: "none",  
        sp: "none",  
        pct: 100  
      },  
      bimiConfig: {  
        selector: "default",  
        logoUrl: "http\://insecure-cdn.legacycorp.com/logo.svg",  
        vmcUrl: "",  
        vmcCertificate: null  
      },  
      rawSvg: \`\<svg xmlns="http\://www\.w3.org/2000/svg" xmlns:xlink="http\://www\.w3.org/1999/xlink" viewBox="0 0 800 400" width="800" height="400"\>  
  \<script type="text/javascript"\>alert('XSS');\</script\>  
  \<foreignObject width="100%" height="100%"\>  
    \<div xmlns="http\://www\.w3.org/1999/xhtml"\>Non compliant text\</div\>  
  \</foreignObject\>  
  \<image xlink:href="data:image/png;base64,iVBORw0KGgo=" width="100" height="100"/\>  
  \<rect width="800" height="400" fill="\#ff0000"/\>  
\</svg\>\`  
    }  
  };

  let currentParsedConfig \= null;  
  let activeSynthTarget \= 'cloudflare';  
  let activeSimScenario \= 'verified';

  // DOM Elements  
  const presetSelector \= document.getElementById('presetSelector');  
  const configInput \= document.getElementById('configInput');  
  const btnAudit \= document.getElementById('btnAudit');  
  const btnReset \= document.getElementById('btnReset');  
  const btnSanitize \= document.getElementById('btnSanitize');  
  const inspDomain \= document.getElementById('inspDomain');  
  const inspProfile \= document.getElementById('inspProfile');  
  const inspRatio \= document.getElementById('inspRatio');  
  const inspVmc \= document.getElementById('inspVmc');  
  const gradeBadge \= document.getElementById('gradeBadge');  
  const scoreText \= document.getElementById('scoreText');  
  const gradeSummary \= document.getElementById('gradeSummary');  
  const scoreSummaryTag \= document.getElementById('scoreSummaryTag');  
  const checksContainer \= document.getElementById('checksContainer');  
  const previewGmailAvatar \= document.getElementById('previewGmailAvatar');  
  const previewAppleAvatar \= document.getElementById('previewAppleAvatar');  
  const previewYahooAvatar \= document.getElementById('previewYahooAvatar');  
  const gmailVerifiedBadge \= document.getElementById('gmailVerifiedBadge');  
  const btnSimVerified \= document.getElementById('btnSimVerified');  
  const btnSimFallback \= document.getElementById('btnSimFallback');  
  const simUnhardenedSummary \= document.getElementById('simUnhardenedSummary');  
  const simHardenedSummary \= document.getElementById('simHardenedSummary');  
  const simUnhardenedTimeline \= document.getElementById('simUnhardenedTimeline');  
  const simHardenedTimeline \= document.getElementById('simHardenedTimeline');  
  const codeOutput \= document.getElementById('codeOutput');  
  const btnCopyCode \= document.getElementById('btnCopyCode');

  function loadPreset(key) {  
    const preset \= PRESETS\[key\] || PRESETS.apex;  
    configInput.value \= JSON.stringify(preset, null, 2);  
    runAnalysis();  
  }

  function runAnalysis() {  
    let parsed;  
    try {  
      parsed \= JSON.parse(configInput.value);  
      currentParsedConfig \= parsed;  
    } catch (e) {  
      alert('JSON Syntax Error: ' \+ e.message);  
      return;  
    }

    const result \= window.BIMIAuditor.auditBimiConfig(parsed);  
    const svgAudit \= result.svgAudit;  
    const bimi \= parsed.bimiConfig || {};  
    const dmarc \= parsed.dmarcPolicy || {};

    // Update Inspector Pills  
    inspDomain.textContent \= \`\${parsed.domain || 'domain.com'} (p=\${dmarc.p || 'none'})\`;  
    inspProfile.textContent \= svgAudit.hasVersion12 && svgAudit.hasTinyPs  
      ? 'Compliant (version 1.2)'  
      : 'Non-Compliant Profile';  
    inspProfile.style.color \= svgAudit.hasVersion12 && svgAudit.hasTinyPs ? 'var(--accent-emerald)' : 'var(--accent-rose)';

    inspRatio.textContent \= \`\${svgAudit.isSquare ? '1:1 Square' : 'Non-Square'} (\${(svgAudit.byteLength / 1024).toFixed(1)} KB)\`;  
    inspRatio.style.color \= svgAudit.isSquare ? 'var(--accent-emerald)' : 'var(--accent-rose)';

    const hasVmc \= Boolean(bimi.vmcCertificate && bimi.vmcUrl);  
    inspVmc.textContent \= hasVmc ? 'VMC Active (Gmail Ready)' : 'No VMC (Yahoo Only)';  
    inspVmc.style.color \= hasVmc ? 'var(--accent-cyan)' : 'var(--accent-amber)';

    // Update Avatar Previews  
    renderAvatarPreviews(parsed.rawSvg, result.summary);

    // Update Scorecard  
    scoreText.textContent \= \`\${result.overallScore} / 100 Points\`;  
    gradeBadge.textContent \= result.grade;  
    gradeBadge.className \= 'grade-circle';  
    if (result.grade \=== 'F' || result.grade \=== 'D') {  
      gradeBadge.classList.add('grade-f');  
    } else if (result.grade \=== 'A+' || result.grade \=== 'A') {  
      gradeBadge.classList.add('grade-a');  
    }

    scoreSummaryTag.textContent \= \`\${result.summary.passed} Passed • \${result.summary.warnings} Warnings • \${result.summary.failures} Failures\`;  
    gradeSummary.textContent \= \`\${result.grade \=== 'A+' ? 'Gmail Blue Checkmark Ready' : 'Audit Complete'} • \${result.summary.failures} Disqualifying Failures\`;

    // Render Checks  
    checksContainer.innerHTML \= '';  
    result.checks.forEach(function (c) {  
      const item \= document.createElement('div');  
      item.className \= 'check-item';

      const badgeClass \=  
        c.status \=== 'PASS'  
          ? 'badge-pass'  
          : c.status \=== 'WARN'  
          ? 'badge-warn'  
          : c.status \=== 'FAIL'  
          ? 'badge-fail'  
          : 'badge-info';

      item.innerHTML \= \`  
        \<div class="check-header"\>  
          \<span class="check-name"\>\${c.title}\</span\>  
          \<span class="badge \${badgeClass}"\>\${c.status} (\${c.score}/\${c.weight})\</span\>  
        \</div\>  
        \<div class="check-desc"\>\${c.details}\</div\>  
        \${c.remediation ? \`\<div class="check-remediation"\>Remediation: \${c.remediation}\</div\>\` : ''}  
      \`;  
      checksContainer.appendChild(item);  
    });

    renderSimulation();  
    renderSynthesizer();  
  }

  function renderAvatarPreviews(rawSvg, summary) {  
    if (\!summary.isSvgValid || \!summary.isDmarcEnforced) {  
      // Fallback: Gray circle with generic initial  
      const letter \= currentParsedConfig && currentParsedConfig.domain  
        ? currentParsedConfig.domain.charAt(0).toUpperCase()  
        : 'A';  
      const fallbackHtml \= \`\<div style="font-size: 26px; font-weight: 700; color: \#64748b;"\>\${letter}\</div\>\`;  
      previewGmailAvatar.innerHTML \= fallbackHtml;  
      previewAppleAvatar.innerHTML \= fallbackHtml;  
      previewYahooAvatar.innerHTML \= fallbackHtml;  
      gmailVerifiedBadge.style.display \= 'none';  
      return;  
    }

    // SVG is valid and DMARC enforced  
    const safeSvg \= rawSvg.replace(/\<script\[\\s\\S\]\*?\<\\/script\>/gi, '');  
    previewAppleAvatar.innerHTML \= safeSvg;  
    previewYahooAvatar.innerHTML \= safeSvg;

    if (summary.hasVmc) {  
      previewGmailAvatar.innerHTML \= safeSvg;  
      gmailVerifiedBadge.style.display \= 'flex';  
    } else {  
      const letter \= currentParsedConfig && currentParsedConfig.domain  
        ? currentParsedConfig.domain.charAt(0).toUpperCase()  
        : 'A';  
      previewGmailAvatar.innerHTML \= \`\<div style="font-size: 26px; font-weight: 700; color: \#64748b;"\>\${letter}\</div\>\`;  
      gmailVerifiedBadge.style.display \= 'none';  
    }  
  }

  function sanitizeSvgInput() {  
    if (\!currentParsedConfig) return;  
    const raw \= currentParsedConfig.rawSvg || '';  
    const brandName \= currentParsedConfig.name || 'Brand Logo';  
    const sanitized \= window.BIMIEngine.sanitizeToTinyPs(raw, brandName);  
    currentParsedConfig.rawSvg \= sanitized;  
    configInput.value \= JSON.stringify(currentParsedConfig, null, 2);  
    runAnalysis();  
  }

  function renderSimulation() {  
    if (activeSimScenario \=== 'verified') {  
      const unhardened \= window.BIMISimulator.simulateInboxExperience({ isDmarcEnforced: false, isSvgValid: false, hasVmc: false });  
      const hardened \= window.BIMISimulator.simulateInboxExperience({ isDmarcEnforced: true, isSvgValid: true, hasVmc: true });

      simUnhardenedSummary.textContent \= unhardened.verdict;  
      simHardenedSummary.textContent \= hardened.verdict;

      simUnhardenedTimeline.innerHTML \= unhardened.steps.map(s \=\> \`  
        \<div class="timeline-step \${s.isFailure ? 'step-outage' : ''}"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: \${s.isFailure ? 'var(--accent-rose)' : 'var(--accent-emerald)'};"\>  
            \${s.statusText}  
          \</div\>  
        \</div\>  
      \`).join('');

      simHardenedTimeline.innerHTML \= hardened.steps.map(s \=\> \`  
        \<div class="timeline-step"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: var(--accent-emerald);"\>  
            \${s.statusText}  
          \</div\>  
        \</div\>  
      \`).join('');  
    } else {  
      const unhardened \= window.BIMISimulator.simulateInboxExperience({ isDmarcEnforced: true, isSvgValid: false, hasVmc: false });  
      const hardened \= window.BIMISimulator.simulateInboxExperience({ isDmarcEnforced: true, isSvgValid: true, hasVmc: false });

      simUnhardenedSummary.textContent \= unhardened.verdict;  
      simHardenedSummary.textContent \= hardened.verdict;

      simUnhardenedTimeline.innerHTML \= unhardened.steps.map(s \=\> \`  
        \<div class="timeline-step \${s.isFailure ? 'step-outage' : ''}"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: \${s.isFailure ? 'var(--accent-rose)' : 'var(--accent-emerald)'};"\>  
            \${s.statusText}  
          \</div\>  
        \</div\>  
      \`).join('');

      simHardenedTimeline.innerHTML \= hardened.steps.map(s \=\> \`  
        \<div class="timeline-step"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: var(--accent-emerald);"\>  
            \${s.statusText}  
          \</div\>  
        \</div\>  
      \`).join('');  
    }  
  }

  function renderSynthesizer() {  
    if (\!currentParsedConfig) return;  
    const synth \= window.BIMIDNSSynthesizer;  
    const domain \= currentParsedConfig.domain || 'yourcompany.com';  
    const bimi \= currentParsedConfig.bimiConfig || {};  
    let code \= '';  
    if (activeSynthTarget \=== 'cloudflare') {  
      code \= synth.generateCloudflareBimi(domain, bimi.logoUrl, bimi.vmcUrl);  
    } else if (activeSynthTarget \=== 'route53') {  
      code \= synth.generateRoute53BimiJson(domain, bimi.logoUrl, bimi.vmcUrl);  
    } else if (activeSynthTarget \=== 'bind') {  
      code \= synth.generateBindBimiZonefile(domain, bimi.logoUrl, bimi.vmcUrl);  
    } else if (activeSynthTarget \=== 'terraform') {  
      code \= synth.generateTerraformBimiHcl(domain, bimi.logoUrl, bimi.vmcUrl);  
    }  
    codeOutput.textContent \= code;  
  }

  // Event Listeners  
  presetSelector.addEventListener('change', function () {  
    loadPreset(this.value);  
  });

  btnAudit.addEventListener('click', runAnalysis);

  btnReset.addEventListener('click', function () {  
    loadPreset(presetSelector.value);  
  });

  btnSanitize.addEventListener('click', sanitizeSvgInput);

  btnSimVerified.addEventListener('click', function () {  
    activeSimScenario \= 'verified';  
    btnSimVerified.classList.add('active');  
    btnSimFallback.classList.remove('active');  
    renderSimulation();  
  });

  btnSimFallback.addEventListener('click', function () {  
    activeSimScenario \= 'fallback';  
    btnSimFallback.classList.add('active');  
    btnSimVerified.classList.remove('active');  
    renderSimulation();  
  });

  document.querySelectorAll('.synth-tabs button').forEach(function (btn) {  
    btn.addEventListener('click', function () {  
      document.querySelectorAll('.synth-tabs button').forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeSynthTarget \= btn.getAttribute('data-target');  
      renderSynthesizer();  
    });  
  });

  btnCopyCode.addEventListener('click', function () {  
    navigator.clipboard.writeText(codeOutput.textContent).then(function () {  
      const originalText \= btnCopyCode.textContent;  
      btnCopyCode.textContent \= 'Copied\!';  
      setTimeout(function () {  
        btnCopyCode.textContent \= originalText;  
      }, 1800);  
    });  
  });

  // Initial Load  
  loadPreset('apex');  
})();  
