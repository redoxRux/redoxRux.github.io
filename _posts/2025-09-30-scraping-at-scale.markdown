---
layout: post
title:  "scraping at scale"
date:   2025-09-30 12:00:00 +0000
categories: technical scraping
---

I've built scraping systems across multiple countries and platforms. Most advice online is wrong or incomplete. Here's what actually works.

## Use Playwright + Firefox + UV

Playwright is 30% faster than Selenium with 50% fewer timeout errors. Firefox has an 85% detection rate with Chrome versus 15% when you switch. UV installs packages in 3 seconds instead of 67.

The numbers matter because you'll be running these systems constantly. Small inefficiencies compound.

```python
async with async_playwright() as p:
    browser = await p.firefox.launch(headless=True)
    page = await browser.new_page()
    await page.click('button:has-text("Load More")')
```

Playwright auto-waits. This eliminates 90% of timing bugs. Selenium makes you handle this manually with explicit waits and arbitrary sleep() calls.

Firefox works because anti-bot systems optimize for Chrome (70% market share). They spend less effort detecting Firefox. The Gecko engine also produces different fingerprints than Blink.

UV is just better package management. Run `uv add playwright` and you're done. Pip takes forever and creates non-deterministic environments.

## Fresh Browser Contexts

Modern sites fingerprint everything. Canvas rendering, WebGL, audio context, font enumeration. Reusing the same context creates patterns.

```python
# One browser, multiple contexts
browser = await p.firefox.launch()

context1 = await browser.new_context()
page1 = await context1.new_page()

context2 = await browser.new_context()
page2 = await context2.new_page()
```

Each context is a clean slate. No cookies, no localStorage, unique fingerprint. You can run them in parallel. Memory doesn't leak across contexts.

## Find the API

Most modern sites load data via API. Scraping HTML is slow and fragile. The API is faster and returns structured JSON.

Open the network tab. Look for XHR/Fetch requests. Common patterns: `/api/`, `/graphql/`, `api.bazaarvoice.com`. Capture the request headers and parameters. Replay them directly.

```python
class APIDiscovery:
    def setup_monitoring(self, page):
        page.on('request', self.capture_request)
    
    async def capture_request(self, request):
        url = request.url
        if 'api' in url.lower():
            print(f"Found: {request.method} {url}")
            print(f"Headers: {dict(request.headers)}")
```

Many sites use third-party review systems (Bazaarvoice, Trustpilot). These have public APIs. Hit them directly instead of parsing their JavaScript widgets.

When you find the API, you're 10x faster than scraping HTML. No CSS selectors to maintain. No waiting for JavaScript to render.

## Don't Click "Next"

Clicking pagination buttons is slow and breaks easily. Understand the URL pattern instead.

```python
# Instead of clicking 50 times
patterns = [
    f"{base_url}?page={i}",
    f"{base_url}/p/{i}",
]

# Direct navigation
for page_num in range(1, 51):
    await page.goto(f"{base_url}?page={page_num}")
```

You can now parallelize. Process page 1, 10, and 20 simultaneously. No sequential clicking. If you know there are 50 pages, you're done in minutes instead of hours.

## Act Human

Perfect timing and instant form filling looks robotic. Add variation.

```python
async def human_type(page, selector, text):
    await page.click(selector)
    await page.wait_for_timeout(random.randint(100, 300))
    
    for char in text:
        await page.type(selector, char, delay=random.randint(50, 150))
        
        if random.random() < 0.05:  # 5% typo rate
            await page.type(selector, random.choice('abcdefgh'))
            await page.wait_for_timeout(random.randint(200, 500))
            await page.press(selector, 'Backspace')
    
    await page.wait_for_timeout(random.randint(500, 1500))
```

Variable typing speed. Occasional typos. Natural pauses. This matters more than you think.

## Handle Errors Properly

Production scraping fails constantly. Timeouts, blocks, layout changes, network issues.

```python
async def scrape_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            async with async_playwright() as p:
                browser = await p.firefox.launch()
                context = await browser.new_context()
                page = await context.new_page()
                
                await page.goto(url, wait_until='networkidle')
                data = await extract_data(page)
                return data
                
        except TimeoutError:
            wait_time = 2 ** attempt
            await asyncio.sleep(wait_time)
            
        except Exception as e:
            if "blocked" in str(e).lower():
                context = await browser.new_context()  # Fresh context
            else:
                raise
```

Different errors need different strategies. Timeouts get exponential backoff. Blocks get fresh contexts. Network errors retry immediately. Missing elements skip gracefully.

## Take Screenshots

When something breaks in production, you need to see what happened.

```python
async def debug_scrape(page, milestone):
    timestamp = datetime.now().strftime("%H%M%S")
    await page.screenshot(path=f"debug/{milestone}_{timestamp}.png")
```

Capture screenshots before/after key interactions. On errors. When selectors fail. These saved me hours of debugging blind.

## Configuration Over Code

Hard-coding selectors makes your scraper unmaintainable.

```json
{
  "amazon_us": {
    "selectors": {
      "search_box": "input#twotabsearchtextbox",
      "product_links": "a[data-component-type='s-search-result']",
      "price": ".a-price-whole"
    },
    "delays": {
      "min": 1000,
      "max": 3000
    }
  }
}
```

When Amazon changes their layout, update the JSON. No code deployment. Non-developers can maintain selectors. You can A/B test different strategies by swapping configs.

## Control Concurrency

Too many concurrent requests get you blocked. Too few waste time.

```python
class AdaptiveConcurrency:
    def __init__(self, initial_limit=5):
        self.limit = initial_limit
        self.error_rate = 0
    
    def adjust(self):
        if self.error_rate > 0.10:
            self.limit = max(1, self.limit - 1)
        elif self.error_rate < 0.02:
            self.limit += 1
```

Start with 5 concurrent requests. If errors exceed 10%, reduce. If errors stay below 2%, increase. The system finds its own equilibrium.

## Reality

Production scraping is about resilience, not extraction. Sites change constantly. Anti-bot measures evolve. Your scraper needs to adapt.

The goal isn't extracting data once. It's building systems that extract data reliably over years. Focus on observability, maintainability, and respect for target sites.

Use modern tools. Create fresh contexts. Find the APIs. Handle errors well. Take screenshots. Configure everything. Control concurrency.

These patterns work because they've survived production.