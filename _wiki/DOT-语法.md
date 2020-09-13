---
layout: article
title: DOT 语法
tags: []
key: 5c5ed54b-7daa-4d16-b636-b33f66133557
---

`DOT`是 [graphviz](https://graphviz.org/) 软件的一种专门用来描述图形的语言, 文件后缀名是 `.gv` `.dot`.

<!-- more -->

## 调试环境

学习 `DOT` 语法可以在 markdown 文件中使用 gravizo 进行编辑预览.

 [gravizo](http://www.gravizo.com/) 是一个工具, 它可以将图形描述语言(`DOT`,`PlantUml`,`UMLGraph`等)描述的图形生成一个图片链接, 以方便在 web应用, markdown 文档等直接引用.

比如如下 `DOT` 描述的图形:

```markdown
 digraph G {
    aize ="4,4";
    main [shape=box];
    main -> parse [weight=8];
    parse -> execute;
    main -> init [style=dotted];
    main -> cleanup;
    execute -> { make_string; printf}
    init -> make_string;
    edge [color=red];
    main -> printf [style=bold,label="100 times"];
    make_string [label="make a string"];
    node [shape=box,style=filled,color=".7 .3 1.0"];
    execute -> compare;
  }
```

可以直接在 markdown 中通过链接 `https://g.gravizo.com/svg` 加上参数引入图片进行展示:

```markdown
![Alt text](https://g.gravizo.com/svg?
  digraph G {
    aize ="4,4";
    main [shape=box];
    main -> parse [weight=8];
    parse -> execute;
    main -> init [style=dotted];
    main -> cleanup;
    execute -> { make_string; printf}
    init -> make_string;
    edge [color=red];
    main -> printf [style=bold,label="100 times"];
    make_string [label="make a string"];
    node [shape=box,style=filled,color=".7 .3 1.0"];
    execute -> compare;
  }
)
```

![Alt text](https://g.gravizo.com/svg?
  digraph G {
    aize ="4,4";
    main [shape=box];
    main -> parse [weight=8];
    parse -> execute;
    main -> init [style=dotted];
    main -> cleanup;
    execute -> { make_string; printf}
    init -> make_string;
    edge [color=red];
    main -> printf [style=bold,label="100 times"];
    make_string [label="make a string"];
    node [shape=box,style=filled,color=".7 .3 1.0"];
    execute -> compare;
  }
)



## 语法规则

一个图形核心概念有图(graph), 子图(subgraph), 节点(node), 边(edge), 属性(attr)这几个概念.

```bash
graph       :   [ strict ] (graph | digraph) [ ID ] '{' stmt_list '}'
stmt_list   :   [ stmt [ ';' ] stmt_list ]
stmt        :   node_stmt
            |   edge_stmt
            |   attr_stmt
            |   ID '=' ID
            |   subgraph
attr_stmt   :   (graph | node | edge) attr_list
attr_list   :   '[' [ a_list ] ']' [ attr_list ]
a_list      :   ID '=' ID [ (';' | ',') ] [ a_list ]
edge_stmt   :   (node_id | subgraph) edgeRHS [ attr_list ]
edgeRHS     :   edgeop (node_id | subgraph) [ edgeRHS ]
node_stmt   :   node_id [ attr_list ]
node_id     :   ID [ port ]
port        :   ':' ID [ ':' compass_pt ]
            |   ':' compass_pt
subgraph    :   [ subgraph [ ID ] ] '{' stmt_list '}'
compass_pt  :   (n | ne | e | se | s | sw | w | nw | c | _)
```

### `ID`

代表标识符, 支持三种

* 字母`[a-zA-Z\200-\377]`, 数字, 下划线组成的非数字开头的字符串
* 纯数字, 包括小数点和负数
* 双引号括起来的字符串
* HTML 格式的字符串 `<>`

### `graph`

代表图形, 有两种类型 `graph`(无向图), `digraph`(有向图), 加前缀 `strict` 会禁止创建多个相同的边

### `subgraph`

代表子图形, 有三种作用

* 声明 `node`, `edge`是一个分组, 比如 `A -> {B C}`等价于两条语句 `A -> B` `A -> C`

* 将子图形的元素设置为相同的属性, 比如 A, B 和 C 有相同的 `rank`: `subgraph {  rank = same; A; B; C; } `

* 如果是 `cluster` 开头, 表明是一个特殊的子图. 如果布局引擎支持 `cluster`，则会将属于 `cluster`的节点绘制在一起，并将`subgraph`的整个图形包含在矩形边框内。

### `node`

代表节点, 完整的声明是 `node_id [ attr_list ]`.

#### node 类型

可以通过给 `node` 设置 `shape`属性修改节点的类型: `node [shape=record];`

##### 默认支持的

`shape` 可以设置为: [Polygon-based Nodes](https://graphviz.gitlab.io/_pages/doc/info/shapes.html#polygon)

##### record

`shape` 可以设置为 `record`(方角) , `Mrecord`(圆角).

此时节点展示的就像数据库的一条记录一样, 展示的字段是节点的 `label` 属性值确定, 它的格式为:

```bash
# 用 | 分割的多个字段
field ( '|' field )*
# {} 声明下级记录行, 即包裹的多个字段会和当前的 fieldId 占用相同的空间
field  =  fieldId or '{' rlabel '}'
# <> 中的是 port, 可以在 edge 声明中引用, string 中的 {} <> | 需要 \ 转义
fieldId  =  [ '<' string '>'] [ string ]
```

记录行中的字段是平行还是垂直由 `graph` 的 `rankdir`属性决定(默认-TB, 垂直-BT, 水平向右-LR, 水平向左RL).

#### edge 中使用

在边中使用的格式: `node_id:port:compass_pt`, 其中 `port`, `compass_pt` 中的是可选的, `port` 使用的之前说的 `shape=record`时 `label`中的值中 `<>`声明的, 而`compass_pt` 值是固定的, 可选值:

| 方向 | 说明       |
| ---- | ---------- |
| n    | north      |
| ne   | north east |
| e    | east       |
| se   | south east |
| s    | south      |
| sw   | south west |
| w    | west       |
| nw   | north west |
| c    | center     |
| _    | 任意       |

如下示例:

```markdown
  digraph G {
    node [shape = record,height=.1];
    begin [label = "<head_1> head|<foot> foot", style=filled color=lightblue, height=.5]
    end [label = "end"]
    begin:s -> end:n [label = "s -> n"]
    begin:head_1:s -> end:n [label = "s -> n"]
    begin:s -> end:ne [label = "s -> ne"]
  }
```

![Alt text](https://g.gravizo.com/svg?
  digraph G {
    node [shape = record,height=.1];
    begin [label = "<head_1> head|<foot> foot", style=filled color=lightblue, height=.5]
    end [label = "end"]
    begin:s -> end:n [label = "s -> n"]
    begin:head_1:s -> end:n [label = "s -> n"]
    begin:s -> end:ne [label = "s -> ne"]
  }
)

### `edge`

边的声明为: `(node_id | subgraph) ( '->' | '--' ) (node_id | subgraph) [ attr_list ]`

#### 边属性

箭头方向: `dir`可选值: `forword`, `back`, `both`, `none`

头部形状: `arrowhead`[可选值](https://graphviz.org/doc/info/arrows.html)

尾部形状: `arrowtail` [可选值](https://graphviz.org/doc/info/arrows.html)

### `attr`

属性是用来描述 `graph`, `node`, `edge`等的, 声明的格式是: `(graph | node | edge) '[' [ a_list ] ']' [ attr_list ]`

由`[]`包裹的多个项组成, 每个项以 `key = value` 的形式存在，项可选择 '**,**' 和 '**;**’ 结尾.

各个元素(E(edge), N(node, G(root graph,S(subgraphs), C(cluster subgraphs))支持的[属性](http://www.graphviz.org/doc/info/attrs.html):

- `arrowhead(E)`：箭头形状，取值：[arrowType](http://www.graphviz.org/doc/info/attrs.html#k:arrowType)，类似`arrowtail`；
- `arrowsize(E)`：箭头大小，取值（0~1.0）；
- `bgcolor(GC)`：背景颜色，可以是纯色[color](http://www.graphviz.org/doc/info/attrs.html#k:color)或者渐变色[colorList](http://www.graphviz.org/doc/info/attrs.html#k:colorList)；
- `clusterrank(G)`：设置为`local`（默认值）后，cluster开头的subgraph会增加边框；
- `color(ENC)`：颜色，一般是边框，`fillcolor(NEG)`是内部颜色；
- `colorscheme(ENCG)`：指定颜色的方案，在指定颜色时可以使用`color=1`的方式，会方便很多；不指定也会有一套默认的颜色方案；
- `compound(G)`：设置`true`后，可在cluster之间连线；
- `concentrate(G)`：设置`true`后，会将重复的连线进行合并；
- `constraint(E)`：设置为`false`后，这条边不会用来考虑节点的排序，节点的排序只依赖其他的边；
- `decorate(E)`：
- `dim(G)`：
- `dimen(G)`：
- `dir(E)`：指定边的方向，可以的值有forward、back、both、none;
- `diredgeconstraints(G)`：
- `distortion(N)`：影响多边形的形状，上面大or下面大；
- `dpi(G)`：每英寸的像素数量，调节生成图片的清晰度；
- `edgeURL、edgehref(E)`：链接；
- `edgetarget(E)`：
- `edgetooltip(E)`：
- `epsilon(E)`：迭代算法用到的一个参数，中止条件，一般应该用不到；
- `esep(G)`：
- `fixedsize(N)`：如果设置为false，节点的大小会设置为刚好放下里面的内容，否则会严格按照给定的width和height来绘制；
- `fontcolor(ENGC)`：文字颜色；
- `fontname(ENGC)`：字体名称；
- `fontnames(G)`：字体名称列表，挑最先能匹配到的字体来用；
- `fontpath(G)`：字体文件路径；
- `fontsize(ENGC)`：文字大小；
- ...

## 参考链接

* [Graphviz绘图（三）：节点图形]([https://www.bitlogs.tech/2019/11/graphviz%E7%BB%98%E5%9B%BE%E4%B8%89%E8%8A%82%E7%82%B9%E5%9B%BE%E5%BD%A2/](https://www.bitlogs.tech/2019/11/graphviz绘图三节点图形/))
* [Dot 语法](http://wsztrush.com/2018/10/16/dot/)
* [Graphviz 画图的一些总结](https://www.cnblogs.com/shuqin/p/11897207.html)



