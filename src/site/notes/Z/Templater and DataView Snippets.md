---
{"dg-publish":true,"dg-path":"Notes/templater_and_dataview_snippets","permalink":"/notes/templater-and-dataview-snippets/","tags":["notes/fern"],"noteIcon":"fern","created":"2024-06-28 09:17","updated":"2024-09-08 22:27"}
---

Here are some Templaterand DataView-Snippets I use in Obsidian.
## Sources
- [reddit](https://www.reddit.com/r/ObsidianMD/)
- [Zach Young](https://zachyoung.dev/posts/templater-snippets/)

## Snippets
The Snippets are not sorted or in any particular order. Some of them are written by my self, others are copied over.
### Daily notes block that only shows on certain days of the week
The following daily notes template has two blocks. The first block will only show on daily notes that land on a Monday. The second block will show on every daily note.
 ```js
<%* if (tp.date.now("dddd", 0, tp.file.title, "YYYY-MM-DD") === "Monday") { %>
Block that only shows on Mondays
<%* } %>
Block that shows every day
```

### Daily note links to yesterday and tomorrow
We can pass in a format, offset, reference, and reference format to [tp.date.now](https://silentvoid13.github.io/Templater/internal-functions/internal-modules/date-module.html#tpdatenowformat-string--yyyy-mm-dd-offset-numberstring-reference-string-reference_format-string) to create links to yesterday and tomorrow relative to the active daily note, instead of being relative to the actual current date.

```js
[[ <% tp.date.now("YYYY-MM-DD", -1, tp.file.title, "YYYY-MM-DD") %>|yesterday ]]

[[ <% tp.date.now("YYYY-MM-DD", 1, tp.file.title, "YYYY-MM-DD") %>|tomorrow ]]
```

### Escaping frontmatter in a template
If you don’t want your frontmatter in your templates to be parsed by Obsidian or any Obsidian plugins (like Dataview), then you can escape your frontmatter by wrapping the starting and ending `---` with Templater tags to escape it.

```js
<% "---" %>
date: <% tp.date.now() %>
key: value that is not frontmatter because the --- is escaped
<% "---" %>
// Rest of template below
```

### Get result from Modal Forms
```js
<%*
const values = {
    name: tp.file.title,
};

const result = await tp.user.openForm('add_person', { values: values });

let personName = result.get("name")
await tp.file.rename(personName)

let personTags = result.get("tags")
let personMail = result.get("email")
let personCompany = result.get("company")
let personPartner = result.get("partner")
-%>
```
 Here is the required snippet. Save as `openForm.js` in your templater snippets folder.
```js
const modalForm = app.plugins.plugins.modalforms.api;
module.exports = (formName, options) => modalForm.openForm(formName, options);
```

### Embed YouTube video
```js
<%*
const url = await tp.system.clipboard()
const response = await fetch(`https://youtube.com/oembed?url=${url}&format=json`)
const data = await response.json()

const title = data.title.replaceAll("", "").replaceAll('"', '').replaceAll("\\", "").replaceAll("/", "").replaceAll("<", "").replaceAll(">", "").replaceAll(":", "").replaceAll("|", "").replaceAll("?", "")
const author = data.author_name
const author_url = data.author_url
const html = data.html
const thumbnail = data.thumbnail_url

const regex = /(?:https?:\/\/)?(?:www\.)?(?:youtube\.com\/(?:[^\/\n\s]+\/\S+\/|(?:v|e(?:mbed)?)\/|\S*?[?&]v=)|youtu\.be\/)([a-zA-Z0-9_-]{11})/
const m = regex.exec(url)
-%>
> [!youtube]- YouTube
> **Title:** <% title %>
> **Channel:** [<% author %>](<% author_url %>)
> **Link:** [View on YouTube](https://www.youtube.com/watch?v=<% m[1] %>)
> <iframe src="https://www.youtube-nocookie.com/embed/<% m[1] %>?vq=hd1080&modestbranding=1&rel=0&iv_load_policy=3" width="569" height="317" frameborder="0" style="margin: 0 auto; display: block;"></iframe>
```

### Link to Parent with kids
```js
<%*
const result = await tp.user.openForm('add_family');

let parent = result.get("parent")
-%>
 und [[]]
```