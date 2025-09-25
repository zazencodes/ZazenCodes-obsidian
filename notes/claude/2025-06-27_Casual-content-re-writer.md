---
date: 2025-06-27
tags:
    - claude
hubs:
    - "[[prompts]]"
---

# 2025-06-27_Casual-re-writer

- Use with Claude
- Can swap out reference blog post for others here: https://galea.medium.com/

```
Here is some copy that I have written for my blog:

INSERT_CONTENT_HERE

However this isn't really in my style. Can you update it to read more naturally given my natural speaking tone. Here's an example below (content unrelated)

Personal knowledge graphs are rapidly growing in popularity as benefits emerge. There are lots to chose from, but here’s why I love Obsidian.
The Black Tusk in Garibaldi Park, BC
Building my own knowledge graph
Roam Research was the first tool like this that I learned about — their revolutionary graph approach to note taking blew my mind a bit.
I had already heard about the concept of your “mind garden” and loved the imagery that brought forth.
What really resonated with me was the idea of connecting thoughts and events in your live, and the benefit that could bring.
Obsidian
I chose this app because it looked like the best free alternative to Roam 💸
Next I watched this video about using Obsidian — I would recommend you do the same if you’re just getting started or switching to Obsidian from something else.
But then I learned a few things about Obsidian that made me fall in love:
* Awesome keyboard shortcuts
* Looks good, modern, super fast, good search
* It literally stores your notes as Markdown files on you filesystem! Here’s some of mine

>>> ls
00 Index.md
10 Health.md
2020-10-28.md
50 History.md
Obsidian - The Basics of Taking Notes - Effective Remote Work.md
Russians in Tlingit America The Battles of Sitka.md
Works with Google drive file sync
Vim keybindings 🤯 Vim ☑️
Really nice looking graph Plugins Using these is super easy.
Launch Obsidian and open / create a vault.
Open the settings by selecting the icon or pressing ⌘, then select the Plugin tab. Then you might want to do…
For Fold heading and Fold indent plugins ON. These guys are game changers. Being able to focus in on content blocks is really important for avoiding clutter as I work. Custom CSS You can find a big awesome list of CSS snippets that you might like. Here’s an example of how to use them.
Launch Obsidian and open / create a vault.
Open the settings by selecting the icon or pressing ⌘, then select the Plugin tab and switch the Custom CSS one to ON. Custom CSS ☑️
Go to the vault folder and create a file named obsidian.css then put the any CSS snippet you want in there. For example,
cd /path/to/my-vault
cat > obsidian.css
Then paste in the following snippet and press enter.

blockquote:before {
  font: 14px/20px italic Times, serif;
  content: "“";
  font-size: 3em;
  line-height: 0.1em;
  vertical-align: -0.4em;
}
blockquote p {
  display: inline;
}
/* Remove blockquote left margin */
blockquote {
  margin-inline-start: 0;
}
Here’s what this snippet does to your blockquotes ( > in Markdown)
Edit mode
Preview mode, default CSS
Default blockquote style
Preview mode, with custom CSS added
Custom CSS blockquote style (snippet above)
Conclusion
I look forward to building my knowledge graph by taking daily notes and connecting them all together, in a crazy virtual data mesh to represent my life — or at least a big part of it.
Hopefully Obsidian will turn out to be as good as it seems now in the long run. And hopefully I’ll maintain my current enthusiasm for using this medium. Only time will tell…
As always, thanks for reading. Now get back to your work! Your projects are missing you ;)
```
