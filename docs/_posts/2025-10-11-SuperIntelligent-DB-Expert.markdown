---
layout: post
title:  "Super Intelligent VS DB Experts"
date:   2025-10-11 10:39:13 -0400
categories: Salesforce
tags: Salesforce
---

{% raw %}
<style>
  .mermaid-container {
    position: relative;
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    margin: 20px 0;
    background: #f9f9f9;
  }
  
  .mermaid-wrapper {
    overflow: auto;
    max-height: 600px;
    position: relative;
  }
  
  .mermaid-zoom-controls {
    position: absolute;
    top: 10px;
    right: 10px;
    display: flex;
    gap: 5px;
    z-index: 10;
    background: rgba(255, 255, 255, 0.9);
    padding: 5px;
    border-radius: 5px;
    border: 1px solid #ccc;
  }
  
  .mermaid-zoom-btn {
    background: #007bff;
    color: white;
    border: none;
    padding: 5px 10px;
    border-radius: 3px;
    cursor: pointer;
    font-size: 12px;
    min-width: 30px;
  }
  
  .mermaid-zoom-btn:hover {
    background: #0056b3;
  }
  
  .mermaid-zoom-btn:disabled {
    background: #ccc;
    cursor: not-allowed;
  }
  
  .mermaid-diagram {
    transition: transform 0.2s ease;
    transform-origin: top left;
  }
  
  .zoom-level {
    font-size: 11px;
    padding: 5px 8px;
    background: #f8f9fa;
    border: 1px solid #dee2e6;
    border-radius: 3px;
    min-width: 45px;
    text-align: center;
  }
</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pako/2.1.0/pako.min.js"></script>
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  
  mermaid.initialize({ 
    startOnLoad: true,
    theme: 'default',
    securityLevel: 'loose'
  });
  window.diagrams = {};
  
  // Convert code blocks to mermaid diagrams with zoom controls
  document.querySelectorAll('pre > code.language-mermaid').forEach((codeBlock, index) => {
    const diagramContent = codeBlock.textContent;
    const diagramId = `mermaid-diagram-${index}`;
    
    // Store diagram content if it doesn't exist
    if (!window.diagrams[diagramId]) {
      window.diagrams[diagramId] = diagramContent;
    }
    
    // Create container with zoom controls
    const container = document.createElement('div');
    container.className = 'mermaid-container';
    container.innerHTML = `
             <div class="mermaid-zoom-controls">
         <button class="mermaid-zoom-btn" onclick="zoomDiagram('${diagramId}', 'in')">+</button>
         <span class="zoom-level" id="${diagramId}-level">100%</span>
         <button class="mermaid-zoom-btn" onclick="zoomDiagram('${diagramId}', 'out')">−</button>
         <button class="mermaid-zoom-btn" onclick="zoomDiagram('${diagramId}', 'reset')">⌂</button>
         <button class="mermaid-zoom-btn" onclick="openInMermaidLive('${diagramId}')" title="Open in Mermaid Live">🔗</button>
       </div>
      <div class="mermaid-wrapper">
        <pre class="mermaid mermaid-diagram" id="${diagramId}">${diagramContent}</pre>
      </div>
    `;
    
    codeBlock.parentElement.replaceWith(container);
  });
  
  // Zoom functionality
  window.zoomDiagram = function(diagramId, action) {
    const diagram = document.getElementById(diagramId);
    const levelSpan = document.getElementById(`${diagramId}-level`);
    
    if (!diagram || !levelSpan) return;
    
    // Get current zoom level
    const currentTransform = diagram.style.transform;
    const currentScale = currentTransform.match(/scale\(([^)]+)\)/) 
      ? parseFloat(currentTransform.match(/scale\(([^)]+)\)/)[1]) 
      : 1;
    
    let newScale = currentScale;
    
    switch(action) {
      case 'in':
        newScale = Math.min(currentScale * 1.2, 5); // Max 300%
        break;
      case 'out':
        newScale = Math.max(currentScale / 1.2, 0.3); // Min 30%
        break;
      case 'reset':
        newScale = 1;
        break;
    }
    
    // Apply transform
    diagram.style.transform = `scale(${newScale})`;
    
    // Update level display
    levelSpan.textContent = `${Math.round(newScale * 100)}%`;
    
    // Update button states
    const container = diagram.closest('.mermaid-container');
    const zoomInBtn = container.querySelector('.mermaid-zoom-btn:first-child');
    const zoomOutBtn = container.querySelector('.mermaid-zoom-btn:nth-child(3)');
    
    zoomInBtn.disabled = newScale >= 3;
    zoomOutBtn.disabled = newScale <= 0.3;
  };

  function jsStringToByte(data) {
  return new TextEncoder().encode(data);
}

