---
title: Grammar Testing
date: 2021-03-15 11:54:13
mathjax: true
comments: false
---

@[toc]

<!-- more -->

## reason

I use **[hexo-theme-next](https://github.com/iissnan/hexo-theme-next)** for my blog, and I met a problem same with [#826](https://github.com/iissnan/hexo-theme-next/issues/826). 

I wrote a plugin [**hexo-renderer-markdown-it-plus**](https://github.com/CHENXCHEN/hexo-renderer-markdown-it-plus) for fix it,and this article is a demo for `hexo-renderer-markdown-it-plus`

![](hello)

![aloha](hello)

{
    { 
    }  
}

1. asdf
2. asdfas
3. a
4. a
5. a
6. a
7. a
8. a
9. a
10. a
11. a
12. a
13. a
14. a

* asdfas
* asdfasdf
+ asdfasd
+ asdfasd

- asdf
- asdfasdf

- [ ] unchecked
- [X] checked

| Name    |   Size | Cost |
|---------|-------:|-----:|
| bre     |    123 |  4.5 |
| laun    |    105 |   21 |
| laun    |    593 |   10 |
| Total   | 1396.0 | 62.0 |
| Average |  349.0 | 15.5 |
<!-- tmf: $4,2=Sum(2:-1) ; $4,3=Sum(2:-1) -->
<!-- tmf: $5,2=Average(2:-2) ; $5,3=Average(2:-2) -->

1) one
2) two
3) three
4) four

unavailable

i. asdf
ii. asdfas

a) asdf
b) asdfas
c) asdf

A) asdfa
B) asdf
C) asdfasd

Term 1
:   Definition 1
:   Definition 2
At The Begin

Term 2 with *inline markup*
: Characteristics
    - Characteristic 1
        ```cpp test.cpp
        int main(){
            return 0;
        }
        ```
    - Characteristic 2

Term 1
    : Definition 2

    : Definition 2
    : Definition 2

