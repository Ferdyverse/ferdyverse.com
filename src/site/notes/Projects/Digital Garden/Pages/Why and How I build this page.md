---
{"dg-publish":true,"permalink":"/projects/digital-garden/pages/why-and-how-i-build-this-page/"}
---

I've gone through several iterations of my website over the years, but this one is unique.

Previous versions of my site have been built with WordPress (usually) or a static site generator such as Hugo.

In the past I have used a standard pages/blog site which requires me to keep information updated on my site (which is only useful for other people) and pressures me to write content. With this new version, I am able to use my favorite and daily used writing tool Obsidian to write articles for this site or just publish a written HowTo, so others can use it.

For publishing this site, I use a GitHub build pipeline that basically builds the static pages and pushes them to my Webspace hosted by AllInkl.
### GitHub Build Pipeline
I just added the rclone-part to the already existing pipeline.
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