function jsBtoa(data) {
  let binary = '';
  const bytes = new Uint8Array(data);
  for (let i = 0; i < bytes.byteLength; i++) {
    binary += String.fromCharCode(bytes[i]);
  }
  return btoa(binary);
}

function genPakoLink(graphMarkdown, editMode) {
  const jGraph = {
    grid: true,
    code: graphMarkdown,
    mermaid: { theme: 'default' },
    panZoom: true,
    rough: false,
    updateDiagram: true,
    renderCount: 5,
    updateEditor: false,
    "autoSync": true
  }
  
  const byteStr = jsStringToByte(JSON.stringify(jGraph));

  const deflated = pako.deflate(byteStr);

  const dEncode = "pako:" + jsBtoa(deflated);

  const link =
    `http://mermaid.live/${editMode ? 'edit' : 'view'}#` +
    dEncode.replace('+', '-').replace('/', '_');

  return link;
}
  
     // Function to open diagram in Mermaid Live
   window.openInMermaidLive = function(diagramId) {
     const diagram = document.getElementById(diagramId);
     if (!diagram) return;
     
     // Get the original diagram content
     //const diagramContent = diagram.textContent.trim();
     const diagramContent = window.diagrams[diagramId];
     console.log("diagramContent:" + diagramContent);
     
     // Encode the diagram content for URL
     try {
       // Try to use pako compression if available
       if (window.pako) {
         const url = genPakoLink(diagramContent);
         console.log('Open live diagram with pako link ' + url);
         window.open(url, '_blank');
       }
     } catch (error) {
       console.error('Error encoding diagram:', error);
     }
   };
   
   // Add keyboard shortcuts
   document.addEventListener('keydown', function(e) {
     if (e.ctrlKey || e.metaKey) {
       const focusedDiagram = document.querySelector('.mermaid-container:hover .mermaid-diagram');
       if (focusedDiagram) {
         if (e.key === '=' || e.key === '+') {
           e.preventDefault();
           zoomDiagram(focusedDiagram.id, 'in');
         } else if (e.key === '-') {
           e.preventDefault();
           zoomDiagram(focusedDiagram.id, 'out');
         } else if (e.key === '0') {
           e.preventDefault();
           zoomDiagram(focusedDiagram.id, 'reset');
         }
       }
     }
   });
</script>

{% endraw %}



I have a complex stored procedure like ![complex stored procedure](https://raw.githubusercontent.com/bobbercheng/blog/main/docs/pictures/PER029_UDF_T_VOC_SUM_Daily.png) in SQL server that depends 
some tables, functions. I need to migrate it as lot of similar SPs to Postgres. Normally it takes a senior DB engineer 1-2 hours for one port if all dependencies are ready. This is a typical mechanical task for DB expert, not hard but need operations with domain knowledge and experiences.  In [Vexorium](https://www.vexorium.net/), we do it in different way. We build the agent with super intelligence that could surpass senior experts. One key feature of super agent is able to work on hard task hours days just by self reflection without human assistant. During our experiences, two LLM models demos great capabilities of self reflection and validation, one is Gemini 2.5 Pro from Google, it takes 12 rounds reflection to pass final validation. The other is GPT-5 from OpenAI, it takes 10 rounds reflection. The agent takes less than half hour to complete the migration, ![trace_per029_udf_t_voc_sum_daily](https://raw.githubusercontent.com/bobbercheng/blog/main/docs/pictures/trace_per029_udf_t_voc_sum_daily.png).

Above stored procedure is just a small part of big IT project/product in big companies. In decades, all IT departments is struggling to modernize legacy systems including me as it's too complex and take too much effort/risk. With super intelligence, we show different possibility for all IT modernization. 

Please DM me @ [my Linkedin] if you are interested in super intelligence.

{% raw %}
<script src="https://giscus.app/client.js"
        data-repo="bobbercheng/blog"
        data-repo-id="R_kgDOLrBZsw"
        data-category="Ideas"
        data-category-id="DIC_kwDOLrBZs84Coy7e"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
{% endraw %}


[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
