# A web page that shows you everything the browser told it without asking

Source: https://sinceyouarrived.world/taken
HN score: 581

taken. — Since You Arrived Vol. IV

">

">

  

    
Since You Arrived · Vol. IV

    
taken.

    
This volume requires JavaScript. That is part of the point — your browser is what is being read.

    
With JavaScript off, the page cannot tell you what your browser disclosed. The data is still there. The disclosure still happened. Only the telling of it stops.

  

  

    
Since You Arrived · Vol. IV

    
taken.

    
You opened this page. It already knows the following.

  

  

    
reading

    

      

        

          

          

          

          

          

          

          

          

        

        

      

    

  

  

  

  

    

      
Vol. I
 is what the world did while you were here.
      
Vol. II
 is the sky you missed.
      
Vol. III
 is what was already at your feet.
      This was Vol. IV. We thought you should know.
    

    
sources & confessions

    
Vol. IV · April 2026

    
Created by 
Matt
, at 
Rise Up Labs
.

  

  

    
— Tell Someone —

    

      
What this page knew

      
A web page just told me everything it learned about me without asking.

      

    

    

      
𝕏 (Twitter)

      
Bluesky

      
LinkedIn

      
Facebook

      
Reddit

      
WhatsApp

      
Copy Link

    

    

      
Close

    

  

  

    
— A Note On What You Gave —

    
Sources & Confessions

    
Every observation on this page came from your own browser, in the first milliseconds after you arrived. The words were written by a human. A few honest footnotes follow.

    

      

        
Your location

        

ip-api.com · Free tier · CC-BY-SA

        
Your IP address arrives in the header of every request your device makes. We pass it to ip-api.com to translate it into a city and an internet provider name. The lookup is transient — neither side stores it. Under GDPR, an IP address can be considered personal data when used for tracking. We do not track. We do not retain. We do not log. We display only the first and last octet on screen. We know the rest. We chose not to display it.

      

      

        
Browser APIs

        

MDN Web Docs · Mozilla · CC-BY-SA 2.5

        
Every observation about your device — screen, browser, language, GPU, cores, battery, fonts, preferences — was retrieved through standard JavaScript APIs documented openly by Mozilla. No exploits, no vulnerabilities, no hacks. Everything on this page is by design. The design is the problem.

      

      

        
Font fingerprinting

        

Electronic Frontier Foundation · Cover Your Tracks (formerly Panopticlick)

        
The technique of detecting installed fonts by measuring rendered text widths has been documented since 2010. The EFF maintains a tool that lets you see how unique your browser is. Most browsers are unique enough to be tracked across the open web without any cookie at all. The combination of fonts is one of the strongest signals.

      

      

        
Canvas fingerprinting

        

Princeton University · Web Transparency & Accountability Project

        
A 2014 study from Princeton was the first to document canvas fingerprinting in the wild. Researchers found it on 5% of the top 100,000 websites — pages that secretly asked the visitor's browser to draw a hidden image, then read the rendered pixels back as an identifier. Your browser supports the technique. We did not draw one. The page you visit after this one might.

      

      

        
Clipboard API

        

MDN · Clipboard API specification

        
With a single user gesture — a click, a tap — a page can request to read the last thing you copied. A password. An address. A draft message. The capability is announced by every modern browser. We did not request it. The capability is there, available to any page that asks at the right moment.

      

      

        
The battery research

        
Olejnik, Englehardt, Narayanan · 2015 · "The Leaking Battery"

        
Published in the proceedings of the Workshop on Data Privacy Management. The paper demonstrated that the combination of battery percentage and discharge time was unique enough to track a visitor across multiple websites for up to thirty minutes — without cookies, without accounts. Firefox removed the API in 2016. Chrome and Edge still expose it.

      

      

        
The technique we did not run

        
Documented · Legal · Widely deployed

        
A page can detect which sites you are logged into by asking your browser to load favicon URLs from those sites and watching which succeed and which fail. Logged-in services return one image; logged-out services return another. The technique requires no permission. With it, a page can know — without asking — whether you are logged into Facebook, Google, X, GitHub, Reddit, LinkedIn, and dozens of others. We did not run this. The technique is documented and legal. Some of the pages you visited today did.

      

      

        
The barcode

        
Computed in your browser · 16 bars · Yours alone

        
Beneath the count, sixteen hairlines whose heights are derived from the data your device handed over — your GPU, your fonts, your screen size, your language, your timezone, your operating system, your browser, your color depth. Same data, same barcode. Different visitor, different barcode. The computation happens in your browser; nothing about it is transmitted. Anyone with your exact fingerprint would see the same bars. The likelihood is small.

      

      

        
The prose

        
Hand-written · Template-based, not generative

        
Every sentence on this page was written by Matt. The code selects among prose templates based on what your browser returned. No language model writes or rewrites anything at runtime. If a condition is not covered by hand-written prose, the page stays quiet about it — we'd rather say less than say something false.

      

      

        
What this page sent

        
Two anonymous events to our server

        
Two events: that you arrived, that you finished. No cookie, no identifier, no IP retained. Our server discards the body of each request and returns nothing. The transport-level record that the request happened exists in our hosting provider's logs for as long as their default retention runs — typically a few days. We did not configure that. Every site you visit has the same record. Most also send hundreds of additional beacons to advertisers, fingerprinters, session-replay tools, and tag managers. We send two, to ourselves, and tell you about them.

      

      

        
What this page stored on your device

        
Nothing

        
No cookies. No localStorage. No sessionStorage. No IndexedDB. No service worker cache. The data you saw was computed in your browser and never left your device, except for the IP geolocation lookup and the two anonymous events above. When you close this tab, this page forgets you exist. We are the only page you will visit today that can say this honestly.

      

      

        
Companion volumes

        

sinceyouarrived.world
 · 
/sky
 · 
/discovered

        
Vol. I is what the world did while you were here. Vol. II is the sky you missed. Vol. III is what was already at your feet. This was Vol. IV. The series zooms in with each volume — global, to city, to coordinates, to you. You may have noticed the path it was walking.

      

    

    

      Open the source. Read it. Everything described above is in the page you are reading. We have nothing to hide. Most pages cannot say that.
      

      
Future editions will appear on 
X
 and 
Bluesky
.

    

    
Close
