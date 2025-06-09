---
layout: post
title:  "System designs for Salesforce developers"
date:   2025-06-08 10:39:13 -0400
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
  
  // Convert code blocks to mermaid diagrams with zoom controls
  document.querySelectorAll('pre > code.language-mermaid').forEach((codeBlock, index) => {
    const diagramContent = codeBlock.textContent;
    const diagramId = `mermaid-diagram-${index}`;
    
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
    code: graphMarkdown,
    mermaid: { theme: 'default' },
  };

  const byteStr = jsStringToByte(JSON.stringify(jGraph));

  const deflated = pako.deflate(byteStr);

  const dEncode = jsBtoa(deflated);

  const link =
    `http://mermaid.live/${editMode ? 'edit' : 'view'}#pako:` +
    dEncode.replace('+', '-').replace('/', '_');

  return link;
}
  
     // Function to open diagram in Mermaid Live
   window.openInMermaidLive = function(diagramId) {
     const diagram = document.getElementById(diagramId);
     if (!diagram) return;
     
     // Get the original diagram content
     const diagramContent = diagram.textContent.trim();
     
     // Encode the diagram content for URL
     try {
       // Try to use pako compression if available
       if (window.pako) {
         console.log('Open live diagram with pako link');
         const url = genPakoLink(diagramContent);
         window.open(url, '_blank');
       }
     } catch (error) {
       console.error('Error encoding diagram:', error);
       // Fallback: copy content to clipboard and open mermaid.live
       navigator.clipboard.writeText(diagramContent).then(() => {
         alert('Diagram content copied to clipboard. Paste it in Mermaid Live.');
         window.open('https://mermaid.live/', '_blank');
       }).catch(() => {
         alert('Please copy the diagram manually and paste it in Mermaid Live.');
         window.open('https://mermaid.live/', '_blank');
       });
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

## General salesforce app architecture like EBikes (generated by Gemini-2.5-pro):
Prompt: ```This project deployed ApexClass,  ApexPage, AuroDefinationBundle, ContentAsset, CspTrustedSite, CustomApplication, CustomField, CustomSite, CustomTab, ExperienceBuddle, FlexiPage, Layout, LightningComponentBundle, ListView, NavigatorMenu, network, networkBranding, permissionset, platformEventChannelMember, prompt and staticResource to salesforce. Please go through code and those depolyment objects, give me system comonents diagram to explain how they work together.```

```mermaid
graph TD
    subgraph "User Interface (Experience Cloud & Salesforce)"
        A[ExperienceBundle: E-Bikes]
        B[CustomSite: E-Bikes Site]
        C[FlexiPage: Lightning Pages]
        D[LWC Components]
        E[Aura Components]
        F[ContentAsset: Logo, Images]
        G[StaticResources: CSS, JS]
        H[NavigationMenu: Site Navigation]
    end

    subgraph "Application"
        I[CustomApplication: E-Bikes App]
        J[CustomTabs]
    end

    subgraph "Business Logic"
        K[Apex Classes]
    end

    subgraph "Data Model"
        L[Custom Objects & Fields]
        M[Layouts]
        N[ListViews]
    end

    subgraph "Security & Access"
        O[PermissionSets]
        P[CspTrustedSite]
    end
    
    subgraph "Events & Messaging"
        Q[PlatformEventChannelMember]
        R[Message Channels]
    end

    %% Relationships
    B -- "Uses" --> A
    A -- "Contains" --> C
    A -- "Uses" --> H
    C -- "Composed of" --> D
    C -- "Composed of" --> E
    I -- "Contains" --> J
    J -- "Displays" --> C
    J -- "Displays" --> N
    D -- "Call" --> K
    E -- "Call" --> K
    K -- "Access/Manipulate" --> L
    L -- "Displayed on" --> M
    M -- "Used by" --> O
    O -- "Grant Access to" --> K
    O -- "Grant Access to" --> L
    O -- "Grant Access to" --> C
    O -- "Grant Access to" --> D
    O -- "Grant Access to" --> E
    A -- "Branded by" --> F
    D -- "Use" --> G
    B -- "Configured with" --> P
    K -- "Publish/Subscribe" --> Q
    D -- "Publish/Subscribe" --> R
    E -- "Publish/Subscribe" --> R
```


## Easy Spaces app architecture (generated by Claude-4-sonnet):
Aux prompt: ```adjust the layout to make it fittable for screen ratio```

```mermaid
graph TD;
    subgraph AppLayer [es-space-mgmt: Application Layer]
        direction TB
        AppDefinition["App, Tabs, PermissionSets"]
        FlexiPages["6 FlexiPages<br/>Contact, Market, Reservation Records<br/>Designer, Manager, UtilityBar"]
        AppAura["4 Aura Components<br/>customerDetails, spaceDesignerAura<br/>reservationHelperAura, designFormCmp"]
        AppLWCs["11 Lightning Web Components<br/>Customer: List, Tile, DetailForm<br/>Reservation: List, Tile, Helper, HelperForm<br/>Space: Designer, DesignForm, RelatedSpaces"]
        AppApex["2 Apex Controllers<br/>ReservationManager<br/>ReservationManagerController"]
        AppFlows["2 Flows<br/>createReservation<br/>spaceDesigner"]
        Prompts["2 Einstein Prompts<br/>CreateReservation, SpaceDesigner"]
    end

    subgraph LogicLayer [es-base-code: Business Logic Layer]
        direction TB
        SharedApex["3 Apex Services<br/>MarketServices<br/>CustomerServices<br/>TestDataFactory"]
        UtilityLWCs["2 Utility LWCs<br/>errorPanel<br/>ldsUtils"]
        UtilityAura["2 Utility Aura<br/>openRecordAction<br/>selectObject Event"]
        CustomMDT["Custom Metadata<br/>Customer_Fields__mdt<br/>Contact & Lead Configurations"]
        MessageChannels["2 Lightning Message Channels<br/>Tile_Selection<br/>Flow_Status_Change"]
    end

    subgraph StyleLayer [es-base-styles: UI Foundation Layer]
        direction TB
        Branding["Branding & Theme<br/>BrandingSet<br/>Lightning Theme"]
        Assets["Content Assets<br/>Logos, Images, Tiles"]
        StyleLWCs["4 Style LWCs<br/>imageGallery, imageTile<br/>pill, pillList"]
        StyleAura["2 Aura Layout Components<br/>AppPage_3_9<br/>AppPage_4_8"]
    end

    subgraph DataLayer [es-base-objects: Data Foundation Layer]
        direction TB
        Objects["3 Core Objects<br/>Market__c, Space__c<br/>Reservation__c"]
        ObjectMeta["Object Support<br/>Tabs, Layouts, ListViews<br/>Path Assistants"]
        ObjectPerms["EasySpacesObjects<br/>PermissionSet"]
    end

    %% Vertical Dependencies
    AppLayer --> LogicLayer
    AppLayer --> StyleLayer
    AppLayer --> DataLayer
    LogicLayer --> DataLayer

    %% Key Interactions (Simplified)
    AppLWCs --> SharedApex
    AppLWCs --> UtilityLWCs
    AppApex --> SharedApex
    AppFlows --> AppApex
    AppAura --> AppLWCs
    FlexiPages --> AppAura
    AppLWCs --> MessageChannels
    AppFlows --> MessageChannels
    SharedApex --> CustomMDT
    SharedApex --> Objects
```
### Coral Cloud Resort System Architecture (generated by Claude-4-sonnet):
Prompt: ```Go through project and all packages, give me a system diagram to explain how sf components from all packages works together. For each flow, please summary it in the diagram.```

```mermaid
graph TB
    subgraph "Coral Cloud Resort System Architecture"
        
        subgraph "cc-base-app (Core Package)"
            subgraph "Data Model"
                Experience["Experience__c<br/>Resort Activities"]
                Session["Session__c<br/>Time Slots"]
                Booking["Booking__c<br/>Reservations"]
                Contact["Contact<br/>Guest Profiles"]
                Credit["Credit__c<br/>Resort Credits"]
                Review["Guest_Review__c<br/>Feedback"]
            end
            
            subgraph "Core Flows"
                CreateBooking["Create_Booking<br/>Booking Process"]
                GetCustomer["Get_Customer_Details<br/>Profile Lookup"]
                GetExperience["Get_Experience_Details<br/>Activity Info"]
                GetSessions["Get_Sessions<br/>Available Times"]
                IssueCredits["Issue_Bulk_Resort_Credits<br/>Credit Management"]
                RouteQueue["Route_to_Queue<br/>Service Routing"]
            end
            
            subgraph "UI Components (LWC)"
                ExperienceOverview["experienceOverview<br/>Activity Display"]
                ExperienceSchedule["experienceSchedule<br/>Time Selection"]
                SessionBooking["sessionBookingOverview<br/>Booking Summary"]
                ContactProfile["contactProfileCard<br/>Guest Info"]
                Paginator["paginator<br/>Navigation"]
            end
        end
        
        subgraph "cc-employee-app (Employee Tools)"
            subgraph "AI-Powered Features"
                GenAIPrompts["GenAI Prompt Templates<br/>- Personalized Schedule<br/>- Social Media Posts<br/>- Guest Reviews Summary<br/>- Welcome Emails"]
                GenAIFunctions["GenAI Functions<br/>- Policy Questions<br/>- Weather Check<br/>- Credit Management"]
                GenAIPlanners["GenAI Planners<br/>AI Orchestration"]
            end
            
            subgraph "Employee Flows"
                PersonalizedSchedule["PersonalizedSchedule<br/>AI-Generated Itineraries"]
                ExperienceLoop["Experiences_Loop<br/>Bulk Processing"]
                AddExperiences["Add_Experiences_to_Prompt<br/>Content Enhancement"]
            end
            
            subgraph "Employee UI"
                SocialMediaGen["generateSocialMediaPosts<br/>Marketing Content"]
            end
        end
        
        subgraph "cc-service-app (Service & Support)"
            subgraph "AI Agent System"
                CoralCloudAgent["Coral Cloud Agent<br/>Einstein Service Agent"]
                BotConfig["Bot Configuration<br/>Multi-channel Support"]
            end
            
            subgraph "Service Infrastructure"
                ServiceChannels["Service Channels<br/>Messaging, Chat, etc."]
                Queues["Queues<br/>Work Distribution"]
                PresenceConfig["Presence Configuration<br/>Agent Availability"]
                RoutingConfig["Routing Configuration<br/>Intelligent Assignment"]
            end
            
            subgraph "Service Flows"
                RouteToAgent["Route_to_Agent<br/>AI Agent Assignment"]
                RouteToQueue2["Route_to_Queue<br/>Human Agent Routing"]
            end
            
            subgraph "Digital Experience"
                DigitalExp["Digital Experiences<br/>Service Portal"]
                EmbeddedService["Embedded Service<br/>Chat Widget"]
            end
        end
        
        subgraph "cc-site (Experience Cloud)"
            subgraph "Public Website"
                SitePages["Site Pages<br/>Public Content"]
                SiteExperiences["siteExperiences<br/>Activity Catalog"]
                ExperiencesTile["experiencesTile<br/>Activity Cards"]
                SiteHeader["siteHeader<br/>Navigation"]
                SiteFooter["siteFooter<br/>Site Info"]
            end
            
            subgraph "Site Infrastructure"
                Network["Network Configuration<br/>Site Settings"]
                NavigationMenus["Navigation Menus<br/>Site Structure"]
                StaticResources["Static Resources<br/>Assets & Files"]
            end
        end
        
        subgraph "External Integrations"
            MessagingChannels["Messaging Channels<br/>WhatsApp, SMS, etc."]
            RemoteSites["Remote Site Settings<br/>External APIs"]
            NamedCredentials["Named Credentials<br/>Secure Connections"]
        end
    end
    
    %% Data Flow Connections
    Experience --> Session
    Session --> Booking
    Contact --> Booking
    Booking --> Credit
    Contact --> Review
    
    %% Core Flow Connections
    CreateBooking --> Booking
    GetCustomer --> Contact
    GetExperience --> Experience
    GetSessions --> Session
    IssueCredits --> Credit
    
    %% UI Component Connections
    ExperienceOverview --> Experience
    ExperienceSchedule --> Session
    SessionBooking --> Booking
    ContactProfile --> Contact
    
    %% Employee App Connections
    PersonalizedSchedule --> GenAIPrompts
    PersonalizedSchedule --> Contact
    PersonalizedSchedule --> Experience
    SocialMediaGen --> GenAIPrompts
    GenAIFunctions --> Contact
    GenAIFunctions --> Credit
    
    %% Service App Connections
    CoralCloudAgent --> RouteToAgent
    CoralCloudAgent --> GenAIFunctions
    RouteToAgent --> ServiceChannels
    RouteToQueue2 --> Queues
    MessagingChannels --> CoralCloudAgent
    EmbeddedService --> CoralCloudAgent
    
    %% Site Connections
    SiteExperiences --> Experience
    ExperiencesTile --> Experience
    Network --> DigitalExp
    
    %% Cross-Package Dependencies
    cc-site -.-> cc-base-app
    cc-employee-app -.-> cc-base-app
    cc-service-app -.-> cc-base-app
    
    %% Flow Summaries
    classDef flowBox fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef dataBox fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef uiBox fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef aiBox fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef serviceBox fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    
    class CreateBooking,GetCustomer,GetExperience,GetSessions,IssueCredits,RouteQueue,PersonalizedSchedule,ExperienceLoop,AddExperiences,RouteToAgent,RouteToQueue2 flowBox
    class Experience,Session,Booking,Contact,Credit,Review dataBox
    class ExperienceOverview,ExperienceSchedule,SessionBooking,ContactProfile,Paginator,SocialMediaGen,SiteExperiences,ExperiencesTile,SiteHeader,SiteFooter uiBox
    class GenAIPrompts,GenAIFunctions,GenAIPlanners,CoralCloudAgent aiBox
    class ServiceChannels,Queues,PresenceConfig,RoutingConfig,EmbeddedService,MessagingChannels serviceBox
```


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
