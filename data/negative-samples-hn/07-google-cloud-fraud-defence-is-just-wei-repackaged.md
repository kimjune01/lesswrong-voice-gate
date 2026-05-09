# Google Cloud Fraud Defence is just WEI repackaged

Source: https://privatecaptcha.com/blog/google-cloud-fraud-defence-wei/
HN score: 686

Google Cloud Fraud Defence is just WEI repackaged | Private Captcha

          

        

    

  

  
  

    

      

        
        

          

            

              

            

            
            

              

                

  
Use cases

  

    

      

        Form protection
      

    

      

        GDPR compliance
      

    

      

        Secure WordPress
      

    

      

        Rate limiting
      

    

      

        Accessibility
      

    

  

                

  
Features

  

    

      

        Smart difficulty scaling
      

    

      

        Custom rules
      

    

      

        EU-only endpoints
      

    

      

        Audit logs
      

    

      

        Platform API
      

    

      

        Usage statistics
      

    

      

        Widget customization
      

    

      

        DNS proxy
      

    

  

                

  
Alternative

  

    

      

        Google reCAPTCHA
      

    

      

        hCaptcha
      

    

      

        CloudFlare Turnstile
      

    

      

        Friendly Captcha
      

    

  

                

                  

                    
                      Docs
                    
                  

                

                
                

                  

                    
                      Pricing
                    
                  

                

                
                

                  

                    
                      Self-hosting
                    
                  

                

                

            

            
            

                
Log In

              
Sign Up

            

            
            

              

                

                  

                

                

                  

                    

                      
Use cases

                      

                        

                          
Form protection

                        

                          
GDPR compliance

                        

                          
Secure WordPress

                        

                          
Rate limiting

                        

                          
Accessibility

                        

                      

                    

                    

                      
Features

                      

                        

                          
Smart difficulty scaling

                        

                          
Custom rules

                        

                          
EU-only endpoints

                        

                          
Audit logs

                        

                          
Platform API

                        

                          
Usage statistics

                        

                          
Widget customization

                        

                          
DNS proxy

                        

                      

                    

                    

                      
Alternative

                      

                        

                          
Google reCAPTCHA

                        

                          
hCaptcha

                        

                          
CloudFlare Turnstile

                        

                          
Friendly Captcha

                        

                      

                    

                    
                    

                      
Docs

                    

                    
                    

                      
Pricing

                    

                    
                    

                      
Self-hosting

                    

                    
                  

                

                
                

                  
Log In

                  
Sign Up

                

              

            

            
            

              

              

              

            

          

        

      

    

  

  

    

      

        
updated on May 9, 2026 / by Taras Kushnir

        
Google Cloud Fraud Defence is just WEI repackaged

        

          

            
In May 2026, Google 
announced
 “Google Cloud Fraud Defense - the next evolution of reCAPTCHA.” The announcement described a QR code challenge where users scan a code with their phone to prove human presence.

Google killed 
Web Environment Integrity
 in 2023 after standards bodies objected. Today, three years later, the same device attestation mechanism launched as a commercial product.

The open web survived because no single company could decide which hardware was legitimate enough to use it. Google is determined to end that status quo - now through a reCAPTCHA update.

    

Table of Contents

    

  

    

Google already tried this in 2023

    

The QR code will be bypassed

    

QR auth codes and device attestation are not new

    

Device attestation bars the users who need privacy most

    

“Legitimate” tracking

    

Final thoughts

  

    

        

        

      

Google already tried this in 2023

In June 2023, a Google engineer named Yoav Weiss posted 
a proposal
 to the Chromium project called 
“Web Environment Integrity.”
 The mechanism was direct: browsers would ask device hardware to sign a cryptographic attestation proving the browser was unmodified and running on Google-certified hardware. Websites could verify the signature and decide whether to serve content without friction or add a challenge. Of course, the proposal framed this as protecting web integrity against bots and automated scraping.

Mozilla published a 
formal position
 within days. The proposal 
“works against users’ interests”
 and 
“creates a gated internet controlled by OS and device vendors.”
 The Electronic Frontier Foundation 
called it
 “Chrome’s Plan to DRM the Web,” noting that by design, only Chrome running on Android or other certified hardware would easily pass attestation, routing traffic toward Google’s ecosystem as a structural consequence, not a side effect.

Google withdrew WEI three weeks after publication. The Chromium GitHub thread closed. Publicly, it was dead.

In May 2026, Google 
announced
 Google Cloud Fraud Defense, described in its blog post as 
“the next evolution of reCAPTCHA.”
 The system challenges users with a QR code: scan it with your phone to confirm human presence. The requirements page specifies the hardware that qualifies: 
“modern Android device with Google Play Services installed, or modern iPhone/iPad.”

“Google Play Services installed”
 is doing significant work in that sentence. Google Play Services is Google’s closed-source software layer that runs on certified Android devices and provides the attestation APIs - the Play Integrity API specifically - that prove a device is unmodified and approved by Google. A device without Play Services cannot satisfy Play Integrity checks at the level Fraud Defense requires. That is not a technical limitation waiting to be engineered around. It is 
the
 mechanism.

The WEI review process, whatever its limitations, required Google to defend the mechanism publicly. The proposal was withdrawn because the objections held. With Fraud Defense, there was no process to respond to. The product launched. The requirements page went live. The same attestation infrastructure that generated those documented objections in 2023 became the underpinning of a commercial service available to any organization with a Google Cloud billing account.

    

        

        

      

The QR code will be bypassed

