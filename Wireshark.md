
## Installation
1. Put the marker/probe in places SPAs commonly use:
    
    - Query string (`?q=...`)
        
    - Hash (`#q=...`)
        
2. In DevTools:
    
    - Network: confirm the server response didn’t already contain it (to distinguish reflected vs DOM).
        
    - Elements: confirm it appears only after JS runs (DOM-based).
        
3. Optional but strong: in **Sources → Event Listener Breakpoints**:
    
    - Enable **DOM Mutations** (subtree modifications / attribute modifications)
        
    - Reload and see what script modifies the DOM near the injection point
        
4. Look at the code: if you see patterns like assigning untrusted data into HTML-producing APIs, that’s typically your root cause.