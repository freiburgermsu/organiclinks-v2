## WordPress: www.organicdivision.org

- REST base: https://www.organicdivision.org/wp-json/wp/v2
- OrganicLinks page: ID **9586** (https://www.organicdivision.org/organicsyntheticfaculty/)
- Auth: always `curl --netrc-file ~/.config/organicdivision/netrc`. Never read, print or copy that file.
- Put working copies in `/tmp/organiclinks-wp/` (`mkdir -p` it first), never in this repo.
- Read the current HTML:
  `curl -s --netrc-file ~/.config/organicdivision/netrc "https://www.organicdivision.org/wp-json/wp/v2/pages/9586?context=edit&_fields=id,modified,content" > /tmp/organiclinks-wp/page.json`
  then `jq -r .content.raw /tmp/organiclinks-wp/page.json > /tmp/organiclinks-wp/page.html`. Keep `modified` from that read.
- Write it back:
  `jq -n --rawfile c /tmp/organiclinks-wp/page.html '{content:$c}' | curl -s --netrc-file ~/.config/organicdivision/netrc -H 'Content-Type: application/json' -X POST --data-binary @- https://www.organicdivision.org/wp-json/wp/v2/pages/9586 | jq '{id,modified,link}'`
- Before any write: show me the diff and wait for my OK. Then re-read `modified`; if it changed since your read, someone else edited the page, so stop and tell me.
- The page is hand-written HTML. wpautop is OFF, so paragraphs need their own `<p>` tags. Never send block-editor (`<!-- wp:... -->`) markup.
- Keep requests modest (no tight loops); the site has a rate limiter.
- Undo: every save creates a revision. If an update goes wrong, open the page in wp-admin, choose **Revisions**, and restore the previous one.

### Where the faculty list comes from

- The list is generated from the organiclinks-v2 Google Sheet (tab `Sheet1`, ID `1_9Y1I1gU1isijy3ljXOKA8C2NM5mbM5tqDEwI6tuWC8`). **Publish** in the sheet's menu commits it to GitHub, and the build writes `_site/organicsyntheticfaculty.html` on the `googlesheet` branch of khoivan88/organiclinks-v2.
- Change names, schools and links in the sheet, then Publish. An edit made only in WordPress is lost the next time a build is pasted in.
- To put the latest build on the page, fetch it from GitHub, not from this checkout (which can be out of date):
  `gh api -H "Accept: application/vnd.github.raw" "repos/khoivan88/organiclinks-v2/contents/_site/organicsyntheticfaculty.html?ref=googlesheet" > /tmp/organiclinks-wp/page.html`
  then diff it against the current page and follow the write steps above. Use `diff --strip-trailing-cr`: WordPress stores the page with CRLF line endings and the build uses LF.
