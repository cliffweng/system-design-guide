# Contributing

Thanks for helping improve the guide. A few ground rules:

- **Scope**: one topic per file under `topics/`. Keep each topic readable in ~10 minutes.
- **Template**: follow the structure already used in existing topic files (Why it matters, Core concepts, Mental model, Interview questions, Watch, Further reading).
- **Links must be real**: only link to YouTube videos and articles you have personally verified exist (open the URL, confirm the title/content — the YouTube oEmbed endpoint is a quick way to check a video URL is live). Never guess a video ID or URL. Prefer ByteByteGo, Gaurav Sen, Hello Interview, NeetCode, IBM Technology, and other reputable system design channels, but any reputable, verified source is fine.
- **No invented product direction**: this guide covers system design fundamentals for software engineers. If you want to propose a new topic or reorganize the curriculum, open an issue first.
- **Local preview**:
  ```bash
  bundle install
  bundle exec jekyll serve
  ```
  Then open `http://localhost:4000`.
- **Pull requests**: keep them focused (one topic or fix per PR) and describe what changed and why.