The hexo default [hexo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked)
do not support LaTex parser, you must referer external link to parse Latex grammar to 
html(That's what `hexo-theme-next` did, `hexo-theme-next` use `mathjax`), 
and the `mathjax` and `hexo-renderer-marked` will cause some problem:

1. `_` parse error, you must change `x_i` to `x\_i`(This problem had been fixed when i test.)
2. do not support lines grammar, example below:

```latex
$$
H=-\sum_{i=1}^N (\sigma_{i}^x \sigma_{i+1}^x+g \sigma_{i}^z)
$$

$$
f(n) = \begin{cases}
 \frac{n}{2},
 & \text{if } n\text{ is even}
 \\ 3n+1, & \text{if } n\text{ is odd}
 \end{cases}
$$

$$
\begin{aligned}
\nabla \times \vec{\mathbf{B}} -\, \frac1c\, \frac{\partial\vec{\mathbf{E}}}{\partial t} & = \frac{4\pi}{c}\vec{\mathbf{j}} \\   
\nabla \cdot \vec{\mathbf{E}} & = 4 \pi \rho \\
\nabla \times \vec{\mathbf{E}}\, +\, \frac1c\, \frac{\partial\vec{\mathbf{B}}}{\partial t} & = \vec{\mathbf{0}} \\
\nabla \cdot \vec{\mathbf{B}} & = 0 \end{aligned}
$$
```

## hexo-renderer-markdown-it-plus

[**hexo-renderer-markdown-it-plus**](https://github.com/CHENXCHEN/hexo-renderer-markdown-it-plus) support lines grammer for $\KaTeX$(Don't worry, it's grammer same with Latex).

**[hexo-renderer-markdown-it-plus](https://github.com/CHENXCHEN/hexo-renderer-markdown-it-plus)**

The latex code of above will be display below:


```cpp
int main()
{
    return 0;
}
```


$$
H=-\sum_{i=1}^N (\sigma_{i}^x \sigma_{i+1}^x+g \sigma_{i}^z)
$$

^link^

$$
f(n) = \begin{cases}
 \frac{n}{2},
 & \text{if } n\text{ is even}
 \\ 3n+1, & \text{if } n\text{ is odd}
 \end{cases}
$$

$$
\begin{aligned}
\nabla \times \vec{\mathbf{B}} -\, \frac1c\, \frac{\partial\vec{\mathbf{E}}}{\partial t} & = \frac{4\pi}{c}\vec{\mathbf{j}} \\   
\nabla \cdot \vec{\mathbf{E}} & = 4 \pi \rho \\
\nabla \times \vec{\mathbf{E}}\, +\, \frac1c\, \frac{\partial\vec{\mathbf{B}}}{\partial t} & = \vec{\mathbf{0}} \\
\nabla \cdot \vec{\mathbf{B}} & = 0 \end{aligned}
$$

BTW, i bundle some plugins, example below:

1. H~2~0
2. x^2^
3. ++inserted++, ~~Delete~~
4. $\KaTeX$, example $x_i + y_i = z_i$ and $y_i + z_i = 10$
5. ​:smile: :joy: :stuck_out_tongue:
6. toc&anchor(do not explain this)
7. deflist
8. *italic* **bold**

[deflist syntax](https://pandoc.org/MANUAL.html#definition-lists)

Term 1
:   Definition 1
:   asdfasdf

Term 2 with *inline markup*
: Characteristics
    - Definition 2
    - Definition 3

        { some code, part of Definition 2 }
    asdf

Term 1
    ~ Definition 2
    ~ Definition 2

8. abbr

*[abbr]: hover this will show you something.

9. Look at the bottom [^hello]

[^hello]: footnote

10. ==mark==, `==mark==`


**The markdown code show as below:**

```markdown
1. H~2~0
2. x^2^
3. ++inserted++, ~~Delete~~
4. $\KaTex$, example $x_i + y_i = z_i$ and $y_i + z_i = 10$
5. :smile: :joy: :stuck_out_tongue:
6. toc&anchor(do not explain this)
7. deflist

Term 1

:   Definition 1

Term 2 with *inline markup*

:   Definition 2

        { some code, part of Definition 2 }

    Third paragraph of definition 2.

8. abbr

*[abbr]: hover this will show you something.

9. Look at the bottom[^hello]

   [^hello]: footnote
10. ==mark==, `==mark==`
```

## Tag-plugins

Docs: https://hexo.io/zh-cn/docs/tag-plugins.html

### Link
{% link Baidu https://www.baidu.com https://www.google.com ExtBaidu %}

### Code Block

{% codeblock test.cpp lang:cpp https://www.aloha.org.cn "Aloha' Blog" first_line:46 mark:1-3,49-56 %}
GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "LearnOpenGL", glfwGetPrimaryMonitor(), NULL);
if (window == NULL)
{
    std::cout << "Failed to create GLFW window" << std::endl;
    :smile deep water:
    glfwTerminate();
    return -1;
}
glfwMakeContextCurrent(window);
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
glfwSetCursorPosCallback(window, mouse_callback);
glfwSetScrollCallback(window, scroll_callback);

// tell GLFW to capture our mouse
glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);
{% endcodeblock %}

```cpp test2.cpp https://www.aloha.org.cn Aloha' Blog
cout << "Hello world!" << endl;
```

### Include Code

{% include_code proj6-camera.cpp lang:cpp from:307 to:325 proj6-camera.cpp %}


### Iframe

{% iframe https://theme-next.iissnan.com/tag-plugins.html 800 600 %}

### Image

{% img right "/images/aloha.gif" 200 "aloha'aloha'" %}

- `img.right`、`img.left` 会分别在右、左边浮动显示。
- `img` 的高度不超过 `35em`。

### Pull Quote

{% pullquote %}
content
What's this?
{% endpullquote %}

### Reference

{% post_link The-TTY-Demystified "The <b>TTY</b> Demystified" false %}

{% post_link The-TTY-Demystified false %}

{% post_link The-TTY-Demystified "The TTY Demystified" false %}

### Asset Resource

> 需要将 `_config.yml` 文件中的 `post_asset_folder` 选项设为 `true` 来打开。
> > asfasdf

```xxx
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```


### NexT tags

#### Call Out

{% note default default@default %}
default
{% endnote %}

{% note primary primary@primary %}
primary
{% endnote %}

{% note success success@success %}
success
{% endnote %}

{% note info info@info %}
info
{% endnote %}

{% note warning warning@warning %}
warning
{% endnote %}

{% note danger danger@danger %}
danger
{% endnote %}

#### Button

`button` or `btn`

sepqrate: ,

{% btn https://cmake.org/cmake/help/v3.20/command/target_include_directories.html, target_include_directories, bras, Bars %}

```javascript
'use strict';

function postButton(args) {
  args = args.join(' ').split(',');
  let url   = args[0];
  let text  = args[1] || '';
  let icon  = args[2] || '';
  let title = args[3] || '';

  if (!url) {
    hexo.log.warn('URL can NOT be empty.');
  }

  text = text.trim();
  icon = icon.trim();
  icon = icon.startsWith('fa') ? icon : 'fa fa-' + icon;
  title = title.trim();

  return `<a class="btn" href="${url}"${title.length > 0 ? ` title="${title}"` : ''}>
            ${icon.length > 0 ? `<i class="${icon}"></i>` : ''}${text}
          </a>`;
}

hexo.extend.tag.register('button', postButton, {ends: false});
hexo.extend.tag.register('btn', postButton, {ends: false});
```

#### caniUse

```javascript
'use strict';

function caniUse(args) {
  args = args.join('').split('@');
  var feature = args[0];
  var periods = args[1] || 'current';

  if (!feature) {
    hexo.log.warn('Caniuse feature can NOT be empty.');
    return '';
  }

  return `<iframe data-feature="${feature}" src="https://caniuse.bitsofco.de/embed/index.html?feat=${feature}&periods=${periods}&accessible-colours=false" frameborder="0" width="100%" height="400px"></iframe>`;
}

hexo.extend.tag.register('caniuse', caniUse);
hexo.extend.tag.register('can', caniUse);
```

#### Center Quote

`centerquote` or `cq`

{% cq %}
So we beat on, boats against the current, borne back ceaselessly into the past.<br/>
我们奋力向前，宛如逆水行舟，与激流抗争勇进，直至淹没入岁月长河。
{% endcq %}

#### Group Pictures

```javascript
'use strict';

var LAYOUTS = {
  2: {
    1: [1, 1],
    2: [2]
  },
  3: {
    1: [3],
    2: [1, 2],
    3: [2, 1]
  },
  4: {
    1: [1, 2, 1],
    2: [1, 3],
    3: [2, 2],
    4: [3, 1]
  },
  5: {
    1: [1, 2, 2],
    2: [2, 1, 2],
    3: [2, 3],
    4: [3, 2]
  },
  6: {
    1: [1, 2, 3],
    2: [1, 3, 2],
    3: [2, 1, 3],
    4: [2, 2, 2],
    5: [3, 3]
  },
  7: {
    1: [1, 2, 2, 2],
    2: [1, 3, 3],
    3: [2, 2, 3],
    4: [2, 3, 2],
    5: [3, 2, 2]
  },
  8: {
    1: [1, 2, 2, 3],
    2: [1, 2, 3, 2],
    3: [1, 3, 2, 2],
    4: [2, 2, 2, 2],
    5: [2, 3, 3],
    6: [3, 2, 3],
    7: [3, 3, 2]
  },
  9: {
    1: [1, 2, 3, 3],
    2: [1, 3, 2, 3],
    3: [2, 2, 2, 3],
    4: [2, 2, 3, 2],
    5: [2, 3, 2, 2],
    6: [3, 2, 2, 2],
    7: [3, 3, 3]
  },
  10: {
    1: [1, 3, 3, 3],
    2: [2, 2, 3, 3],
    3: [2, 3, 2, 3],
    4: [2, 3, 3, 2],
    5: [3, 2, 2, 3],
    6: [3, 2, 3, 2],
    7: [3, 3, 2, 2]
  }
};

function groupBy(group, data) {
  var r = [];
  for (let count of group) {
    r.push(data.slice(0, count));
    data = data.slice(count);
  }
  return r;
}

var templates = {

  dispatch: function(pictures, group, layout) {
    var rule = LAYOUTS[group] ? LAYOUTS[group][layout] : null;
    return rule ? this.getHTML(groupBy(rule, pictures)) : templates.defaults(pictures);
  },

  /**
   * Defaults Layout
   *
   * □ □ □
   * □ □ □
   * ...
   *
   * @param pictures
   */
  defaults: function(pictures) {
    var ROW_SIZE = 3;
    var rows = pictures.length / ROW_SIZE;
    var pictureArr = [];

    for (var i = 0; i < rows; i++) {
      pictureArr.push(pictures.slice(i * ROW_SIZE, (i + 1) * ROW_SIZE));
    }

    return this.getHTML(pictureArr);
  },

  getHTML: function(rows) {
    var rowHTML = rows.map(row => {
      return `<div class="group-picture-row">${this.getColumnHTML(row)}</div>`;
    }).join('');

    return `<div class="group-picture-container">${rowHTML}</div>`;
  },

  getColumnHTML: function(pictures) {
    var columnWidth = 100 / pictures.length;
    var columnStyle = `style="width: ${columnWidth}%;"`;
    return pictures.map(picture => {
      return `<div class="group-picture-column" ${columnStyle}>${picture}</div>`;
    }).join('');
  }
};

function groupPicture(args, content) {
  args = args[0].split('-');
  var group = parseInt(args[0], 10);
  var layout = parseInt(args[1], 10);

  content = hexo.render.renderSync({text: content, engine: 'markdown'});

  var pictures = content.match(/<img[\s\S]*?>/g);

  return `<div class="group-picture">${templates.dispatch(pictures, group, layout)}</div>`;
}

hexo.extend.tag.register('grouppicture', groupPicture, {ends: true});
hexo.extend.tag.register('gp', groupPicture, {ends: true});
```

#### Label

sepqrate: @

{% label default@《了不起的盖茨比》 %}

- default
- primary
- info
- success
- danger
- warning

#### Mermaid

```javascript
'use strict';

function mermaid(args, content) {
  return `<pre class="mermaid" style="text-align: center;">
            ${args.join(' ')}
            ${content}
          </pre>`;
}

hexo.extend.tag.register('mermaid', mermaid, {ends: true});
```

#### Pdf

```javascript
'use strict';

function pdf(args) {
  let theme = hexo.theme.config;
  return `<div class="pdfobject-container" data-target="${args[0]}" data-height="${args[1] || theme.pdf.height}"></div>`;
}

hexo.extend.tag.register('pdf', pdf, {ends: false});
```

#### Tabs

`tabs` or `subtabs` or `subsubtabs`

https://blog.zhujian.life/posts/5213c80b.html

{% tabs Unique name, 1 %}
<!-- tab Top@bars -->

Top content 1

<!-- endtab -->

<!-- tab Top@bars -->

Top content 2

<!-- endtab -->

<!-- tab Top@bars -->

Top content 3

{% subtabs Unique name 2, 1 %}
<!-- tab Sub@bars -->

Sub content 1

<!-- endtab -->

<!-- tab Sub@bars -->

Sub content 2

<!-- endtab -->
{% endsubtabs %}

{% subtabs Unique name 3, 2 %}
<!-- tab Sub@bars -->

Sub content 1

<!-- endtab -->

<!-- tab Sub@bars -->

Sub content 2

<!-- endtab -->
{% endsubtabs %}

<!-- endtab -->

{% endtabs %}
