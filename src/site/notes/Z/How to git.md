---
{"dg-publish":true,"dg-path":"HowTo/git","permalink":"/how-to/git/","tags":["git","notes/evergreen"],"dgShowToc":true,"noteIcon":"tree","created":"2022-11-11 11:18","updated":"2024-11-29 22:26"}
---

## Defining a commit Message Convention

```yaml
type(scope): subject
<body>
<footer>
```

### 1. Type of commit

```yaml
feat:     The 'new' feature being added to a particular application
fix:      A bug fix
style:    Feature and updates related to styling
refactor: Refactoring a specific section of the codebase
test:     Everything related to testing
docs:     Everything related to documentation
chore:    Regular code maintenance
```

### 2. Scope (optional)

A scope MUST consist of a noun describing a section of the codebase affected by the changes (or simply the epic name) surrounded by parenthesis. Example:

```yaml
feat(claims)
fix(orders)
```

### 3. Subject

Short description of the applied changes.
- Limit the subject line to 50 characters
- Your commit message should not contain any whitespace errors or punctuation marks
- Do not end the subject line with a period
- Use the present tense or imperative mood in the subject line

```yaml
feat(claims): add claims detail page
fix(orders): validation of custom specification
```

### 4. Body (Optional)

Use the body to explain what changes you have made and why you made them.
- Separate the subject from the body with a blank line
- Limit each line to 72 characters
- Do not assume the reviewer understands what the original problem was, ensure you add it
- Do not think your code is self-explanatory

```yaml
refactor!: drop support for Node 6
BREAKING CHANGE: refactor to use JavaScript features not available in Node 6.
```

### 5. Footer (Optional)

Use this section to reference issues affected by the code changes or comment to another developers or testers.

```yaml
fix(orders): correct minor typos in code
See the issue for details on typos fixed.
Reviewed-by: @John Doe
Refs #133
```

### SemVer Tips
(SemVer = Semantic Versioning)

**fix:** a commit of the _type_ _fix_ patches a bug in your codebase (this correlates with **_PATCH_** in semantic versioning).

**feature:** a commit of the _type_ _feat_ introduces a new feature to the codebase (this correlates with **_MINOR_** in semantic versioning).

**BREAKING CHANGE:** a commit that has a footer _BREAKING CHANGE:_, or appends a _!_ after the type/scope, introduces a breaking API change (correlating with **_MAJOR_** in semantic versioning). A BREAKING CHANGE can be part of commits of any _type_.

### Force this messages

You can force this type of messages by adding the following pattern to your gitlab-projects push rules (Settings → Repository → Push rules):

```
^(feat|fix|build|chore|docs|style|refactor|perf|test)(\(.*\))?!?: (.+[^.\r\n])([\r\n]+(.+[\r\n]+)+)?$
```