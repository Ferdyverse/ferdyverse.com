---
{"dg-publish":true,"dg-path":"Notes/obsidian_css_snippets","permalink":"/notes/obsidian-css-snippets/","tags":["obsidian","css","📝/🌿"],"noteIcon":"fern","created":"2024-04-23 17:43","updated":"2024-07-06 15:45"}
---

## Plugin: Style Settings
The [Plugin: Style Settings](https://github.com/mgmeyers/obsidian-style-settings) can be used to create style settings that can be altered in the UI. To use them, you have to define the UI-elements in a css-file:

```css
/* @settings

name: Brazier85
id: brazier85
settings:
    - 
        id: header-progressbar-main
        title: Progressbars
        description: "Settings for the Mental Progress Bars"
        type: heading
        collapsed: true
        level: 1
    - 
        id: header-progressbar-all
        title: General Settings
        type: heading
        level: 2
    - 
        id: progressbar-height
        title: Bar height (px)
        type: variable-number
        opacity: false
        format: px
        default: 22
    - 
        id: header-progressbar-sleep
        title: Sleep Bar
        type: heading
        level: 2
    - 
        id: bar-sleep-background
        title: Background Color
        type: variable-color
        opacity: false
        format: hex
        alt-format:
            -
                id: accent-rgb
                format: rgb
        default: '#3a69df'
    - 
        id: header-progressbar-sport
        title: Sport Bar
        type: heading
        level: 2
    - 
        id: bar-sport-background
        title: Background Color
        type: variable-color
        opacity: false
        format: hex
        alt-format:
            -
                id: accent-rgb
                format: rgb
        default: '#d2691e'
    - 
        id: header-progressbar-mental
        title: Mental Bar
        type: heading
        level: 2
    - 
        id: bar-mental-background
        title: Background Color
        type: variable-color
        format: hex
        alt-format:
            -
                id: accent-rgb
                format: rgb
        default: '#228b22'
    - 
        id: header-spaceing
        title: Spaceing
        description: "Spacing settings"
        type: heading
        level: 1
    - 
        id: heading-spacing
        title: Heading Spaceing (em)
        description: Top margin above Headlines
        type: variable-number
        format: em
        default: 1
    - 
        id: p-spacing
        title: Paragraph spacing (rem)
        description: Spacing between blocks
        type: variable-number
        format: rem
        default: 1.75

*/
```

Now you can use the defined settings in your CSS file:

```css
/* We have to initiate the variables so there are values when
   nothing is entered in the UI */
body {
  --progressbar-height: 22px;
  --bar-sleep-background: #3a69df;
  --bar-sport-background: #d2691e;
  --bar-mental-background: #228b22;
}

/* Progressbars */

.bar-sleep .mb-progress-bar-progress {
  background-color: var(--bar-sleep-background);
}

.bar-sport .mb-progress-bar-progress {
  background-color: var(--bar-sport-background);
}

.bar-mental .mb-progress-bar-progress {
  background-color: var(--bar-mental-background);
}

/* Progress bar height */
.mb-progress-bar-input {
  height: var(--progressbar-height);
}
```
## Icons
### Callout
Obsidian has [Lucide Icons](https://lucide.dev/icons/) included. You can use them in your CSS-files like this:

```css
.callout[data-callout="brain"] {  
    --callout-icon: lucide-brain-cog;  
    --callout-color: 166, 227, 161;
}
```
**Note:** Try the Plugin `Callout Manager` for a better/easier way to define callouts and to style them.
### Sidebar
Here is a snipped to create icons in your sidebar in front of some notes:

```css
/* Default icon for all folders and files */
.nav-file-title-content::before { content: "📝️ "; }
.nav-folder-title-content::before { content: '📂 '; }

/* For a file */
.nav-file-title-content[data-link-data-href="Research"]::before { content: '🖥️ '; }

/* For a folder */
.nav-folder-title[data-path="People"] .nav-folder-title-content::before { content: "👥 "; font-size: 1.3em; }
```

For more complex definitions, I recommend the combination of `Supercharged Links` and `Style Settings`.
## Available CSS-Classes
This list is for me, so I do not forget about the classes I have available in my vault. Some of them are from the minimal-theme

| CSS-Class                               | Function                                            |
| --------------------------------------- | --------------------------------------------------- |
| `col-alt`                               | Add striped background to alternating table columns |
| `col-lines`                             | Add borders between table columns                   |
| `daily`                                 | Changes for Daily notes                             |
| `dashboard`                             | Dashboard styling for the home view                 |
| `dv-equal-columns`                      | DataView Table Columns are all the same size        |
| `img-grid`                              | Activate image grids                                |
| `no-title`                              | Hides the title/headline                            |
| `row-alt`                               | Add striped background to alternating table rows    |
| `row-lines`                             | Add borders between table rows                      |
| `show_properties`                       | Unhide the properties view in reading mode          |
| `table-100`, `img-100`, `iframe-100`    | Fill 100% of the pane width                         |
| `table-lines`                           | Add borders around all table cells                  |
| `table-max`, `img-max`, `iframe-max`    | Fill the max line width (default 88%)               |
| `table-nowrap`                          | Disable line wrapping in table cells                |
| `table-numbers`                         | Add row numbers to tables                           |
| `table-small`                           | Use small font size in tables                       |
| `table-tabular`                         | Use tabular figures in tables                       |
| `table-tiny`                            | Use tiny font size in tables                        |
| `table-wide`, `img-wide`, `iframe-wide` | Fill the wide line width                            |
| `table-wrap`                            | Force line wrapping in table cells                  |
## Links
You can find all the mentioned plugins here: [[Z/Obsidian Plugins\|Obsidian Plugins]].
Some of the CSS changes are from [here](https://github.com/r-u-s-h-i-k-e-s-h/Obsidian-CSS-Snippets)