title: css 强制换行及强制不换行
date: 2016-06-08 15:58:15
categories: 自用笔记
tags:
- word-break
- word-wrap
- white-space
- ellipsis
---

默认情况下，元素都有默认的 CSS 属性 white-space:normal（自动换行；ps: 不换行是 nowrap），当内容文字超过所定宽度时会自动换行；但如果其内的文字是堆没有空格的字符（正常情况不会出现这样的字符，但有些BT的测试人员就喜欢这样！），就会溢出容器或者把容器撑大，只有遇到空格时才换行。
<!-- more -->
## 常见解决方案
主要分两种情况，一是允许切断单词（暴力）换行`word-break`，二是以单词（单词超过一行时自动换行）为单位换行`word-wrap`。以IE，Chrome，FF为测试浏览器。
```css
{
  word-break: break-all; /* 支持IE，Chrome，低版FF不支持 */
  word-wrap: break-word; /* 支持IE，Chrome，FF */
}
```

1. `word-break:break-all` 例如：div宽200px，它的内容就会到200px自动换行，如果该行末端有个英文单词很长（congratulation等），它会把单词截断，变成该行末端为congra(congratulation的前端部分)，下一行为tulation（conguatulation的后端部分）了。
  `word-break:break-all` 支持版本：IE5以上 该行为与亚洲语言的 normal 相同。也允许非亚洲语言文本行的任意字内断开。该值适合包含一些非亚洲文本的亚洲文本。
  > __语法：__ word-break : normal | break-all | keep-all
    __参数：__ normal : 依照亚洲语言和非亚洲语言的文本规则，允许在字内换行
    　　　break-all : 该行为与亚洲语言的normal相同。也允许非亚洲语言文本行的任意字内断开。该值适合包含一些非亚洲文本的亚洲文本
    　　　keep-all : 与所有非亚洲语言的normal相同。对于中文，韩文，日文，不允许字断开。适合包含少量亚洲文本的非亚洲文本

2. `word-wrap:break-word` 例子与上面一样，但区别就是它会把congratulation整个单词看成一个整体，如果该行末端宽度不够显示整个单词，它会自动把整个单词放到下一行，而不会把单词截断掉的。
  `word-wrap:break-word` 支持版本：IE5.5以上 内容将在边界内换行。如果需要，词内换行( word-break )也将发生。
  > __语法：__ word-wrap : normal | break-word
    __参数：__ normal : 允许内容顶开指定的容器边界
    　　　 break-word : 内容将在边界内换行。如果需要，词内换行用（word-break）也能实现。

## 总结
目前 firefox 较新版的浏览器也支持`word-break` 属性了，所以用一个属性就能搞定所有常用浏览器了，如下：
```css
{
  word-break: break-all; /* 强制换行任意字符，支持IE，Chrome，FF */
}
```
__另：__ 强制不换行，结束显示省略号`...`，代码如下：
```css
{
  white-space: nowrap; /* 强制不换行 */
  text-overflow:ellipsis; /* 溢出部分用省略号代替，IE低版本无省略号 */
  overflow:hidden; /* 必须设置，否则无省略号 */
}
```