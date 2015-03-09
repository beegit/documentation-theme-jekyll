---
title: Search configuration
permalink: /search_configuration/
---

The search is configured through the search.json file in the root directory. Take a look at that code if you want to change what content is included. 

The search is a simple search that looks at content in pages or posts. It looks at both titles and bodies. However, the search doesn't work like google -- you can't hit return and see a list of results on the search results page, with the keywords in bold. Instead, this search shows a list of page titles that contain keyword matches. It's fast, but simple.


## Excluding pages form search

If you want to exclude a page from search add `search: exclude` in the frontmatter. By default, every page is included. The configuration file specifies this default in the frontmatter of every page:

```
defaults:
  -
    scope:
      path: ""
      type: "pages"
    values:
      layout: "page"
      comments: true
      search: include
  -
    scope:
      path: ""
      type: "posts"
    values:
      layout: "post"
      comments: true
      search: include
 ```
This is why you need to add `search: exclude` if you want do not want a page to appear in search.

## Troubleshooting search

You should exclude any files from search that you don't want appearing in the search results. For example, if you have a tooltips.json file, don't include it, as the formatting will be invalid in the search. 

If any formatting in the search.json file is invalid (in the build), search won't work. You'll know that search isn't working if no results appear when you start typing in the search box.

If this happens, go directly to the search.json file in your browser, and then copy the content. Go to a [JSON validator](http://jsonlint.com/) and paste in the content. Look for the line causing trouble. Edit the file to either exclude it from search or fix the syntax so that it doesn't invalidate the JSON. 

The search.json file already strips out content that would otherwise make the JSON invalid:

{% raw %}

```
      "body": "{{ page.content | strip_html | strip_newlines | replace: '\', '\\\\' | replace: '"', '\\"' }}",
```
{% endraw %}

However, it's possible that the formatting may not account for all the scenarios that would invalidate the JSON.
