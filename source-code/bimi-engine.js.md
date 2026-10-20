/\*\*  
 \* BIMIVerifyForge Pro \- Core BIMI & SVG Tiny-PS Engine  
 \* RFC 8463 (BIMI) and AuthIndicators SVG Tiny 1.2 Portable/Secure Profile Analyzer & Sanitizer  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.BIMIEngine \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  const FORBIDDEN\_TAGS \= \['script', 'style', 'foreignobject', 'iframe', 'audio', 'video', 'animate', 'image'\];  
  const FORBIDDEN\_ATTRS \= \['onclick', 'onload', 'onerror', 'onmouseover', 'onfocus', 'onblur'\];

  /\*\*  
   \* Parse viewBox attribute into numbers  
   \*/  
  function parseViewBox(viewBoxStr) {  
    if (\!viewBoxStr) return null;  
    const parts \= viewBoxStr.trim().split(/\[\\s,\]+/).map(Number);  
    if (parts.length \=== 4 && \!parts.some(isNaN)) {  
      return { minX: parts\[0\], minY: parts\[1\], width: parts\[2\], height: parts\[3\] };  
    }  
    return null;  
  }

  /\*\*  
   \* Validate SVG compliance with SVG Tiny 1.2 Portable/Secure (Tiny-PS)  
   \*/  
  function validateSvgTinyPs(svgContent) {  
    const issues \= \[\];  
    if (\!svgContent || typeof svgContent \!== 'string') {  
      return { isValid: false, issues: \['SVG content is empty or invalid string.'\] };  
    }

    const clean \= svgContent.trim();  
    if (\!clean.startsWith('\<svg') || \!clean.endsWith('\</svg\>')) {  
      issues.push('Content is not an enclosed \<svg\> root element.');  
    }

    // 1\. Version & baseProfile checks  
    const hasVersion12 \= /version\\s\*=\\s\*\["'\]1\\.2\["'\]/i.test(clean);  
    const hasTinyPs \= /baseProfile\\s\*=\\s\*\["'\]tiny-ps\["'\]/i.test(clean);  
    if (\!hasVersion12) {  
      issues.push('Missing or invalid SVG version: must be version="1.2" per RFC 8463.');  
    }  
    if (\!hasTinyPs) {  
      issues.push('Missing or invalid baseProfile: must be baseProfile="tiny-ps".');  
    }

    // 2\. Aspect Ratio (1:1 square check)  
    const vbMatch \= clean.match(/viewBox\\s\*=\\s\*\["'\](\[^"'\]+)\["'\]/i);  
    let isSquare \= false;  
    if (vbMatch) {  
      const vb \= parseViewBox(vbMatch\[1\]);  
      if (vb && vb.width \> 0 && vb.width \=== vb.height) {  
        isSquare \= true;  
      } else if (vb) {  
        issues.push(\`Non-square aspect ratio: viewBox is \${vb.width}x\${vb.height}, but BIMI mandates square 1:1 ratio.\`);  
      }  
    } else {  
      issues.push('Missing "viewBox" attribute on root \<svg\> element.');  
    }

    // 3\. Mandatory \<title\> element  
    const hasTitle \= /\<title\[^\>\]\*\>\[\\s\\S\]\*?\<\\/title\>/i.test(clean);  
    if (\!hasTitle) {  
      issues.push('Missing mandatory \<title\> tag inside SVG root element.');  
    }

    // 4\. Forbidden Tags check  
    FORBIDDEN\_TAGS.forEach(function (tag) {  
      const tagRegex \= new RegExp('\<' \+ tag \+ '\[\\\\s\>/\]', 'i');  
      if (tagRegex.test(clean)) {  
        issues.push(\`Forbidden element detected: \<\${tag}\> is strictly prohibited by SVG Tiny-PS specification.\`);  
      }  
    });

    // 5\. Forbidden Attributes check  
    FORBIDDEN\_ATTRS.forEach(function (attr) {  
      const attrRegex \= new RegExp('\\\\s' \+ attr \+ '\\\\s\*=', 'i');  
      if (attrRegex.test(clean)) {  
        issues.push(\`Forbidden event handler attribute detected: "\${attr}".\`);  
      }  
    });

    // 6\. External link / raster check  
    if (/xlink:href\\s\*=\\s\*\["'\]http/i.test(clean) || /href\\s\*=\\s\*\["'\]http/i.test(clean)) {  
      issues.push('Forbidden external hyperlink reference detected in SVG.');  
    }

    // 7\. Size check (\< 32 KB)  
    const byteLength \= (new TextEncoder().encode(clean)).length;  
    const isUnderSizeLimit \= byteLength \<= 32768;  
    if (\!isUnderSizeLimit) {  
      issues.push(\`SVG file size is \${(byteLength / 1024).toFixed(1)} KB (exceeds recommended 32 KB threshold for email clients).\`);  
    }

    return {  
      isValid: issues.length \=== 0,  
      issues: issues,  
      byteLength: byteLength,  
      isSquare: isSquare,  
      hasVersion12: hasVersion12,  
      hasTinyPs: hasTinyPs,  
      hasTitle: hasTitle  
    };  
  }

  /\*\*  
   \* Automated SVG Tiny-PS Sanitizer  
   \* Cleans standard SVGs into strict SVG Tiny-PS compliant format  
   \*/  
  function sanitizeToTinyPs(rawSvg, brandName) {  
    if (\!rawSvg || typeof rawSvg \!== 'string') return '';  
    let clean \= rawSvg.trim();

    // 1\. Remove forbidden tags entirely  
    FORBIDDEN\_TAGS.forEach(function (tag) {  
      const removeRegex \= new RegExp('\<' \+ tag \+ '\[^\>\]\*\>\[\\\\s\\\\S\]\*?\<\\\\/' \+ tag \+ '\>', 'gi');  
      clean \= clean.replace(removeRegex, '');  
      const selfClosingRegex \= new RegExp('\<' \+ tag \+ '\[^\>\]\*\\\\/\>', 'gi');  
      clean \= clean.replace(selfClosingRegex, '');  
    });

    // 2\. Strip forbidden attributes  
    FORBIDDEN\_ATTRS.forEach(function (attr) {  
      const attrRegex \= new RegExp('\\\\s+' \+ attr \+ '\\\\s\*=\\\\s\*\["\\'\]\[^"\\'\]\*\["\\'\]', 'gi');  
      clean \= clean.replace(attrRegex, '');  
    });

    // 3\. Extract or compute viewBox  
    const vbMatch \= clean.match(/viewBox\\s\*=\\s\*\["'\](\[^"'\]+)\["'\]/i);  
    let size \= 512;  
    if (vbMatch) {  
      const vb \= parseViewBox(vbMatch\[1\]);  
      if (vb) {  
        size \= Math.max(vb.width, vb.height);  
      }  
    }

    // 4\. Clean root \<svg\> tag attributes and inject Tiny-PS requirements  
    clean \= clean.replace(/\<svg\\b(\[^\>\]\*)\>/i, function (match, attrs) {  
      // Remove existing version, baseProfile, width, height, viewBox  
      let newAttrs \= attrs  
        .replace(/\\bversion\\s\*=\\s\*\["'\]\[^"'\]\*\["'\]/gi, '')  
        .replace(/\\bbaseProfile\\s\*=\\s\*\["'\]\[^"'\]\*\["'\]/gi, '')  
        .replace(/\\bviewBox\\s\*=\\s\*\["'\]\[^"'\]\*\["'\]/gi, '')  
        .replace(/\\bwidth\\s\*=\\s\*\["'\]\[^"'\]\*\["'\]/gi, '')  
        .replace(/\\bheight\\s\*=\\s\*\["'\]\[^"'\]\*\["'\]/gi, '');

      return \`\<svg xmlns="http\://www\.w3.org/2000/svg" version="1.2" baseProfile="tiny-ps" viewBox="0 0 \${size} \${size}" width="\${size}" height="\${size}"\${newAttrs}\>\`;  
    });

    // 5\. Ensure \<title\> tag exists  
    const titleText \= brandName || 'Brand Logo';  
    if (\!/\<title\[^\>\]\*\>\[\\s\\S\]\*?\<\\/title\>/i.test(clean)) {  
      clean \= clean.replace(/\<svg\\b\[^\>\]\*\>/i, \`\$&\\\\n  \<title\>\${titleText}\</title\>\`);  
    }

    return clean;  
  }

  /\*\*  
   \* Synthesize standard BIMI DNS Record  
   \*/  
  function generateBimiRecord(logoUrl, vmcUrl) {  
    const l \= logoUrl ? logoUrl.trim() : '';  
    const a \= vmcUrl ? vmcUrl.trim() : '';  
    return \`v=BIMI1; l=\${l}; a=\${a};\`;  
  }

  return {  
    FORBIDDEN\_TAGS,  
    FORBIDDEN\_ATTRS,  
    parseViewBox,  
    validateSvgTinyPs,  
    sanitizeToTinyPs,  
    generateBimiRecord  
  };  
});  
