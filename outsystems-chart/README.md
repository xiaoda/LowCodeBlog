OutSystems 在开发常规页面特别是管理后台类型网站时具有很高的开发效率，但在富前端内容较多的领域，例如统计图表、自定义图表时，OutSystems 提供的低代码工具并不擅长，反而是传统方式的前端开发更加有效。OutSystems 目前支持的图表主要有 Area Chart, Bar Chart, Column Chart, Donut Chart, Line Chart, Pie Chart 这六种，其它类型的图表需求就需要传统前端开发的介入了。

OutSystems 嵌入富前端内容的方式有两种：iframe 嵌入以及 JS 集成。

# A. iframe 嵌入
该方式使用 React / Vue 等前端框架开发单页应用部署到服务器，然后用 HTML 的 &lt;iframe&gt; 标签将相关页面嵌到 OutSystems 中。Traditional Web App 有专门的 iframe 组件可供使用，而在 Reactive Web App 里，可选择 HTML Element 组件，将 Tag 属性设置为 iframe 即可。

使用 iframe 嵌入相当于将富前端内容与 OutSystems 进行了分离，我们需要为富前端内容部署服务器，可能还需要开发额外接口给前端页面调用，这些额外成本是该方法必须考虑的。所以另一种方法，JS 集成，可能更适用于这类场景。

# B. JS 集成
JS 集成相当于直接将图表插件及定制 JS 脚本插入到 OutSystems 页面，在实现需求的同时不需要增加额外的服务器资源及后端接口。该方式有两个问题需要解决：如何插入 JS 脚本或运行 JS 代码，以及如何将所需的 Entity 数据交到 JS 代码进行处理。

## 问题一：插入 JS 脚本
OutSystems 官方列举了插入 JS 脚本或代码的 [几种方法](https://success.outsystems.com/Documentation/11/Extensibility_and_Integration/JavaScript/Extend_your_Traditional_Web_App_using_JavaScript)。就使用外部 JS 库来说，我们可以与传统方式一样，通过 [动态添加 &lt;script&gt; 标签](https://stackoverflow.com/questions/13121948/dynamically-add-script-tag-with-src-that-may-include-document-write) 的方式进行插入引用。在实际使用该方法插入图表库 AntV 及 Echarts 的过程中发现，外部 JS 脚本引入后无法正确运行，经过一系列分析后推测出是 OutSystems JS 脚本执行环境的问题，由于 OutSystems 对 JS 脚本执行环境的改造，导致外部 JS 脚本开头的环境判断出现问题因而不能正确执行。解决方法是依次排查找到引发判断出错的具体语句并改写，以 AntV 为例：

``` javascript
!function(t,e){"object"==typeof exports&amp;&amp;"object"==typeof module?module.exports=e():"function"==typeof define&amp;&amp;define.amd?define([],e):"object"==typeof exports?exports.G2=e():t.G2=e()}(window,(function(){return function(t){var e={};function n(r){if(e[r])return e[r].exports;var i=e[r]={i:r,l:!1,exports:{}};return t[r].call(i.exports,i,i.exports,n),i.l=!0,i.exports}return n.m=t,n.c=e,n.d=function(t,e,r){n.o(t,e)||Object.defineProperty(t,e,{enumerable:!0,get:r})},n.r=function(t){"undefined"!=typeof Symbol&amp;&amp;Symbol.toStringTag&amp;&amp;Object.defineProperty(t,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(t,"__esModule",{value:!0})},n.t=function(t,e){if(1&amp;e&amp;&amp;(t=n(t)),8&amp;e)return t;if(4&amp;e&amp;&amp;"object"==typeof t&amp;&amp;t&amp;&amp;t.__esModule)return t;var r=Object.create(null);if(n.r(r),Object.defineProperty(r,"default",{enumerable:!0,value:t}),2&amp;e&amp;&amp;"string"!=typeof t)for(var i in t)n.d(r,i,function(e){return t[e]}.bind(null,i));return r},n.n=function(t){var e=t&amp;&amp;t.__esModule?function(){return t.default}:function(){return t};return n.d(e,"a",e),e},n.o=function(t,e){return Object.prototype.hasOwnProperty.call(t,e)},n.p="",n(n.s=245)}([function(t,e,n){"use strict";
```

其中，

``` javascript
"function"==typeof define
```

在 OutSystems JS 脚本执行环境与正常环境返回出不同的值，导致条件判断错误。改造方法不难，将该条件改为：

``` javascript
"function"!=typeof define
```

即可。由于 JS 库文件需要改造，因此不能直接引入外部 JS 库文件了。将微调后的 JS 库文件保存到 Interface \/ Scripts \/ xxx_js，需要的页面在 Required Scripts 属性中选择该 JS 文件即可加载使用。

## 问题二：Entity 数据传递到 JS 代码
Entity 数据在 OutSystems 内部获取和使用非常方便，但 JS 代码必须在 Client action 的 JavaScript 节点内运行，那么如何将图表所需的 Entity 数据通过 Aggregate 获取后传递到 JavaScript 节点便是下一个需要解决的问题。这个问题有点类似传统 Web 开发早期、前后端还没有实现分离的时候，PHP 向 JS 传递数据的场景。OutSystems 将数据写入 HTML 模板传递给 JS 是一种方法，但效率很低，并且给系统增加了额外的冗余；最好是能找到一种方法在不借助 HTML 模板的情况下，直接将后端数据传递给 JS 进行处理。

在 Client action 中，JavaScript 节点不支持 Aggregate 等复杂类型的数据作为入参，但 Client action 另有一个 JSON Serialize 节点，方便将 Aggregate 等复杂类型数据转换为字符串类型，这样就可以作为入参传递给 JavaScript 节点并在 JS 代码中进行使用。
