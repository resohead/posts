A collection of markdown files for my blog.

SveleteKit Blog:

- [-] store markdown posts in dedicated repository
- [-] site should be prerendered
- [ ] search page should not be pre-rendered
- [ ] search page should use Algolia
- [ ] Images should be copied to static assets folder
- [ ] Markdown should be pre-processed to convert relative image URLs to
  - [ ] assets url (if stored locally), or
  - [ ] the github url
- [-] create a JSON endpoint of files and content (for Algolia search index)
- [ ] post-build send data to Algolia index
- [ ] use Cloudflare deploy hooks to trigger new build when markdown repo changes