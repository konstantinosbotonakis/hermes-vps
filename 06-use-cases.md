# 06, Hermes Automation Use Cases

These are practical examples you can copy and adapt.

The best automations are boring. They save you from checking the same things manually.

## 1, Daily AI news briefing

Use this if you want to stay current without reading 20 links every morning.

```text
Every weekday at 8am, search the web for important AI agent, LLM, and automation news from the last 24 hours. Find at least 5 sources. Summarise the top 3 stories in plain English, include links, and deliver the result to Telegram. Keep it under 300 words.
```

Good for:

- Engineering managers.
- Founders.
- Product teams.
- People tracking the AI tooling market.

## 2, Website uptime monitor

Use this for a small business website, SaaS site, eCommerce store, or support portal.

```text
Every 10 minutes, check whether https://example.com is online. If the site returns HTTP 200 within 3 seconds, reply with [SILENT]. If the site is down, slow, or returns an error, send me a Telegram alert with the status code, response time, and likely cause.
```

Good for:

- WordPress sites.
- OpenCart stores.
- SaaS landing pages.
- Support portals.

## 3, VPS health check

Use this to avoid logging into the server just to check basics.

```text
Every 6 hours, check disk usage, memory usage, CPU load, uptime, and Docker container status on this VPS. If everything looks healthy, reply with [SILENT]. If anything looks wrong, send me a Telegram alert with the exact issue and a suggested next action.
```

Good alert conditions:

- Disk usage over 85%.
- Memory usage consistently high.
- Load average much higher than CPU count.
- A required Docker container has stopped.
- The server restarted unexpectedly.

## 4, Disk space warning

This is a simple and useful first automation.

```text
Every hour, check disk usage on this VPS. If disk usage is below 80%, reply with [SILENT]. If disk usage is 80% or higher, send me a Telegram warning with the exact partition, percentage used, and the largest directories to check.
```

## 5, GitHub daily summary

Use this for a project you want to follow without checking GitHub constantly.

```text
Every weekday at 9am, check the GitHub repository OWNER/REPO. Summarise new pull requests, merged pull requests, open issues, failed workflows, and anything that needs attention. Format it as a short engineering standup update.
```

Replace:

```text
OWNER/REPO
```

with the real repository.

## 6, Competitor website monitoring

Use this if you run a product, plugin business, SaaS, agency, or eCommerce company.

```text
Every Monday at 9am, check these competitor websites: [add URLs]. Look for meaningful product changes, pricing changes, new features, new blog posts, and new landing page messaging. Ignore small visual changes. Send me a concise report with links.
```

Use this for:

- Plugin vendors.
- SaaS products.
- Agencies.
- eCommerce stores.
- Hosting companies.

## 7, Newsletter ideas

Useful if you publish content but do not want to start from a blank page.

```text
Every Friday at 10am, research the latest topics in OpenCart, WooCommerce, WordPress, eCommerce automation, and AI tools for merchants. Suggest 5 newsletter ideas. For each idea, include a title, short angle, why customers would care, and one suggested call to action.
```

## 8, Customer support digest

This works best if Hermes has access to the right source, such as exported support data, email, or a file.

```text
Every weekday at 5pm, review today's support requests from the available support source. Group issues by topic, identify urgent items, summarise recurring problems, and suggest which issue should be handled first tomorrow.
```

Good output format:

```text
1. Urgent issues
2. Repeated issues
3. Customers waiting for reply
4. Product bugs
5. Suggested next actions
```

## 9, Weekly business report

Use this when you want a small management summary every week.

```text
Every Monday at 8am, prepare a weekly business report. Include website uptime, support issues, GitHub activity, marketing content published, and anything that needs attention this week. Keep it under 500 words and deliver it to Telegram.
```

## 10, Product or pricing page watcher

Useful for competitor tracking or monitoring your own published pages.

```text
Every day at 9am, check these URLs for meaningful content, price, plan, feature, or call-to-action changes: [add URLs]. Ignore minor layout changes. Send me a summary only if something important changed.
```

## 11, Personal reminder with research

This is different from a normal reminder because it does some work before messaging you.

```text
Every Monday at 7am, check the weather for this week and suggest which day is best for outdoor family activities. Keep the answer short and send it to Telegram.
```

## 12, Automation for an OpenCart extension business

Example for a software shop or plugin vendor:

```text
Every Monday at 9am, check our public product pages and support articles. Identify outdated wording, missing FAQs, broken links, or support pages that need screenshots. Send me a prioritised list of the top 5 fixes.
```

Another example:

```text
Every Friday at 4pm, review recent product updates, support topics, and customer questions. Suggest 3 small plugin improvement ideas that could reduce support tickets or increase sales.
```

## Principle

Do not automate vague thinking.

Automate checks, summaries, reminders, reports, and monitoring tasks where the input and output are clear.
