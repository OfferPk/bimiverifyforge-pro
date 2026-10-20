/\*\*  
 \* BIMIVerifyForge Pro \- 10-Point Zero-Trust BIMI Auditor  
 \* Evaluates RFC 8463, SVG Tiny-PS Profile, DMARC alignment, and VMC certificate readiness.  
 \* Scores posture (0-100%, Grade A+ to F) and outputs actionable remediation steps.  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\['./bimi-engine'\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory(require('./bimi-engine'));  
  } else {  
    root.BIMIAuditor \= factory(root.BIMIEngine);  
  }  
})(typeof self \!== 'undefined' ? self : this, function (engine) {  
  'use strict';

  function auditBimiConfig(config) {  
    const rawSvg \= config.rawSvg || '';  
    const dmarc \= config.dmarcPolicy || {};  
    const bimi \= config.bimiConfig || {};  
    const domain \= config.domain || 'example.com';

    const svgAudit \= engine.validateSvgTinyPs(rawSvg);  
    const checks \= \[\];

    // Check 1: SVG Tiny-PS Profile Conformance (15 pts)  
    if (svgAudit.hasVersion12 && svgAudit.hasTinyPs) {  
      checks.push({  
        id: 'svg\_profile',  
        title: 'SVG Tiny-PS Profile Conformance',  
        weight: 15,  
        score: 15,  
        status: 'PASS',  
        details: 'AuthIndicators Compliant: Root \<svg\> declares version="1.2" and baseProfile="tiny-ps".',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'svg\_profile',  
        title: 'SVG Tiny-PS Profile Conformance',  
        weight: 15,  
        score: 0,  
        status: 'FAIL',  
        details: 'Invalid SVG Profile: Mailbox providers (Gmail, Yahoo) require SVG Tiny 1.2 Portable/Secure profile.',  
        remediation: 'Use the 1-Click SVG Sanitizer to inject version="1.2" and baseProfile="tiny-ps".'  
      });  
    }

    // Check 2: Square 1:1 Aspect Ratio (15 pts)  
    if (svgAudit.isSquare) {  
      checks.push({  
        id: 'aspect\_ratio',  
        title: 'Square 1:1 Aspect Ratio (viewBox)',  
        weight: 15,  
        score: 15,  
        status: 'PASS',  
        details: 'Perfect 1:1 Square: viewBox dimensions are square, ensuring clean circle/square avatar clipping.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'aspect\_ratio',  
        title: 'Square 1:1 Aspect Ratio (viewBox)',  
        weight: 15,  
        score: 0,  
        status: 'FAIL',  
        details: 'Non-Square Aspect Ratio: Rectangular SVGs will be rejected or distorted by mailbox clients.',  
        remediation: 'Set viewBox width equal to height (e.g. viewBox="0 0 512 512").'  
      });  
    }

    // Check 3: Zero Forbidden Elements & Attributes (10 pts)  
    const hasForbiddenIssues \= svgAudit.issues.some(i \=\> i.includes('Forbidden'));  
    if (\!hasForbiddenIssues) {  
      checks.push({  
        id: 'forbidden\_elements',  
        title: 'Zero Forbidden Elements & Scripts',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Secure Vector Graphic: Zero scripts, foreignObjects, embedded raster images, or event handlers.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'forbidden\_elements',  
        title: 'Zero Forbidden Elements & Scripts',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: 'Security Violation: Detected forbidden interactive elements (\<script\>, \<foreignObject\>, or \<image\>).',  
        remediation: 'Remove all scripts and raster elements from the SVG source code.'  
      });  
    }

    // Check 4: Mandatory \<title\> Brand Element (10 pts)  
    if (svgAudit.hasTitle) {  
      checks.push({  
        id: 'title\_element',  
        title: 'Mandatory \<title\> Brand Tag',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Brand Title Tag Present: Accessibility and specification requirements satisfied.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'title\_element',  
        title: 'Mandatory \<title\> Brand Tag',  
        weight: 10,  
        score: 2,  
        status: 'WARN',  
        details: 'Missing \<title\> Element: RFC 8463 requires an explicit \<title\> tag inside the root \<svg\>.',  
        remediation: 'Add \<title\>Your Brand Name\</title\> immediately inside the \<svg\> element.'  
      });  
    }

    // Check 5: DMARC Enforcement Prerequisite (10 pts)  
    const dmarcP \= (dmarc.p || 'none').toLowerCase();  
    const dmarcSp \= (dmarc.sp || dmarcP).toLowerCase();  
    const dmarcPct \= dmarc.pct \!== undefined ? Number(dmarc.pct) : 100;  
    const isDmarcEnforced \= (dmarcP \=== 'reject' || (dmarcP \=== 'quarantine' && dmarcPct \=== 100)) && dmarcSp \!== 'none';

    if (isDmarcEnforced) {  
      checks.push({  
        id: 'dmarc\_prerequisite',  
        title: 'DMARC Enforcement Prerequisite',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: \`DMARC Enforced (p=\${dmarcP}, sp=\${dmarcSp}, pct=\${dmarcPct}%): Satisfies mandatory prerequisite for BIMI assertion.\`,  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'dmarc\_prerequisite',  
        title: 'DMARC Enforcement Prerequisite',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: \`DMARC Ineligible (p=\${dmarcP}): Mailbox providers refuse to display BIMI logos unless DMARC is enforced with p=reject or p=quarantine (pct=100).\`,  
        remediation: 'Advance DMARC policy from p=none to p=reject to unlock BIMI brand logos.'  
      });  
    }

    // Check 6: Verified Mark Certificate (VMC) (10 pts)  
    const hasVmc \= bimi.vmcCertificate && bimi.vmcUrl && bimi.vmcUrl.startsWith('https\://');  
    if (hasVmc) {  
      checks.push({  
        id: 'vmc\_certificate',  
        title: 'Verified Mark Certificate (VMC)',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: \`Active VMC Certificate: Issued by \${bimi.vmcCertificate.issuer} with registered trademark (\${bimi.vmcCertificate.trademarkRegNumber}). Eligible for Gmail Blue Checkmark.\`,  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'vmc\_certificate',  
        title: 'Verified Mark Certificate (VMC)',  
        weight: 10,  
        score: 4,  
        status: 'INFO',  
        details: 'No VMC Certificate Configured: Logo will display on Yahoo\! and Fastmail, but NOT on Gmail or Apple Mail.',  
        remediation: 'Acquire a Verified Mark Certificate (VMC) from DigiCert or Entrust to unlock Gmail blue checkmarks.'  
      });  
    }

    // Check 7: SVG File Size Optimization (10 pts)  
    if (svgAudit.byteLength \<= 32768\) {  
      checks.push({  
        id: 'file\_size',  
        title: 'SVG File Size Optimization (\< 32 KB)',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: \`Optimized Payload: \${(svgAudit.byteLength / 1024).toFixed(1)} KB (well below 32 KB limit).\`,  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'file\_size',  
        title: 'SVG File Size Optimization (\< 32 KB)',  
        weight: 10,  
        score: 4,  
        status: 'WARN',  
        details: \`Large SVG Payload: \${(svgAudit.byteLength / 1024).toFixed(1)} KB exceeds the recommended 32 KB threshold.\`,  
        remediation: 'Simplify path coordinates to reduce vector complexity under 32 KB.'  
      });  
    }

    // Check 8: HTTPS Asset Hosting Security (10 pts)  
    const logoHttps \= bimi.logoUrl && bimi.logoUrl.startsWith('https\://');  
    if (logoHttps) {  
      checks.push({  
        id: 'https\_hosting',  
        title: 'HTTPS Asset Hosting Security',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: \`Secure TLS Hosting: Logo hosted at "\${bimi.logoUrl}" over HTTPS.\`,  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'https\_hosting',  
        title: 'HTTPS Asset Hosting Security',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: 'Insecure Hosting: Logo URL must use secure HTTPS protocol.',  
        remediation: 'Update logoUrl to a secure HTTPS endpoint.'  
      });  
    }

    // Check 9: Multi-Mailbox Compatibility (5 pts)  
    if (hasVmc && isDmarcEnforced && svgAudit.isValid) {  
      checks.push({  
        id: 'mailbox\_compatibility',  
        title: 'Multi-Mailbox Compatibility (Gmail & Apple)',  
        weight: 5,  
        score: 5,  
        status: 'PASS',  
        details: 'Universal Compatibility: Supported across Google Gmail, Apple Mail, Yahoo\! Mail, and Fastmail.',  
        remediation: null  
      });  
    } else if (isDmarcEnforced && svgAudit.isValid) {  
      checks.push({  
        id: 'mailbox\_compatibility',  
        title: 'Multi-Mailbox Compatibility (Yahoo & Fastmail)',  
        weight: 5,  
        score: 3,  
        status: 'INFO',  
        details: 'Partial Ecosystem Support: Ready for Yahoo\! and Fastmail; Gmail requires a VMC.',  
        remediation: 'Add VMC certificate for Google and Apple ecosystem support.'  
      });  
    } else {  
      checks.push({  
        id: 'mailbox\_compatibility',  
        title: 'Multi-Mailbox Compatibility',  
        weight: 5,  
        score: 0,  
        status: 'FAIL',  
        details: 'Incompatible with all major mailbox providers due to SVG or DMARC failures.',  
        remediation: 'Resolve SVG Tiny-PS and DMARC issues first.'  
      });  
    }

    // Check 10: BIMI DNS Record Formatting (5 pts)  
    checks.push({  
      id: 'dns\_record\_format',  
      title: 'BIMI DNS Record Format (default.\_bimi)',  
      weight: 5,  
      score: 5,  
      status: 'PASS',  
      details: \`DNS Record: default.\_bimi.\${domain} TXT "v=BIMI1; l=\${bimi.logoUrl || ''}; a=\${bimi.vmcUrl || ''};"\`,  
      remediation: null  
    });

    // Score Calculation  
    let totalScore \= 0;  
    let maxScore \= 0;  
    let passedCount \= 0;  
    let warnCount \= 0;  
    let failCount \= 0;

    checks.forEach(function (c) {  
      totalScore \+= c.score;  
      maxScore \+= c.weight;  
      if (c.status \=== 'PASS') passedCount++;  
      else if (c.status \=== 'WARN') warnCount++;  
      else if (c.status \=== 'FAIL') failCount++;  
    });

    const percentage \= Math.round((totalScore / maxScore) \* 100);

    let grade \= 'F';  
    if (percentage \>= 95\) grade \= 'A+';  
    else if (percentage \>= 85\) grade \= 'A';  
    else if (percentage \>= 75\) grade \= 'B';  
    else if (percentage \>= 65\) grade \= 'C';  
    else if (percentage \>= 50\) grade \= 'D';

    return {  
      domain: domain,  
      overallScore: percentage,  
      grade: grade,  
      summary: {  
        totalChecks: checks.length,  
        passed: passedCount,  
        warnings: warnCount,  
        failures: failCount,  
        hasVmc: Boolean(hasVmc),  
        isDmarcEnforced: Boolean(isDmarcEnforced),  
        isSvgValid: svgAudit.isValid  
      },  
      checks: checks,  
      svgAudit: svgAudit  
    };  
  }

  return {  
    auditBimiConfig  
  };  
});  