Here is how the challenge works: a user encounters a Fraud Defense prompt and is asked to scan a QR code with their phone camera. The phone, authenticated against Google’s Play Integrity API, confirms the device is certified hardware. That confirmation returns to the originating site as proof of human presence.

The defeat is mechanical. Bot operators point a camera at a screen, a trivial automation with off-the-shelf hardware. For operations that need Play Integrity attestation specifically, a compliant Android device costs approximately 
$30
 (
$29.88
 
in Wallmart
 to be precise) - for a professional bot farm, which purchases devices in bulk, this is the fixed cost without material disruption to operations.

One additional failure worth noting: one incident response professional in the 
HN thread
, raised a concern that operates independently of the bot problem:

How should we realistically teach Susan from HR the difference between a real Google Captcha QR code and a malicious phishing QR code - you (realistically) can’t.

The QR challenge trains users to scan codes to access websites. Phishing campaigns will exploit that trained behavior immediately.

    

        

        

      

QR auth codes and device attestation are not new

In the Apple world iOS App Attestation verifies that an app was installed through the App Store and has not been modified. It governs apps: a walled garden users chose when they purchased an iPhone. The extension to open web browsing is categorically different: it conditions URL access on hardware a private company has certified. No precedent exists for this applied to the open internet. App stores are opt-in ecosystems with explicit terms of service. The web was not designed to have terms of hardware.

QR-based authentication systems themselves already exist for a while. Estonia’s 
Smart ID
 uses QR codes to verify users, but for bounded, consent-scoped resources: banking portals, government services, health records. The user chooses to authenticate. The protected resource is defined in advance. The scope is explicit. Google Cloud Fraud Defense applies device attestation to the open web, to any URL an operator chooses to gate, without equivalent consent architecture, without purpose limitation, and very likely without user awareness that their hardware identity is functioning as an access credential.

    

        

        

      

Device attestation bars the users who need privacy most

Google Play Integrity attestation requires Google Play Services. 
GrapheneOS
, the security-hardened Android fork recommended by the EFF and used by journalists, lawyers, and activists in high-risk environments, does not ship Play Services by default. It supports a sandboxed compatibility layer that runs some Play Services functionality, but this does not satisfy Play Integrity at the 
MEETS_DEVICE_INTEGRITY
 level that Fraud Defense requires. 
LineageOS
 for microG (a privacy-oriented Android distribution built specifically for users who want an open-source alternative) fails for the same reason. Any custom ROM that excludes Play Services fails.

Firefox for Android does not appear in Google’s stated browser support list for Fraud Defense. This is not an oversight. Firefox does not integrate Google Play Integrity by design - Mozilla’s position on device attestation in 2023 was explicit and remains current. The practical effect: users of the most privacy-respecting major mobile browser are excluded from verified access by default, not because they are bots, but because they use software that declines to participate in Google’s certification architecture.

    

        

        

      

“Legitimate” tracking

The governance problem is the obvious objection. The tracking problem is the one that gets less attention.

Every Fraud Defense challenge that resolves successfully sends a signal to Google: this certified device accessed this site at this time. Device attestation does not just gate access - it produces attribution. A device with a stable hardware identity creates a persistent identifier that crosses sessions, browsers, and private browsing modes. The company that defines which hardware is 
“legitimate”
 also accumulates a running record of where that hardware goes on the open web. That is not a side effect of fraud defense. It is an architectural 
consequence
 decision of tying verification to certified device identity.

A technically credible alternative exists that avoids both the governance problem and the tracking problem. Private Captcha and 
similar proof-of-work systems
 issue cryptographic challenges that require computational effort 
(dis)
 proportional to volume. One human solving a single challenge pays a negligible cost. A bot farm running concurrent sessions faces exponential compute costs with each additional attempt and AI agents, which consume GPU cycles to operate, face identical penalties regardless of how sophisticated their reasoning is. No hardware identifier is transmitted, no attestation is required, and no certification layer determines who may participate. User privacy is structurally preserved, not promised.

    

        

        

      

Final thoughts

Google Cloud Fraud Defense is not a reCAPTCHA update. The QR code is the visible mechanism, but device attestation is the real product. Every resolved challenge tells Google which certified hardware accessed which site at which time. The same infrastructure standards bodies rejected in 2023 now operates behind a commercial release, accumulating attribution data that WEI, as a public proposal, would never have been permitted to build unchallenged. Ironically, it will fail to stop bots similarly to the version it is designed to “improve” upon.

          

        

      

    

  

  

  

    

      

        

          

        

        
Made and hosted in the EU 🇪🇺

      

      

          
General

          

                

                Status 

              

            

                

                Documentation
              

            

                

                Integrations
              

            

                

                Self-hosting
              

            

                

                Blog
              

            

                

                FAQ
              

            

                

                About
              

            

        

          
Legal

          

                

                Terms and Conditions
              

            

                

                DPA
              

            

                

                End-User Privacy Policy
              

            

                

                Service Privacy Policy
              

            

                

                EULA
              

            

        

          
Use cases

          

                

                Form protection
              

            

                

                GDPR compliance
              

            

                

                Secure WordPress
              

            

                

                Rate limiting
              

            

                

                Accessibility
              

            

        

          
Alternative to

          

                

                Google reCAPTCHA
              

            

                

                hCaptcha
              

            

                

                CloudFlare Turnstile
              

            

                

                Friendly Captcha
              

            

        

    

    

      

        Copyright © 2025-2026 Intmaker OÜ. All rights reserved.
