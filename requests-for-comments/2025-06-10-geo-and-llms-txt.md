# RFC: GEO (Generative Engine Optimization) and llms.txt

## TL;DR
We’re kicking off some groundwork to improve PostHog’s discoverability and compatibility in LLM web retrieval. This falls under GEO (Generative Engine Optimization), the buzzword equivalent of SEO for AI or AI search.

This RFC is: 
1. A rough starting point for the team to think about GEO (I spent a lot of time digging into this right before I left LastCo, so I wanted to jot my thoughts down now as the topic picks up momentum)
2. A ceremonial kickoff for work like `llms.txt` (PR coming this week)

## What's the problem

People are turning to web-enabled AI tools like ChatGPT and Perplexity for search. There's plenty of debate about what this means for Google (they have a new [AI search mode](https://www.youtube.com/watch?v=qbqZQFOVfA8) in beta) and traditional search engines, but AI search is growing fast. 

And because our target audience consists of developers and founders — early adopters of this kind of tech — this is where they'll be looking for answers and solutions.

So, how do we make sure PostHog is mentioned for prompts like:
1. `Which product analytics tools are open-source`
2. `Alternatives to Mixpanel or Amplitude` 
3. `Top data warehouse solutions`
4. `What CRM should I use for my startup?`
5. `What is the *ultimate* customer data platform?`

We're well-positioned to show up for the first two prompts today, but not the rest. Why? The simple GEO answer: it's because there's not enough content in enough places for the LLMs to confidently mention PostHog as an answer.

The name of the game is to influence what information LLMs retrieve, especially through their *web search* and *deep research* features, for the prompts we care about. But what sources do they trust and cite during web retrieval? And what kind of content?

*Spoiler*: It's a lot of channels.
- Our own web pages
- Traditional SERP, top 20 results on Google
- Press announcements and PR articles like techcrunch and forbes
- Organic discussions on Reddit or other community boards 
- Third-party blogs
- Crappy syndicated posts like "Top 5 tools for X" on sites like dev.to
- YouTube
- Wikipedia
- And the list goes on...

AI web search pulls from a broad and fuzzy pool of sources. What gets prioritized depends on the prompt, the industry, and other external factors (e.g., [OpenAI partnering with Reddit](https://openai.com/index/openai-and-reddit-partnership/) or [Perplexity partnering with Bing](https://www.hyperlinker.ai/en/seo/perplexity-ai-bing)). It's basically a black box. 

But wait! There are fancy new GEO tools that track branded mentions across LLMs, so we can measure our AI "rank" or "visibility" for the prompts we're targeting. We just started using one called [Gauge](https://withgauge.com/).

## Okay, so what actually changes?

In terms of creating content, not much. Content still has to be [great](https://newsletter.posthog.com/i/156101009/your-great-content-probably-sucks), and there's no easy hack for that.

But where things *do* change is in distribution and coverage. 

In GEO, our primary goal is to have our brand, content, or insights included in the AI's synthesized response. This means we need more content in more places for the LLMs to retrieve and reference.

For example, in the context GEO, it's more valuable for PostHog to appear across multiple spots in the top 20 Google results than to only be mentioned just once in the #1 spot. 

This also implies more third-party content. Think:
- Traditional PR and press (TechCrunch, Forbes, etc.)
- Partnerships and syndication (e.g. comparison articles on dev.to or newsletters)
- Youtube content
- Community discussion on forums like Reddit

![PR is important again 😂](https://res.cloudinary.com/dmukukwp6/image/upload/SCR_20250611_lqlo_3ca5da33be.png)

## A note on training vs. retrieval

Our content can appear in AI responses through either pathway: by being part of the model's training data or by being dynamically retrieved during a web search operation or other RAG methods. 

GEO focuses on the second path, where LLMs act more like reasoning engines, pulling from the web, distilling a bunch of information, and providing a structured answer with citations.

## Some good reads
- https://a16z.com/geo-over-seo/
- https://www.mbcrosier.com/posts/ai-search-optimization
- https://www.ignorance.ai/p/seo-for-ai-a-look-at-generative-engine

## Solutions

### llms.txt protocol
I have a working branch. Submitting a PR this week.

It's a [new standard](https://llmstxt.org/) that's been adopted by [Anthropic](https://docs.anthropic.com/llms.txt), [Cloudflare](https://developers.cloudflare.com/llms.txt), [MCP](https://modelcontextprotocol.io/llms.txt), [Cursor](https://docs.cursor.com/llms.txt), and others. We don’t know yet if the big LLM companies will officially treat it like `robots.txt`, but developers are already using it for context injection during retrieval.

It basically turns all our public-facing content (docs, blog, and newsletters) into LLM-friendly sitemaps and markdown pages for AI crawling.

### Content distribution

Open ended. Food for thought for content marketing.

### Third-party content

Open ended. Food for thought for content marketing.

## Reasons to do it

- GEO is early, investing could create a long-term advantage
- LLMs are fast becoming a major organic discovery channel for developers (and people in general)
- We’re great at content marketing, plays to our strength

## Reasons not to do it

- GEO is early, lots of speculation and there are no best practices
- Attribution will be unclear
- Still lots to do within traditional SEO

## What does success look like?

- Referral traffic coming from ChatGPT, Perplexity, etc.
  - Caveat: Many LLMs link to third-party sources, not PostHog directly. This might lead to branded search instead of clickthroughs, so attribution may show up as “direct” or "search"
- More mentions and citations tracked in GEO tools like Gauge 
- More signups citing ChatGPT or other LLMs on signup, "How did you hear about us?"

## Open questions

- Should we track bot traffic? We don't today, but some talking heads are predicting that it will become a key way to track GEO. LLM bots will crawl our site on behalf of users – so monitoring LLM bot activity could help us measure our AI search performance.

## Proposed next steps
- `llms.txt`
- ?
