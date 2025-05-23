# RFC: HogTok videos #project-dopamine-brainrot

## TL;DR

For hackathon, @andyvan-ph @jabahamondes and I created a series of short, punchy TikTok- and Instagram story-style videos to help developers quickly discover and learn more about PostHog features. 

We also built a video player UI that can be embedded within the PostHog app (us/eu.posthog.com) and the website (posthog.com/docs).

@jabahamondes has a PR up [here](https://github.com/PostHog/posthog/pull/32226)

Slack channel #project-dopamine-brainrot

## What's the problem

PostHog's platform continues to expand with new product lines and features :hog-party:. But key features sometimes go unnoticed if you're not digging in the docs, changelogs, or UI. 

We hear lots of anecdotal feedback about how our platform's feature set can be hard to grok at a glance. In some cases, users never bother to find or learn about the very PostHog features that actually solve their problems. @daniloc and I heard from a Sales AE about a customer who was going to migrate to MixPanel because they were convinced that PostHog doesn't have group analytics....

What if developers had a more bite-sized, engaging way to discover PostHog features that could supplement reading the docs and changelogs? That's the idea behind HogTok videos.

## Solution
Create short-form video content that lives *within* the PostHog app and website. These embedded videos highlight individual features or key product concepts in under 60 seconds, with fast-paced editing, captions, and auto-playing the next video to keep things engaging and speedy.

We think developers, and people in general, enjoy watching short videos and will often rip through a bunch in one sitting. It could be an effective way way to onboard users to the PostHog platform.

## Why use a video format like TikTok?

Well, it's the most consumed content format in the world for a reason – highly engaging, addicting, and flexible. We're meeting users where their attention span is 😵‍💫. 
 
@andyvan-ph and I also agree that most technical videos made by dev fool companies that are longer than a few minutes tend to lose focus and often ramble. They're really hard to execute well, and most viewers tune out. These longer-form videos serve a different purpose – they're better suited for deep dives and implementation, not feature discovery.

Maybe the biggest advantage of the 30-60s length format is how much faster and easier it is to make than traditional Youtube-style tutorials. I was able to create four HogTok videos in a single hackathon day.

## HogTok requirements

Each HogTok video:
- Has a light, humorous tone with music
- Is 9:16 vertical format
- Is under 60 seconds (ideal target: 30 seconds)
- Uses CapCut for editing
  - Can reuse their massive library of editing templates, auto captions, transitions, music, etc.
- Uses screen.studio for screen recording
- Uses RoboNilo AI voiceover from [ElevenLabs](https://www.elevenlabs.io) (I'm so sorry @daniloc)
- Has a video thumbnail that grabs attention (e.g. bold text, faces, UI highlights)

The HogTok video player:
- Hosts videos via Cloudinary
- Is a collection of React components
  - A carousel of video thumbnails
  - A modal that opens the video player on click
- Videos can be grouped by playlists or tags
- Can be embedded in the PostHog app (us/eu.posthog.com)
- Can be embedded in the PostHog website (posthog.com/docs)
- Is tracked with PostHog events

## Examples

Videos:
- [Changelog: Save filters for session replay](https://res.cloudinary.com/dmukukwp6/video/upload/changelog_save_filters_replay_1_9aabb9799c.mp4)
- [Changelog: Linear share modal for session replay](https://res.cloudinary.com/dmukukwp6/video/upload/changelog_linear_share_1_11fdaee4cd.mp4)
- [Toolbar overview](https://res.cloudinary.com/dmukukwp6/video/upload/toolbar_2_0a34e62550.mp4)
- [Toolbar actions](https://res.cloudinary.com/dmukukwp6/video/upload/toolbar_actions_07e751a76a.mp4)

Images: 
- https://res.cloudinary.com/dmukukwp6/image/upload/toolbar_inspect_a43e85f7d3.png
- https://res.cloudinary.com/dmukukwp6/image/upload/toolbar_heatmap_d68abfa438.png
- https://res.cloudinary.com/dmukukwp6/image/upload/toolbar_feature_flags_6723eb2704.png
- https://res.cloudinary.com/dmukukwp6/image/upload/toolbar_cool_features_f4e9a08d4c.png

## Reasons to do it
- Helps developers discover and learn more about PostHog features
- It's unique, gives PostHog more cool + weird vibes, and helps us stand out even more from the B2B SaaS crowd
- Let's us experiment with new content formats that blend education, style / tone, and product marketing

## Reasons not to do it
- It's too much effort to create this type of content
- It doesn't help developers learn about our features and products
- It adds to the clutter and busyness of PostHog's platform

## What does success look like?
- Video engagement (watches, clicks, shares)
- Increase in feature awareness 
- Increase in feature adoption

## Open Questions
- Is the short-form format only effective for UI-level or high-level product overviews? 
    - Can we meaningfully show and explain code in 30s-60s?
- Should the goal be conversion (i.e. getting users to sign up) or feature adoption (i.e. getting existing users to try new features)?
    - IMO we should focus on feature adoption / awareness
- What other topics could HogTok videos be great for?
    - Behind-the-scenes or "Founderfluence" videos (e.g., James or Tim sharing roadmap vision)
    - Adapted customer stories
    - Community spotlights

## Proposed next steps
- Ship hackathon MVP under feature flag (this week May 19-23) @jabahamondes
- Create an overview HogTok video for each PostHog top-line product like error tracking, LLM observability, data warehouse, surveys etc. (next week May 26-30)
- Create a few more feature tutorial Hogtok videos (?)