---
{"dg-publish":true,"dg-path":"Garden/","permalink":"/garden//","created":"2024-06-15T09:22","updated":"2024-06-15T20:51"}
---

I've gone through several iterations of my website over the years, but this one is unique.

Previous versions of my site have been built with WordPress (usually) or a static site generator such as Hugo.

In the past, I have used a standard pages/blog site which requires me to keep information updated on my site (which is only useful for other people) and pressures me to write content. With this new version, I am able to use my favorite and daily used writing tool Obsidian to write articles for this site or just publish an already written Howto, so others/you can read it.

The current/new solution is based on [GitHub - oleeskild/obsidian-digital-garden](https://github.com/oleeskild/obsidian-digital-garden) with some small custom changes - like hosting on my own webspace instead of [Vercel](https://vercel.com). The full documentation on this awesome plugin can be found [here](https://dg-docs.ole.dev).

## Technical part
As mentioned on the Homepage, I use Obsidian as my second brain. All the information I want to keep and remember are written down in my personal vault. To share some of them, I had to copy the content to WordPress or a second folder to use it with Hugo. I tried some other Obsidian plugins to share my notes, but all of them had some things I would like to change or do differently. With the current solution, I can change almost every part of this page and then publish it easily using GitHub Actions.
### GitHub Build Pipeline
For publishing this site, I use a GitHub build pipeline that basically builds the static pages and pushes them to my Webspace hosted by AllInkl. I just added the rclone-part to the already existing pipeline.
```
name: Check for any build error

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-22.04

    strategy:
      matrix:
        node-version: [20.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm install
    - run: npm run build --if-present

    - name: Setup Rclone
      uses: AnimMouse/setup-rclone@v1
      with:
        rclone_config: |
          [allinkl-ftp]
          type = ftp
          host = ${{ secrets.RCLONE_CONF_SERVER }}
          user: ${{ secrets.RCLONE_CONF_USER }}
          pass: ${{ secrets.RCLONE_CONF_PASSWORD }}
          tls = false
          no_check_certificate: true
          port = 21
          concurrency: 5
          explicit_tls = true
          disable_tls13 = true
        disable_base64: true
    - run: 'rclone sync dist allinkl-ftp:/'
```