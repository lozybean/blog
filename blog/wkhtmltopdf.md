Title: 使用wkhtmltopdf
Date: 2016-07-06
Category: pages
Tags: htmltopdf, report

## 将HTML转为PDF

PDF格式的文档相比HTML而言, 具有不需要浏览器(更加不需要考虑浏览器兼容性等), 随处可阅的优点, 在发送报告的时候常常会使用PDF格式的文档, 但是PDF文档的生成却没有HTML那么自由灵活, 所以将已有的HTML转为PDF格式是一个重要的步骤;
[wkhtmltopdf](http://wkhtmltopdf.org/)是一个比较好用的转换工具, 该工具基于QT的webkit实现


## 添加页眉页脚

该工具支持在左中右三个位置添加页眉页脚, 并且加入一些可选变量, 如页码, 日期/时间等等;

示例:

```bash
wkhtmltopdf --footer-right '[page]/[sitepages]' 
--header-center 'xxxxx' --header-font-size 8 
--header-right 'SampleID:xxx'  in.html out.pdf
```

更加详细的用法参见[wkhtmltopdf使用帮助](http://wkhtmltopdf.org/usage/wkhtmltopdf.txt)


## 换页符

在HTML转到PDF格式的过程中, 如果需要在特定的位置换页, 则需要借助CSS样式:`page-break-before:always`;
通过在HTML的需要换页的位置添加如下空节点即可实现换页:

```html
<div style="page-break-before:always"></div>
```

## 跨页表格

当出现跨页表格时, 表头会在新的一页再次展现, 但是如果不加处理很容易出现问题, 如某一行因为行末空白超出一页, 新页表头和行内容重叠等等;
为了解决该问题, 同样需要借助CSS样式中的`page-break-inside`属性, 该属性会控制元素是否能够跨页显示, 用于表格则可以避免大多数表格转换时的问题;
以下是使用示例:

```css
tr {
    page-break-inside: avoid;	
}
```

上述例子将阻止所有的行跨页, 如果只需指定特定行只需修改CSS选择器即可;