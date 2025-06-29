# Search Engine Discovery & Google Dorking

Search operators are like secret codes for search engines, letting you precisely target information across the web. Here are some essential and advanced operators:

| Operator         | Description                                              | Example                                      | Example Description                                      |
|------------------|----------------------------------------------------------|----------------------------------------------|----------------------------------------------------------|
| site:            | Limit results to a specific site/domain                  | site:example.com                             | Find all public pages on example.com                      |
| inurl:           | Find pages with a term in the URL                        | inurl:login                                  | Search for login pages                                   |
| filetype:        | Search for files of a specific type                      | filetype:pdf                                 | Find downloadable PDFs                                   |
| intitle:         | Find pages with a term in the title                      | intitle:"confidential report"               | Look for documents titled "confidential report"          |
| intext:          | Search for a term in the body text                       | intext:"password reset"                     | Find pages mentioning "password reset"                   |
| cache:           | Show cached version of a page                            | cache:example.com                            | View cached content                                      |
| link:            | Find pages linking to a URL                              | link:example.com                             | See who links to example.com                             |
| related:         | Find related websites                                    | related:example.com                          | Discover similar sites                                   |
| info:            | Get info about a page                                    | info:example.com                             | See title, description, etc.                             |
| define:          | Get definitions                                          | define:phishing                              | Define "phishing"                                       |
| numrange:        | Search for numbers in a range                            | site:example.com numrange:1000-2000          | Find numbers between 1000 and 2000                       |
| allintext:       | All words in body text                                   | allintext:admin password reset               | Both "admin" and "password reset" in text               |
| allinurl:        | All words in URL                                         | allinurl:admin panel                         | Both "admin" and "panel" in URL                         |
| allintitle:      | All words in title                                       | allintitle:confidential report 2023          | All words in title                                       |
| AND              | Require all terms                                        | site:example.com AND (inurl:admin OR inurl:login) | Find admin/login pages on example.com              |
| OR               | Any of the terms                                         | "linux" OR "ubuntu" OR "debian"             | Pages mentioning any of these                            |
| NOT              | Exclude a term                                           | site:bank.com NOT inurl:login                | Exclude login pages                                      |
| * (wildcard)     | Any character/word                                       | site:social.com filetype:pdf user* manual    | Find user guides/handbooks in PDF                        |
| .. (range)       | Numbers in a range                                       | site:ecommerce.com "price" 100..500          | Products priced 100–500                                  |
| " " (quotes)     | Exact phrase                                             | "information security policy"                | Exact phrase search                                      |
| - (minus)        | Exclude term                                             | site:news.com -inurl:sports                  | Exclude sports news                                      |

## Google Dorking
Google Dorking (Google Hacking) uses these operators to uncover sensitive info, vulnerabilities, or hidden content via Google Search.

**Common Google Dork Examples:**

| Purpose                    | Example(s)                                                                 |
|----------------------------|----------------------------------------------------------------------------|
| Finding Login Pages        | site:example.com inurl:login<br>site:example.com (inurl:login OR inurl:admin) |
| Identifying Exposed Files  | site:example.com filetype:pdf<br>site:example.com (filetype:xls OR filetype:docx) |
| Uncovering Config Files    | site:example.com inurl:config.php<br>site:example.com (ext:conf OR ext:cnf) |
| Locating DB Backups        | site:example.com inurl:backup<br>site:example.com filetype:sql           |

For more, see the Google Hacking Database. 