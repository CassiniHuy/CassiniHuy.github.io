# Landscape-F

A theme based on [Hexo]'s default theme [Landscape].

- [Preview](http://howiefh.github.io/hexo-theme-landscape-f/)
- [My Blog](http://howiefh.github.io/2014/04/20/hexo-optimize-and-my-theme-landscape-f/)

## Installation

### Install

``` bash
$ cd <your blog path>
$ git clone https://github.com/howiefh/hexo-theme-landscape-f.git themes/landscape-f
```

**Landscape requires Hexo 2.4 and above.** If you would like to enable the RSS, the [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed) plugin is also required.

### Enable

Modify `theme` setting in `_config.yml` to `landscape-f`.

```
theme: landscape-f
```

``` diff
_config.yml
- theme: some-theme
+ theme: landscape-f
```

### Update

``` bash
cd themes/landscape-f
git pull
```

## Configuration

``` yml
# Header
menu:
  Home: /
  Archives: /archives
rss: /atom.xml
github: https://github.com/howiefh
banner: images/banner.jpg

# Content
excerpt_link: Read More
fancybox: true
recent_posts_limits: 5

# Sidebar
sidebar: right
widgets:
- about_me
- category
- tag
- tagcloud
- archive
- calendar
- recent_posts
- links

# Miscellaneous
google_analytics:
favicon: /favicon.ico
twitter:
google_plus:
fb_admins:
fb_app_id:

# Toc
toc:
  article: true   ## show contents in article.
  aside: true     ## show contents in aside.

# Scroll to top
go_top: true

# baidu share
baidushare: true

# blogroll
links:
- name: 404 page
  link: http://yibo.iyiyun.com/js/yibo404/key/1

# about me
about_me:
  title: ABOUT ME
  gravatar: a@abc.com
  avatar: /images/github.jpg
  texts:
  - Hi,I'm howiefh.

# display updated
display_updated: true

# busuanzi
busuanzi: true

# calendar
calendar:
  language: zh-CN
  options:
    months: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December']
    dayOfWeekShort: ['S', 'M', 'T', 'W', 'T', 'F', 'S']
    dayOfWeek: ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']
    postsMonthTip: 'Posts published in LMM yyyy'
    titleFormat: 'yyyy LMM'
    titleLinkFormat: 'archives/yyyy/MM/'
    headArrows: {previous: '<span class="cal-prev"></span>', next: '<span class="cal-next"></span>'}
    footArrows: {previous: ' ', next: ' '}
    weekOffset: 0
    root: '/'
    calendarSingle: false
    calendarRoot: 'calendar/'
    calendarUrl: 'calendar.json'
```

- **menu** - Navigation menu
- **rss** - RSS link
- **github** - Github link
- **banner** - Path of title banner image of page top
- **excerpt_link** - "Read More" link at the bottom of excerpted articles. `false` to hide the link.
- **fancybox** - Enable [Fancybox]
- **recent_posts_limits** - How many posts display in Home page.
- **sidebar** - Sidebar style. You can choose `left`, `right`, `bottom` or `false`.
- **widgets** - Widgets displaying in sidebar
- **google_analytics** - Google Analytics ID
- **favicon** - Favicon path
- **twitter** - Twiiter ID
- **google_plus** - Google+ ID
- **toc** - Show toc in article or sidebar
- **go_top** - Go to top
- **baidushare** - Show baidu share in post
- **links** - Links displaying in sidebar
- **about_me** - About me displaying in sidebar. The gravatar is your gravatar e-mail. You can choose gravatar or avatar to show your picture.
- **display_updated** - Display updated in article footer
- **busuanzi** - Display PV and UV
- **calendar** - Options of calendar widget
    - language - See [languages.js]. Priority: theme's `_config.yml` `>` hexo's `_config.yml` `>` [calendar.js]'s default option.
    - options - Priority of months, dayOfWeekShort, dayOfWeek, ostsMonthTip and titleFormat : theme's `_config.yml` `>` [languages.js] `>` [calendar.js]'s default option. Priority of calendarSingle and calendarRoot: hexo's `_config.yml` `>` theme's `_config.yml` `>` [calendar.js]'s default option. Priority of other options: theme's `_config.yml` `>` [calendar.js]'s default option.

## Features

### Fancybox

Landscape-F uses [Fancybox] to showcase your photos. You can use Markdown syntax or fancybox tag plugin to add your photos.

```
![img caption](img url)

{% fancybox img_url [img_thumbnail] [img_caption] %}
```

### Sidebar

You can put your sidebar in left side, right side or bottom of your site by editing `sidebar` setting.

Landscape-F provides 8 built-in widgets:

- category
- tag
- tagcloud
- archive
- recent_posts
- about_me
- calendar: Dates appear links if there are posts for that day when use [hexo-generator-calendar].
- links

All of them are enabled by default. You can edit them in `widget` setting.

[Fancybox]: https://github.com/fancyapps/fancyBox
