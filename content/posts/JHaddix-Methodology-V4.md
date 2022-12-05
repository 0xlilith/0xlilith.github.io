---
author: "lilith"
title: "JHaddix Methodology V4"
date: "2022-02-09"
description: "My Recon Notes For JHaddix Methodology V4"
tags:
  - blog
---
```
[ASN]───────────┐                 ┌───────[Root Domain]
                    |────[Recon]──{[TARGET]}
    [Acquisition]───|
                    |
    [Linked]────────|────[Whois]
                    |
        [SubDom]────┘
```
### FINDING SEED DOMAIN

- https://crunchbase.com -> find related acquasitions of the company
- (ASN Finding) -> search for company name and asn numbers (doesnt tarcks cloud infra. spaces)
    - https://bgp.he.net
    - asnlookup.py(yassineaboukir)
    - metabigor(j3ssiejjj) -> [echo 'tesla' | metabigor net --org -v]
    - asnrecon
- (ASN Enum) -> For discovering more seed domains and return root domains
    - amass(Jeff Foley) -> [amass intel -asn 46489] 
- (Reverse whois)
    * https://www.whoxy.com -> who has owned the company in the past 
    * DOMLink.py(Vincent Yiu) -> recursively query whoxy whois api
- (Ad/Analytics Relationship)
    * https://builtwith.com/ -> gives technology profile and relations profile   
- (Google-Fu)
    * eg: "2019 Twitch INteractive, Inc." inurl:twitch
- (Shodan)
    * https://www.shodan.io/search?query=twitch.tv -> spiders infrastructure on the internet
    

### SUBDOMAIN ENUMERATION
```
 [subdomain enum]
    |-[linked and js discovery] 
    |-[subdomain scraping]
    |-[subdomain bruitforce]
    |- ++
```

- (Linked and Js discovery)
    * Burp Suite Pro -> hybrid technique to find both root/seed and subdomain
        + visit a seed/root and spider all the linked html and js
        + turn off passive scanning
        + set forms auto to submit
        + set scope to advance control and use "keyword" of target
        + browse the site, then spider all hosts
    * gospider(j3ssiejjj) -> [gospider -s https://www.twitch.tv] 
    * hakrawler(hakluke)
    * subdomainizer(Neeraj Edwards) -> analyzing js and find out subdomains
        + find cloud services referenced in js files
        + use shannon entrophy formula to find sensitive items in js files such as api keys
    * subscraper(Cillian Collins) -> just scrapes js files and finds subdomains

- (Subdomain Scraping) -> find more subdomains
    - (Google) { - is used to exclude and narrow down the results}
        + site:twitch.tv -www.twitch.tv
        + site:twitch.tv -www.twitch.tv -watch.twitch.tv
        + site:twitch.tv -www.twitch.tv -watch.twitch.tv -dev.twitch.tv
    * amass -> [amass -d twitch.tv]
        + amass correlates the scraped domain to ASN and list where they came forms
        + this helps to know the technology they are using
    * subfinder(projectdiscovery.io) -> [subfinder -d twitch.tv -v]
    * github-subdomains.py(Gwendal Le) -> scrapes github repos and parse into list of subdomains
    * shosubgo(inc0gbyt3) -> shodan scraper to find subdomain using {needs api key}
    * (Cloud Ranges) -> technique to monitor whole cloud range for SSL sites and parse cert. to match target
        + eg: [curl 'https://tls.bufferover.run/dns?q=.twitch.tv' 2>/dev/null | jq .Results]
        + Sam Erb defcon talk on how to set up such service

- (Subdomain Bruitforce) -> checking for live subdomains by resolving them
    * amass -> [amass enum -bruit -d twitch.tv -src] | [amass enum -bruit -d twitch.tv -rf resolvers.txt -w wordlist.list]
    * shuffledns(projectdiscovery.io) -> [shuffledns -d twitch.tv -w list.txt -r resolvers.txt]
    - (wordlist)
        * Tailered wordlist
            + make wordlist in fly
            + TomNomNom tool for making Tailered wordlist {who, what, where when, wordlist talk}
        * Massive wordlist
            + use all in one massive wordlist
            + all.txt by JHaddix
            + https://github.com/assetnote/commonspeak2
    (Alteration Scanning) -> pattern of bruit forcing when a common sequence
        * eg: dev.company.com | dev1.company.com | dev2.company.com | dev-1.company.com | Dev.2.company.com
         
### OTHERS

- (port scan)
    * masscan(Robert Graham) -> Used for scanning open ports {only for ip and not domain and much faster than nmap}
                                [masscan -p1-65535 -iL $ipfile --max-rate 1800 -oG $output.log]
        + syntax guide: https://danielmiessler.com/study/masscan
    * dnmasscan(rastating)   -> Wrapper around masscan to scan domains
                                [dnmasscan example.txt dns.log -oG scan.log] 
                                {example.txt contains dns and the ip will be stored in dns.log for further scanning}
- (service scan)
    * brutespray(x90skysn3k) -> scans for default remote admin protocols after port scan
- (github dorking) 
    * Jhaddix bash script for github dork wrapper
    * github-search(Gwendal Le)
    * th3g3ntelman's full module on github sensitive data exposure
- (Screenshotting)
    * Aquatone
    * HTTPscreenshot
    * Eyewitness (for both http and https)
      
### SUBDOMAIN TAKEOVER

> occurs when a sub is pointing to a service which is removed or deleted.
> you can setup a page in similar service that was being used and point to that sub.

* can-i-take-over-xyz(EdOverflow) -> github repo on how to claim subs for different services
* subover(Ic3man/projectdiscovery.io)
* nuclei(projectdiscovery.io) -> large scale tool
         
### AUTOMATION++

* Interlace(Codingo) -> help glue up together these recon tools {hakluke's guide on it}
    + CIDR input
    + Glob input
    + threading
    + proxying
    + queue command
    + etc..