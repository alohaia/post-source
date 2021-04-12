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

1) asdf
2) asdfa
3) asdf
4) asdfas

unavailable

i. asdf
ii. asdfas

a) asdf
b) asdfas
c) asdf

A) asdfa
B) asdf
C) asdfasd

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

Term 1

:   Definition 1

Term 2 with *inline markup*

:   - Definition 2
    - Definition 3

        { some code, part of Definition 2 }

    Third paragraph of definition 2.

Term 1
  ~ Definition 1

Term 2
  ~ Definition 2a
  ~ Definition 2b

8. abbr

*[abbr]: hover this will show you something.

9. Look at the bottom[^hello]

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

{% img image /resources/Grammar-Testing/20020955_p0.jpg 750 960 '"20020955_p0" "20020955_p0"' %}


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

```xxx
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```
