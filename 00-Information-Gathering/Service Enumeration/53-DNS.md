# DNS
- UDP/TCP 53
- gobuster dns -d megacorpone.com -w wordlist.txt -t 10
- 
## Enumeration
```
dig @192.168.205.172  vault.offsec any


```

## Zone Transfer
```bash
dig @192.168.205.172  vault.offsec axfr
```

##  DNS AI Wordlist
AI PROMPT: 
```LLM

Using public data from MegacorpOne's website and any information that can be inferred about its organizational structure, products, or services, generate a comprehensive list of potential subdomain names.
	•	Incorporate common patterns used for subdomains, such as:
	•	Infrastructure-related terms (e.g., "api", "dev", "test", "staging").
	•	Service-specific terms (e.g., "mail", "auth", "cdn", "status").
	•	Departmental or functional terms (e.g., "hr", "sales", "support").
	•	Regional or country-specific terms (e.g., "us", "eu", "asia").
	•	Factor in industry norms and frequently used terms relevant to MegacorpOne's sector.

Finally, compile the generated terms into a structured wordlist of 1000  words, optimized for subdomain brute-forcing against megacorpone.com

Ensure the output is in a clean, lowercase format with no duplicates, no bulletpoints and ready to be copied and pasted.
Make sure the list contains 1000 unique entries.
```
* We know feed our wordlist into gobuster, for DNS enum
	* `gobuster dns -d megacorpone.com -w wordlist.txt -t 10`