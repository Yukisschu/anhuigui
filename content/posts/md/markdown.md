+++
title = 'Learn Basic Markdown'
date = 2023-12-07T17:36:18+01:00
draft = false
slug = 'markdown'
tags = ['WIP', 'draft', 'markdown', 'website building', 'summary']
series_order = 3
+++


{{< lead >}}
Markdown is super flexible that comes in handy for many using cases. You can pick up the basics really soon and start creating your first web page.
{{< /lead >}}

Markdown is a lightweight markup language for creating formatted text using a plain-text editor [^1]. Here's the [official website](https://daringfireball.net/projects/markdown/syntax) that covers the comprehensive syntax. In this summary, I picked the commonly used ones that could serve as a good starting point.

## 1. Font Formats

``` 
# heading 1
## heading 2
### heading 3
```
{{< alert icon="wand-magic-sparkles" cardColor="#10b981" iconColor="#1d3557">}}
<u>***Rendered Outputs:***</u><br>
# heading 1
## heading 2
### heading 3
{{< /alert >}}
```
**bold**
*italic*
***bold and italic***
<u>underline</u>
~~strikethrough~~
`mark`
```
{{< alert icon="wand-magic-sparkles" cardColor="#10b981" iconColor="#1d3557">}}
<u>***Rendered Outputs:***</u><br>
**bold**<br>
*italic*<br>
***bold and italic***<br>
<u>underline</u><br>
~~strikethrough~~<br>
`mark`
{{< /alert >}}

## 2. Lists

**Unordered List**
```
- a
- b
- c
```
{{< alert icon="wand-magic-sparkles" cardColor="#10b981" iconColor="#1d3557">}}
<u>***Rendered Outputs:***</u><br>
- a
- b
- c
{{< /alert >}}

**Nested Unordered List**

```
- 1
- 2
 	- 2.2
 	- 2.3
- 3
```
{{< alert icon="wand-magic-sparkles" cardColor="#10b981" iconColor="#1d3557">}}
<u>***Rendered Outputs:***</u><br>
- 1
- 2
  - 2.2
  - 2.3
- 3
{{< /alert >}}


**Ordered List**

```
1. write
2. any
3. content
```
{{< alert icon="wand-magic-sparkles" cardColor="#10b981" iconColor="#1d3557">}}
<u>***Rendered Outputs:***</u><br>
1. write
2. any
3. content
{{< /alert >}}

## 3. Insert New Content <br>
```
[website](https://anhui-gui.com/)

Latex equations: $\alpha + \beta$

blockquote:
>As you wish.

nested blockquotes:
> As
>> you
>>> wish.
```
{{< alert icon="wand-magic-sparkles" cardColor="#10b981" iconColor="#1d3557">}}
<u>***Rendered Outputs:***</u><br>
[website](https://anhui-gui.com/)

{{< katex >}}\\(\alpha + \beta\\)

>As you wish.

> As
>> you
>>> wish.
{{< /alert >}}

**Image:**<br>
```
![Alt Text](path/to/image.format)

```

**Code:**
```

```[indicate the language, python in this case]

print("hello world")

```[leave blank]

```
<u>***Rendered Outputs:***</u><br>
```python
print("hello world")
```


[^1]: <https://en.wikipedia.org/wiki/Markdown>
