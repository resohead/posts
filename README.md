A collection of markdown files for my blog.

## Posts
- [x] store markdown posts in separate dedicated repository (here)
- [x] posts can be stored in directories with related assets (images, videos, etc.)
- [x] frontmatter should contain article details

## Deployment
- [x] posts should be
- [x] blog should statically generated
- [x] trigger Cloudflare page builds when SvelteKit blog repository changes
- [x] use Cloudflare deploy hooks to trigger new build when this markdown repo changes

## Features
- [x] fuzzy search
- [x] posts should be searchable
- [x] filter by tags
- [x] automatic table of contents
- [x] mobile table of contents slideover
- [x] edit on GitHub link
- [x] automatic (or curated) related posts
- [x] automatic series (multiple markdown posts in same directory)
- [x] dark mode

## Development (in Blog repository)
- [x] code blocks should be rendered in a dedicated component
  - [x] grouped code
  - styles
    - [x] individual
    - [x] grouped/tabbed
    - [ ] editor/tree view
  - [x] metadata on code fence, i.e. ```js file="index.js" comment="related note here"
  - [ ] frontmatter metadata inside code fence
  - [x] copy button
  - [x] syntax highlighting using Prism
  - [x] diff highlighting
  - [x] code block comments
  - [ ] links (e.g. code screenshot, github gist link, tailwind play url)
  - embeds using custom link tag
    - [-] youtube `[!youtube](https://youtube.com/<video-id>)`
    - [-] GitHub gists `[!gist](https://gist.github.com/<gist-id>)`
    - [-] Twitter `[!tweet](https://twitter.com/<tweet-id>)`
- custom callout component using quote syntax
  - support different syntax
    - [x] `> :warning: This is a warning` (custom style)
    - [ ] `> **Warning** This is a warning` (GitHub style)
    - [ ] `::: warning \n This is a warning \n:::` (Virepress style)
  - types
    - [x] quote
    - [x] information
    - [x] tip
    - [x] danger
    - [x] warning
    - [x] success