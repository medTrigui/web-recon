# Web Recon

This repository is a resource hub for web reconnaissance and crawling techniques. It provides concise, practical guides and hands-on tools to help you understand and perform web recon efficiently.

## What's Inside?

- **Guides:**
  - Clear, actionable markdown files on web crawling, crawling strategies (breadth-first, depth-first), and the use of robots.txt.
  - Explanations of what data can be extracted during recon and why context matters.

- **Tools:**
  - `crawlies/ReconSpider.py`: A powerful Python crawler built with Scrapy that automates the process of discovering emails, links, files, and more from target websites.
  - `crawlies/requirements.txt`: Dependencies for running the ReconSpider script.

## Why Use This Repo?
- Learn the essentials of web crawling and reconnaissance.
- Get practical, ready-to-use scripts for automating recon tasks.
- Understand how to analyze and interpret the data you collect.

## Getting Started
1. Check out the guides in the `crawlies` folder for a quick understanding of web crawling and reconnaissance.
2. Use the ReconSpider script to automate your own web recon tasks:
   ```bash
   cd crawlies
   pip install -r requirements.txt
   python ReconSpider.py <start_url>
   ```
   Results will be saved in `results.json`.

---

Whether you're a security professional, researcher, or enthusiast, this repo is designed to help you master the art of web reconnaissance. 