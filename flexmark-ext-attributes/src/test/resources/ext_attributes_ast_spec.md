---
title: Attributes Extension
author:
version:
date: '2017-12-20'
license: '[CC-BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/)'
...

---

## Attributes

Converts attributes `{...}` syntax into attributes AST nodes and adds an attribute provider to
set attributes for preceding node based on attribute assignment rules.

The attributes is a space separated list of attribute syntax of one of the following:

* `attr=value`
* `attr='value'`
* `attr="value"`
* `#anchor` : equivalent to `id="anchor"`
* `.class-name` : equivalent to `class="class-name"`

<!-- 
* `:attr=value` : equivalent to `style="attr: value"`
 -->

**NOTE**: Handling of multiple value assignment for attributes depends on its name:

* `class` values are accumulated as a space (` `) separated list.
* `style` values are accumulated as a semicolon (`;`) separated list.
* all others override any previous values of the same name.

The goal for this extension is to give maximum flexibility of assigning attributes to any
element in the markdown hierarchy in an intuitive manner so the target of the assignment can be
determined at a glance.

The attributes are used with a postfix notation, where they define the attributes for preceding
element in the document.

The following terms are used in the specification of rules for determining the attribute owner:

previous sibling
:   a markdown element preceding the element within its parent element's child list

no previous sibling
:   a markdown element which is first within its parent element's child list

next sibling
:   a markdown element following the element within its parent element's child list

no next sibling
:   a markdown element which is last within its parent element's child list

paragraph item container
:   a markdown element which is an item in a parent list element. In which case the first child
    paragraph is considered the item's text container and not a regular paragraph. eg.
    `ListItem`, `DefinitionItem`, `Footnote`

text element
:   markdown element representing a contiguous span of undecorated plain text uninterrupted by
    any other markdown element.

anchor target node
:   any node which can contain an `id` or `name` attribute and be a target of an anchor
    reference in a link and naturally will compute its own `id` absent one being explicitly
    assigned. eg. `Heading`

attached attributes
attributes are attached
:   attributes element without intervening white space between it and its previous sibling is
    said to be attached to its previous sibling.

unattached attributes
attributes are not attached
attributes are unattached
:   attributes element with intervening white space between it and its previous sibling is said
    to be unattached.

There are two modes of attribute assignment determination for text nodes:

## Text Node Previous Sibling

Assignment of attributes to text elements is determined by the `boolean` extension option
`ASSIGN_TEXT_ATTRIBUTES`, by default `true`.

For the rest of the specification the `options` following the example will contain:
`no-text-attributes` if `ASSIGN_TEXT_ATTRIBUTES` is `false`, and `true` otherwise.

* if text assignment is false
  * Cond 1.1 attributes are assigned to the parent of attributes element.
* else
  * if attributes are attached to the previous sibling
    * Cond 1.2 attributes are assigned to the text element
  * else
    * Cond 1.3 attributes are assigned to the text element

attributes assigned to paragraph

```````````````````````````````` example(Text Node Previous Sibling: 1) options(no-text-attributes)
Cond 1.1 text node{.red}

Cond 1.1 text node {.red}
.
<p class="red">Cond 1.1 text node</p>
<p class="red">Cond 1.1 text node</p>
.
Document[0, 51]
  Paragraph[0, 25] isTrailingBlankLine
    Text[0, 18] chars:[0, 18, "Cond  …  node"]
    AttributesNode[18, 24] textOpen:[18, 19, "{"] text:[19, 23, ".red"] textClose:[23, 24, "}"]
      AttributeNode[19, 23] name:[19, 20, "."] value:[20, 23, "red"] isImplicit isClass
  Paragraph[26, 51]
    Text[26, 44] chars:[26, 44, "Cond  …  node"]
    AttributesNode[45, 51] textOpen:[45, 46, "{"] text:[46, 50, ".red"] textClose:[50, 51, "}"]
      AttributeNode[46, 50] name:[46, 47, "."] value:[47, 50, "red"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example Text Node Previous Sibling: 2
Cond 1.2 text node{.red}

Cond 1.3 text node {.red}
.
<p><span class="red">Cond 1.2 text node</span></p>
<p class="red">Cond 1.3 text node</p>
.
Document[0, 52]
  Paragraph[0, 25] isTrailingBlankLine
    TextBase[0, 18] chars:[0, 18, "Cond  …  node"]
      Text[0, 18] chars:[0, 18, "Cond  …  node"]
    AttributesNode[18, 24] textOpen:[18, 19, "{"] text:[19, 23, ".red"] textClose:[23, 24, "}"]
      AttributeNode[19, 23] name:[19, 20, "."] value:[20, 23, "red"] isImplicit isClass
  Paragraph[26, 52]
    Text[26, 44] chars:[26, 44, "Cond  …  node"]
    AttributesNode[45, 51] textOpen:[45, 46, "{"] text:[46, 50, ".red"] textClose:[50, 51, "}"]
      AttributeNode[46, 50] name:[46, 47, "."] value:[47, 50, "red"] isImplicit isClass
````````````````````````````````


can delimit with comments

```````````````````````````````` example Text Node Previous Sibling: 3
Cond 1.2 text <!---->node{.red}

Cond 1.3 text <!---->node {.red}
.
<p>Cond 1.2 text <!----><span class="red">node</span></p>
<p class="red">Cond 1.3 text <!---->node</p>
.
Document[0, 66]
  Paragraph[0, 32] isTrailingBlankLine
    Text[0, 14] chars:[0, 14, "Cond  … text "]
    HtmlInlineComment[14, 21] chars:[14, 21, "<!---->"]
    TextBase[21, 25] chars:[21, 25, "node"]
      Text[21, 25] chars:[21, 25, "node"]
    AttributesNode[25, 31] textOpen:[25, 26, "{"] text:[26, 30, ".red"] textClose:[30, 31, "}"]
      AttributeNode[26, 30] name:[26, 27, "."] value:[27, 30, "red"] isImplicit isClass
  Paragraph[33, 66]
    Text[33, 47] chars:[33, 47, "Cond  … text "]
    HtmlInlineComment[47, 54] chars:[47, 54, "<!---->"]
    Text[54, 58] chars:[54, 58, "node"]
    AttributesNode[59, 65] textOpen:[59, 60, "{"] text:[60, 64, ".red"] textClose:[64, 65, "}"]
      AttributeNode[60, 64] name:[60, 61, "."] value:[61, 64, "red"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example(Text Node Previous Sibling: 4) options(no-text-attributes)
Cond 1.2 **text node{.red}**

Cond 1.3 **text node {.red}**
.
<p>Cond 1.2 <strong class="red">text node</strong></p>
<p>Cond 1.3 <strong class="red">text node</strong></p>
.
Document[0, 59]
  Paragraph[0, 29] isTrailingBlankLine
    Text[0, 9] chars:[0, 9, "Cond 1.2 "]
    StrongEmphasis[9, 28] textOpen:[9, 11, "**"] text:[11, 26, "text node{.red}"] textClose:[26, 28, "**"]
      Text[11, 20] chars:[11, 20, "text node"]
      AttributesNode[20, 26] textOpen:[20, 21, "{"] text:[21, 25, ".red"] textClose:[25, 26, "}"]
        AttributeNode[21, 25] name:[21, 22, "."] value:[22, 25, "red"] isImplicit isClass
  Paragraph[30, 59]
    Text[30, 39] chars:[30, 39, "Cond 1.3 "]
    StrongEmphasis[39, 59] textOpen:[39, 41, "**"] text:[41, 57, "text node {.red}"] textClose:[57, 59, "**"]
      Text[41, 50] chars:[41, 50, "text node"]
      AttributesNode[51, 57] textOpen:[51, 52, "{"] text:[52, 56, ".red"] textClose:[56, 57, "}"]
        AttributeNode[52, 56] name:[52, 53, "."] value:[53, 56, "red"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example Text Node Previous Sibling: 5
Cond 1.2 **text node{.red}**

Cond 1.3 **text node {.red}**
.
<p>Cond 1.2 <strong><span class="red">text node</span></strong></p>
<p>Cond 1.3 <strong class="red">text node</strong></p>
.
Document[0, 60]
  Paragraph[0, 29] isTrailingBlankLine
    Text[0, 9] chars:[0, 9, "Cond 1.2 "]
    StrongEmphasis[9, 28] textOpen:[9, 11, "**"] text:[11, 26, "text node{.red}"] textClose:[26, 28, "**"]
      TextBase[11, 20] chars:[11, 20, "text node"]
        Text[11, 20] chars:[11, 20, "text node"]
      AttributesNode[20, 26] textOpen:[20, 21, "{"] text:[21, 25, ".red"] textClose:[25, 26, "}"]
        AttributeNode[21, 25] name:[21, 22, "."] value:[22, 25, "red"] isImplicit isClass
  Paragraph[30, 60]
    Text[30, 39] chars:[30, 39, "Cond 1.3 "]
    StrongEmphasis[39, 59] textOpen:[39, 41, "**"] text:[41, 57, "text node {.red}"] textClose:[57, 59, "**"]
      Text[41, 50] chars:[41, 50, "text node"]
      AttributesNode[51, 57] textOpen:[51, 52, "{"] text:[52, 56, ".red"] textClose:[56, 57, "}"]
        AttributeNode[52, 56] name:[52, 53, "."] value:[53, 56, "red"] isImplicit isClass
````````````````````````````````


can delimit with comments

```````````````````````````````` example Text Node Previous Sibling: 6
Cond 1.2 **text <!---->node{.red}**

Cond 1.3 **text <!---->node {.red}**
.
<p>Cond 1.2 <strong>text <!----><span class="red">node</span></strong></p>
<p>Cond 1.3 <strong class="red">text <!---->node</strong></p>
.
Document[0, 74]
  Paragraph[0, 36] isTrailingBlankLine
    Text[0, 9] chars:[0, 9, "Cond 1.2 "]
    StrongEmphasis[9, 35] textOpen:[9, 11, "**"] text:[11, 33, "text <!---->node{.red}"] textClose:[33, 35, "**"]
      Text[11, 16] chars:[11, 16, "text "]
      HtmlInlineComment[16, 23] chars:[16, 23, "<!---->"]
      TextBase[23, 27] chars:[23, 27, "node"]
        Text[23, 27] chars:[23, 27, "node"]
      AttributesNode[27, 33] textOpen:[27, 28, "{"] text:[28, 32, ".red"] textClose:[32, 33, "}"]
        AttributeNode[28, 32] name:[28, 29, "."] value:[29, 32, "red"] isImplicit isClass
  Paragraph[37, 74]
    Text[37, 46] chars:[37, 46, "Cond 1.3 "]
    StrongEmphasis[46, 73] textOpen:[46, 48, "**"] text:[48, 71, "text <!---->node {.red}"] textClose:[71, 73, "**"]
      Text[48, 53] chars:[48, 53, "text "]
      HtmlInlineComment[53, 60] chars:[53, 60, "<!---->"]
      Text[60, 64] chars:[60, 64, "node"]
      AttributesNode[65, 71] textOpen:[65, 66, "{"] text:[66, 70, ".red"] textClose:[70, 71, "}"]
        AttributeNode[66, 70] name:[66, 67, "."] value:[67, 70, "red"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example(Text Node Previous Sibling: 7) options(no-text-attributes)
> Cond 1.1 text node{.red}

> Cond 1.1 text node {.red}
.
<blockquote>
  <p class="red">Cond 1.1 text node</p>
</blockquote>
<blockquote>
  <p class="red">Cond 1.1 text node</p>
</blockquote>
.
Document[0, 55]
  BlockQuote[0, 27] marker:[0, 1, ">"]
    Paragraph[2, 27]
      Text[2, 20] chars:[2, 20, "Cond  …  node"]
      AttributesNode[20, 26] textOpen:[20, 21, "{"] text:[21, 25, ".red"] textClose:[25, 26, "}"]
        AttributeNode[21, 25] name:[21, 22, "."] value:[22, 25, "red"] isImplicit isClass
  BlockQuote[28, 55] marker:[28, 29, ">"]
    Paragraph[30, 55]
      Text[30, 48] chars:[30, 48, "Cond  …  node"]
      AttributesNode[49, 55] textOpen:[49, 50, "{"] text:[50, 54, ".red"] textClose:[54, 55, "}"]
        AttributeNode[50, 54] name:[50, 51, "."] value:[51, 54, "red"] isImplicit isClass
````````````````````````````````


attributes assigned to paragraph

```````````````````````````````` example(Text Node Previous Sibling: 8) options(no-text-attributes)
> Cond 1.2 text node{.red}

> Cond 1.3 text node {.red}
.
<blockquote>
  <p class="red">Cond 1.2 text node</p>
</blockquote>
<blockquote>
  <p class="red">Cond 1.3 text node</p>
</blockquote>
.
Document[0, 55]
  BlockQuote[0, 27] marker:[0, 1, ">"]
    Paragraph[2, 27]
      Text[2, 20] chars:[2, 20, "Cond  …  node"]
      AttributesNode[20, 26] textOpen:[20, 21, "{"] text:[21, 25, ".red"] textClose:[25, 26, "}"]
        AttributeNode[21, 25] name:[21, 22, "."] value:[22, 25, "red"] isImplicit isClass
  BlockQuote[28, 55] marker:[28, 29, ">"]
    Paragraph[30, 55]
      Text[30, 48] chars:[30, 48, "Cond  …  node"]
      AttributesNode[49, 55] textOpen:[49, 50, "{"] text:[50, 54, ".red"] textClose:[54, 55, "}"]
        AttributeNode[50, 54] name:[50, 51, "."] value:[51, 54, "red"] isImplicit isClass
````````````````````````````````


attributes assigned to list item

```````````````````````````````` example(Text Node Previous Sibling: 9) options(no-text-attributes)
* Cond 1.1 text node{.red}

* Cond 1.1 text node {.red}
.
<ul>
  <li class="red">
    <p>Cond 1.1 text node</p>
  </li>
  <li class="red">
    <p>Cond 1.1 text node</p>
  </li>
</ul>
.
Document[0, 55]
  BulletList[0, 55] isLoose
    BulletListItem[0, 27] open:[0, 1, "*"] isLoose hadBlankLineAfter
      Paragraph[2, 27] isTrailingBlankLine
        Text[2, 20] chars:[2, 20, "Cond  …  node"]
        AttributesNode[20, 26] textOpen:[20, 21, "{"] text:[21, 25, ".red"] textClose:[25, 26, "}"]
          AttributeNode[21, 25] name:[21, 22, "."] value:[22, 25, "red"] isImplicit isClass
    BulletListItem[28, 55] open:[28, 29, "*"] isLoose
      Paragraph[30, 55]
        Text[30, 48] chars:[30, 48, "Cond  …  node"]
        AttributesNode[49, 55] textOpen:[49, 50, "{"] text:[50, 54, ".red"] textClose:[54, 55, "}"]
          AttributeNode[50, 54] name:[50, 51, "."] value:[51, 54, "red"] isImplicit isClass
````````````````````````````````


attributes assigned to list item

```````````````````````````````` example Text Node Previous Sibling: 10
* Cond 1.2 text node{.red}

* Cond 1.3 text node {.red}
.
<ul>
  <li>
    <p><span class="red">Cond 1.2 text node</span></p>
  </li>
  <li class="red">
    <p>Cond 1.3 text node</p>
  </li>
</ul>
.
Document[0, 56]
  BulletList[0, 56] isLoose
    BulletListItem[0, 27] open:[0, 1, "*"] isLoose hadBlankLineAfter
      Paragraph[2, 27] isTrailingBlankLine
        TextBase[2, 20] chars:[2, 20, "Cond  …  node"]
          Text[2, 20] chars:[2, 20, "Cond  …  node"]
        AttributesNode[20, 26] textOpen:[20, 21, "{"] text:[21, 25, ".red"] textClose:[25, 26, "}"]
          AttributeNode[21, 25] name:[21, 22, "."] value:[22, 25, "red"] isImplicit isClass
    BulletListItem[28, 56] open:[28, 29, "*"] isLoose
      Paragraph[30, 56]
        Text[30, 48] chars:[30, 48, "Cond  …  node"]
        AttributesNode[49, 55] textOpen:[49, 50, "{"] text:[50, 54, ".red"] textClose:[54, 55, "}"]
          AttributeNode[50, 54] name:[50, 51, "."] value:[51, 54, "red"] isImplicit isClass
````````````````````````````````


## Non Text Node Previous Sibling

* if not attached to previous sibling
  * Cond 2.1 attributes are assigned to the parent element
* else
  * if previous sibling is an attributes element
    * Cond 2.2 attributes are assigned to the same element as previous sibling attributes
  * else
    * Cond 2.3 attributes are assigned to the previous sibling

```````````````````````````````` example(Non Text Node Previous Sibling: 1) options(no-text-attributes)
Cond 2.1 Some text **bold text** {.red}

Cond 2.2 Some text **bold text**{.red}{.blue}

Cond 2.3 Some text **bold text**{.red}
.
<p class="red">Cond 2.1 Some text <strong>bold text</strong></p>
<p>Cond 2.2 Some text <strong class="red blue">bold text</strong></p>
<p>Cond 2.3 Some text <strong class="red">bold text</strong></p>
.
Document[0, 126]
  Paragraph[0, 40] isTrailingBlankLine
    Text[0, 19] chars:[0, 19, "Cond  … text "]
    StrongEmphasis[19, 32] textOpen:[19, 21, "**"] text:[21, 30, "bold text"] textClose:[30, 32, "**"]
      Text[21, 30] chars:[21, 30, "bold text"]
    AttributesNode[33, 39] textOpen:[33, 34, "{"] text:[34, 38, ".red"] textClose:[38, 39, "}"]
      AttributeNode[34, 38] name:[34, 35, "."] value:[35, 38, "red"] isImplicit isClass
  Paragraph[41, 87] isTrailingBlankLine
    Text[41, 60] chars:[41, 60, "Cond  … text "]
    StrongEmphasis[60, 73] textOpen:[60, 62, "**"] text:[62, 71, "bold text"] textClose:[71, 73, "**"]
      Text[62, 71] chars:[62, 71, "bold text"]
    AttributesNode[73, 79] textOpen:[73, 74, "{"] text:[74, 78, ".red"] textClose:[78, 79, "}"]
      AttributeNode[74, 78] name:[74, 75, "."] value:[75, 78, "red"] isImplicit isClass
    AttributesNode[79, 86] textOpen:[79, 80, "{"] text:[80, 85, ".blue"] textClose:[85, 86, "}"]
      AttributeNode[80, 85] name:[80, 81, "."] value:[81, 85, "blue"] isImplicit isClass
  Paragraph[88, 126]
    Text[88, 107] chars:[88, 107, "Cond  … text "]
    StrongEmphasis[107, 120] textOpen:[107, 109, "**"] text:[109, 118, "bold text"] textClose:[118, 120, "**"]
      Text[109, 118] chars:[109, 118, "bold text"]
    AttributesNode[120, 126] textOpen:[120, 121, "{"] text:[121, 125, ".red"] textClose:[125, 126, "}"]
      AttributeNode[121, 125] name:[121, 122, "."] value:[122, 125, "red"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example Non Text Node Previous Sibling: 2
Cond 2.1 Some text **bold text** {.red}

Cond 2.2 Some text **bold text**{.red}{.blue}

Cond 2.3 Some text **bold text**{.red}
.
<p class="red">Cond 2.1 Some text <strong>bold text</strong></p>
<p>Cond 2.2 Some text <strong class="red blue">bold text</strong></p>
<p>Cond 2.3 Some text <strong class="red">bold text</strong></p>
.
Document[0, 127]
  Paragraph[0, 40] isTrailingBlankLine
    Text[0, 19] chars:[0, 19, "Cond  … text "]
    StrongEmphasis[19, 32] textOpen:[19, 21, "**"] text:[21, 30, "bold text"] textClose:[30, 32, "**"]
      Text[21, 30] chars:[21, 30, "bold text"]
    AttributesNode[33, 39] textOpen:[33, 34, "{"] text:[34, 38, ".red"] textClose:[38, 39, "}"]
      AttributeNode[34, 38] name:[34, 35, "."] value:[35, 38, "red"] isImplicit isClass
  Paragraph[41, 87] isTrailingBlankLine
    Text[41, 60] chars:[41, 60, "Cond  … text "]
    StrongEmphasis[60, 73] textOpen:[60, 62, "**"] text:[62, 71, "bold text"] textClose:[71, 73, "**"]
      Text[62, 71] chars:[62, 71, "bold text"]
    AttributesNode[73, 79] textOpen:[73, 74, "{"] text:[74, 78, ".red"] textClose:[78, 79, "}"]
      AttributeNode[74, 78] name:[74, 75, "."] value:[75, 78, "red"] isImplicit isClass
    AttributesNode[79, 86] textOpen:[79, 80, "{"] text:[80, 85, ".blue"] textClose:[85, 86, "}"]
      AttributeNode[80, 85] name:[80, 81, "."] value:[81, 85, "blue"] isImplicit isClass
  Paragraph[88, 127]
    Text[88, 107] chars:[88, 107, "Cond  … text "]
    StrongEmphasis[107, 120] textOpen:[107, 109, "**"] text:[109, 118, "bold text"] textClose:[118, 120, "**"]
      Text[109, 118] chars:[109, 118, "bold text"]
    AttributesNode[120, 126] textOpen:[120, 121, "{"] text:[121, 125, ".red"] textClose:[125, 126, "}"]
      AttributeNode[121, 125] name:[121, 122, "."] value:[122, 125, "red"] isImplicit isClass
````````````````````````````````


## No Previous Sibling

* if parent is a paragraph
  * if paragraph parent is a paragraph item container
    * if paragraph has no previous sibling then
      * Cond 3.1 attributes go to the paragraph parent's parent
    * else
      * if paragraph only contains attributes then
        * Cond 3.2 attributes go to paragraph's previous sibling,
      * else
        * Cond 3.3 attributes go to the paragraph
  * else
    * if paragraph only contains attributes then
      * if paragraph has no previous sibling then
        * Cond 3.4 attributes go to paragraph's parent
      * else
        * Cond 3.5 attributes go to paragraph's previous sibling,
    * else
      * Cond 3.6 attributes go to the paragraph
* else
  * Cond 3.7 attributes go to the parent

### Cond 3.1

Cond 3.1 attributes go to the paragraph parent's parent

attributes are assigned to the list element

```````````````````````````````` example Cond 3.1: 1
* {.red} list item 1
* list item 2

* list item 1
* {.red} list item 2
.
<ul class="red">
  <li>
    <p>list item 1</p>
  </li>
  <li>
    <p>list item 2</p>
  </li>
  <li>
    <p>list item 1</p>
  </li>
  <li>
    <p>list item 2</p>
  </li>
</ul>
.
Document[0, 71]
  BulletList[0, 71] isLoose
    BulletListItem[0, 21] open:[0, 1, "*"] isLoose
      Paragraph[2, 21]
        AttributesNode[2, 8] textOpen:[2, 3, "{"] text:[3, 7, ".red"] textClose:[7, 8, "}"]
          AttributeNode[3, 7] name:[3, 4, "."] value:[4, 7, "red"] isImplicit isClass
        Text[9, 20] chars:[9, 20, "list  … tem 1"]
    BulletListItem[21, 35] open:[21, 22, "*"] isLoose hadBlankLineAfter
      Paragraph[23, 35] isTrailingBlankLine
        Text[23, 34] chars:[23, 34, "list  … tem 2"]
    BulletListItem[36, 50] open:[36, 37, "*"] isLoose
      Paragraph[38, 50]
        Text[38, 49] chars:[38, 49, "list  … tem 1"]
    BulletListItem[50, 71] open:[50, 51, "*"] isLoose
      Paragraph[52, 71]
        AttributesNode[52, 58] textOpen:[52, 53, "{"] text:[53, 57, ".red"] textClose:[57, 58, "}"]
          AttributeNode[53, 57] name:[53, 54, "."] value:[54, 57, "red"] isImplicit isClass
        Text[59, 70] chars:[59, 70, "list  … tem 2"]
````````````````````````````````


attributes are assigned to the definition list

```````````````````````````````` example Cond 3.1: 2
Definition Term
:   {.red} definition item 1
:   definition item 2

Definition Term
:   definition item 1
:   {.red} definition item 2
.
<dl class="red">
  <dt>Definition Term</dt>
  <dd>definition item 1</dd>
  <dd>definition item 2</dd>
  <dt>Definition Term</dt>
  <dd>definition item 1</dd>
  <dd>definition item 2</dd>
</dl>
.
Document[0, 135]
  DefinitionList[0, 135] isTight
    DefinitionTerm[0, 16]
      Paragraph[0, 16]
        Text[0, 15] chars:[0, 15, "Defin …  Term"]
    DefinitionItem[16, 45] open:[16, 17, ":"] isTight
      Paragraph[20, 45]
        AttributesNode[20, 26] textOpen:[20, 21, "{"] text:[21, 25, ".red"] textClose:[25, 26, "}"]
          AttributeNode[21, 25] name:[21, 22, "."] value:[22, 25, "red"] isImplicit isClass
        Text[27, 44] chars:[27, 44, "defin … tem 1"]
    DefinitionItem[45, 67] open:[45, 46, ":"] isTight hadBlankLineAfter
      Paragraph[49, 67] isTrailingBlankLine
        Text[49, 66] chars:[49, 66, "defin … tem 2"]
    DefinitionTerm[68, 84]
      Paragraph[68, 84]
        Text[68, 83] chars:[68, 83, "Defin …  Term"]
    DefinitionItem[84, 106] open:[84, 85, ":"] isTight
      Paragraph[88, 106]
        Text[88, 105] chars:[88, 105, "defin … tem 1"]
    DefinitionItem[106, 135] open:[106, 107, ":"] isTight
      Paragraph[110, 135]
        AttributesNode[110, 116] textOpen:[110, 111, "{"] text:[111, 115, ".red"] textClose:[115, 116, "}"]
          AttributeNode[111, 115] name:[111, 112, "."] value:[112, 115, "red"] isImplicit isClass
        Text[117, 134] chars:[117, 134, "defin … tem 2"]
````````````````````````````````


### Cond 3.2 

Cond 3.2 attributes go to paragraph's previous sibling,

Assigned to paragraph of list item 2

```````````````````````````````` example Cond 3.2: 1
* list item 1
* list item 2

  {.red}
.
<ul>
  <li>
    <p>list item 1</p>
  </li>
  <li>
    <p class="red">list item 2</p>
  </li>
</ul>
.
Document[0, 38]
  BulletList[0, 38] isLoose
    BulletListItem[0, 14] open:[0, 1, "*"] isLoose
      Paragraph[2, 14]
        Text[2, 13] chars:[2, 13, "list  … tem 1"]
    BulletListItem[14, 38] open:[14, 15, "*"] isLoose hadBlankLineAfter
      Paragraph[16, 28] isTrailingBlankLine
        Text[16, 27] chars:[16, 27, "list  … tem 2"]
      Paragraph[31, 38]
        AttributesNode[31, 37] textOpen:[31, 32, "{"] text:[32, 36, ".red"] textClose:[36, 37, "}"]
          AttributeNode[32, 36] name:[32, 33, "."] value:[33, 36, "red"] isImplicit isClass
````````````````````````````````


Assigned to paragraph of list item 2

```````````````````````````````` example Cond 3.2: 2
* list item 1

* list item 2

  {.red}
.
<ul>
  <li>
    <p>list item 1</p>
  </li>
  <li>
    <p class="red">list item 2</p>
  </li>
</ul>
.
Document[0, 39]
  BulletList[0, 39] isLoose
    BulletListItem[0, 14] open:[0, 1, "*"] isLoose hadBlankLineAfter
      Paragraph[2, 14] isTrailingBlankLine
        Text[2, 13] chars:[2, 13, "list  … tem 1"]
    BulletListItem[15, 39] open:[15, 16, "*"] isLoose hadBlankLineAfter
      Paragraph[17, 29] isTrailingBlankLine
        Text[17, 28] chars:[17, 28, "list  … tem 2"]
      Paragraph[32, 39]
        AttributesNode[32, 38] textOpen:[32, 33, "{"] text:[33, 37, ".red"] textClose:[37, 38, "}"]
          AttributeNode[33, 37] name:[33, 34, "."] value:[34, 37, "red"] isImplicit isClass
````````````````````````````````


Assigned to definition item 2 paragraph

```````````````````````````````` example Cond 3.2: 3
definition term
: definition item 1
: definition item 2

  {.red}

.
<dl>
  <dt>definition term</dt>
  <dd>definition item 1</dd>
  <dd>definition item 2</dd>
</dl>
.
Document[0, 67]
  DefinitionList[0, 66] isTight
    DefinitionTerm[0, 16]
      Paragraph[0, 16]
        Text[0, 15] chars:[0, 15, "defin …  term"]
    DefinitionItem[16, 36] open:[16, 17, ":"] isTight
      Paragraph[18, 36]
        Text[18, 35] chars:[18, 35, "defin … tem 1"]
    DefinitionItem[36, 66] open:[36, 37, ":"] isTight hadBlankLineAfter
      Paragraph[38, 56] isTrailingBlankLine
        Text[38, 55] chars:[38, 55, "defin … tem 2"]
      Paragraph[59, 66] isTrailingBlankLine
        AttributesNode[59, 65] textOpen:[59, 60, "{"] text:[60, 64, ".red"] textClose:[64, 65, "}"]
          AttributeNode[60, 64] name:[60, 61, "."] value:[61, 64, "red"] isImplicit isClass
````````````````````````````````


Assigned to definition item 2 paragraph

```````````````````````````````` example Cond 3.2: 4
definition term
: definition item 1

: definition item 2

  {.red}

.
<dl>
  <dt>definition term</dt>
  <dd>
    <p>definition item 1</p>
  </dd>
  <dd>
    <p class="red">definition item 2</p>
  </dd>
</dl>
.
Document[0, 68]
  DefinitionList[0, 67] isLoose
    DefinitionTerm[0, 16]
      Paragraph[0, 16]
        Text[0, 15] chars:[0, 15, "defin …  term"]
    DefinitionItem[16, 36] open:[16, 17, ":"] isLoose hadBlankLineAfter
      Paragraph[18, 36] isTrailingBlankLine
        Text[18, 35] chars:[18, 35, "defin … tem 1"]
    DefinitionItem[37, 67] open:[37, 38, ":"] isLoose hadBlankLineAfter
      Paragraph[39, 57] isTrailingBlankLine
        Text[39, 56] chars:[39, 56, "defin … tem 2"]
      Paragraph[60, 67] isTrailingBlankLine
        AttributesNode[60, 66] textOpen:[60, 61, "{"] text:[61, 65, ".red"] textClose:[65, 66, "}"]
          AttributeNode[61, 65] name:[61, 62, "."] value:[62, 65, "red"] isImplicit isClass
````````````````````````````````


### Cond 3.3

Cond 3.3 attributes go to the paragraph

Assigned to paragraph

```````````````````````````````` example Cond 3.3: 1
* list item 1
* list item 2

  {.red} Some text

.
<ul>
  <li>
    <p>list item 1</p>
  </li>
  <li>
    <p>list item 2</p>
    <p class="red">Some text</p>
  </li>
</ul>
.
Document[0, 49]
  BulletList[0, 48] isLoose
    BulletListItem[0, 14] open:[0, 1, "*"] isLoose
      Paragraph[2, 14]
        Text[2, 13] chars:[2, 13, "list  … tem 1"]
    BulletListItem[14, 48] open:[14, 15, "*"] isLoose hadBlankLineAfter
      Paragraph[16, 28] isTrailingBlankLine
        Text[16, 27] chars:[16, 27, "list  … tem 2"]
      Paragraph[31, 48] isTrailingBlankLine
        AttributesNode[31, 37] textOpen:[31, 32, "{"] text:[32, 36, ".red"] textClose:[36, 37, "}"]
          AttributeNode[32, 36] name:[32, 33, "."] value:[33, 36, "red"] isImplicit isClass
        Text[38, 47] chars:[38, 47, "Some text"]
````````````````````````````````


Assigned to definition item 2

```````````````````````````````` example Cond 3.3: 2
definition term
: definition item 1
: definition item 2

  {.red} some text

.
<dl>
  <dt>definition term</dt>
  <dd>definition item 1</dd>
  <dd>definition item 2
  <p class="red">some text</p>
  </dd>
</dl>
.
Document[0, 77]
  DefinitionList[0, 76] isTight
    DefinitionTerm[0, 16]
      Paragraph[0, 16]
        Text[0, 15] chars:[0, 15, "defin …  term"]
    DefinitionItem[16, 36] open:[16, 17, ":"] isTight
      Paragraph[18, 36]
        Text[18, 35] chars:[18, 35, "defin … tem 1"]
    DefinitionItem[36, 76] open:[36, 37, ":"] isTight hadBlankLineAfter
      Paragraph[38, 56] isTrailingBlankLine
        Text[38, 55] chars:[38, 55, "defin … tem 2"]
      Paragraph[59, 76] isTrailingBlankLine
        AttributesNode[59, 65] textOpen:[59, 60, "{"] text:[60, 64, ".red"] textClose:[64, 65, "}"]
          AttributeNode[60, 64] name:[60, 61, "."] value:[61, 64, "red"] isImplicit isClass
        Text[66, 75] chars:[66, 75, "some text"]
````````````````````````````````


### Cond 3.4

* paragraph parent is not a paragraph item container
* if paragraph only contains attributes then
  * if paragraph has no previous sibling then
    * Cond 3.4 attributes go to paragraph's parent

Assigned to block quote

```````````````````````````````` example Cond 3.4: 1
> {.red}
> 
> block quote text
.
<blockquote class="red">
  <p>block quote text</p>
</blockquote>
.
Document[0, 31]
  BlockQuote[0, 31] marker:[0, 1, ">"]
    Paragraph[2, 9] isTrailingBlankLine
      AttributesNode[2, 8] textOpen:[2, 3, "{"] text:[3, 7, ".red"] textClose:[7, 8, "}"]
        AttributeNode[3, 7] name:[3, 4, "."] value:[4, 7, "red"] isImplicit isClass
    Paragraph[14, 31]
      Text[14, 30] chars:[14, 30, "block …  text"]
````````````````````````````````


### Cond 3.5 

Cond 3.5 attributes go to paragraph's previous sibling,

Assigned to previous paragraph

```````````````````````````````` example Cond 3.5: 1
Some paragraph
 
{.red}
.
<p class="red">Some paragraph</p>
.
Document[0, 24]
  Paragraph[0, 15] isTrailingBlankLine
    Text[0, 14] chars:[0, 14, "Some  … graph"]
  Paragraph[17, 24]
    AttributesNode[17, 23] textOpen:[17, 18, "{"] text:[18, 22, ".red"] textClose:[22, 23, "}"]
      AttributeNode[18, 22] name:[18, 19, "."] value:[19, 22, "red"] isImplicit isClass
````````````````````````````````


Assigned to previous block quote

```````````````````````````````` example Cond 3.5: 2
> Some paragraph
 
{.red}
.
<blockquote class="red">
  <p>Some paragraph</p>
</blockquote>
.
Document[0, 26]
  BlockQuote[0, 17] marker:[0, 1, ">"]
    Paragraph[2, 17]
      Text[2, 16] chars:[2, 16, "Some  … graph"]
  Paragraph[19, 26]
    AttributesNode[19, 25] textOpen:[19, 20, "{"] text:[20, 24, ".red"] textClose:[24, 25, "}"]
      AttributeNode[20, 24] name:[20, 21, "."] value:[21, 24, "red"] isImplicit isClass
````````````````````````````````


Assigned to previous paragraph in the block quote

```````````````````````````````` example Cond 3.5: 3
> Some paragraph
> 
> {.red}
.
<blockquote>
  <p class="red">Some paragraph</p>
</blockquote>
.
Document[0, 29]
  BlockQuote[0, 29] marker:[0, 1, ">"]
    Paragraph[2, 17] isTrailingBlankLine
      Text[2, 16] chars:[2, 16, "Some  … graph"]
    Paragraph[22, 29]
      AttributesNode[22, 28] textOpen:[22, 23, "{"] text:[23, 27, ".red"] textClose:[27, 28, "}"]
        AttributeNode[23, 27] name:[23, 24, "."] value:[24, 27, "red"] isImplicit isClass
````````````````````````````````


Assigned to previous list

```````````````````````````````` example Cond 3.5: 4
* list item 1
* list item 2
 
{.red}
.
<ul class="red">
  <li>list item 1</li>
  <li>list item 2</li>
</ul>
.
Document[0, 37]
  BulletList[0, 28] isTight
    BulletListItem[0, 14] open:[0, 1, "*"] isTight
      Paragraph[2, 14]
        Text[2, 13] chars:[2, 13, "list  … tem 1"]
    BulletListItem[14, 28] open:[14, 15, "*"] isTight hadBlankLineAfter
      Paragraph[16, 28] isTrailingBlankLine
        Text[16, 27] chars:[16, 27, "list  … tem 2"]
  Paragraph[30, 37]
    AttributesNode[30, 36] textOpen:[30, 31, "{"] text:[31, 35, ".red"] textClose:[35, 36, "}"]
      AttributeNode[31, 35] name:[31, 32, "."] value:[32, 35, "red"] isImplicit isClass
````````````````````````````````


Assigned to previous definition list

```````````````````````````````` example Cond 3.5: 5
definition term 1
: definition item 1.1
 
definition term 2
: definition item 2.1
 
{.red}
.
<dl class="red">
  <dt>definition term 1</dt>
  <dd>definition item 1.1</dd>
  <dt>definition term 2</dt>
  <dd>definition item 2.1</dd>
</dl>
.
Document[0, 91]
  DefinitionList[0, 82] isTight
    DefinitionTerm[0, 18]
      Paragraph[0, 18]
        Text[0, 17] chars:[0, 17, "defin … erm 1"]
    DefinitionItem[18, 40] open:[18, 19, ":"] isTight hadBlankLineAfter
      Paragraph[20, 40] isTrailingBlankLine
        Text[20, 39] chars:[20, 39, "defin … m 1.1"]
    DefinitionTerm[42, 60]
      Paragraph[42, 60]
        Text[42, 59] chars:[42, 59, "defin … erm 2"]
    DefinitionItem[60, 82] open:[60, 61, ":"] isTight hadBlankLineAfter
      Paragraph[62, 82] isTrailingBlankLine
        Text[62, 81] chars:[62, 81, "defin … m 2.1"]
  Paragraph[84, 91]
    AttributesNode[84, 90] textOpen:[84, 85, "{"] text:[85, 89, ".red"] textClose:[89, 90, "}"]
      AttributeNode[85, 89] name:[85, 86, "."] value:[86, 89, "red"] isImplicit isClass
````````````````````````````````


Assigned to previous table

```````````````````````````````` example Cond 3.5: 6

| head |
|------|
| body |

{.red}
.
<table class="red">
  <thead>
    <tr><th>head</th></tr>
  </thead>
  <tbody>
    <tr><td>body</td></tr>
  </tbody>
</table>
.
Document[0, 36]
  TableBlock[1, 28]
    TableHead[1, 9]
      TableRow[1, 9] rowNumber=1
        TableCell[1, 9] header textOpen:[1, 2, "|"] text:[3, 7, "head"] textClose:[8, 9, "|"]
          Text[3, 7] chars:[3, 7, "head"]
    TableSeparator[10, 18]
      TableRow[10, 18]
        TableCell[10, 18] textOpen:[10, 11, "|"] text:[11, 17, "------"] textClose:[17, 18, "|"]
          Text[11, 17] chars:[11, 17, "------"]
    TableBody[19, 27]
      TableRow[19, 27] rowNumber=1
        TableCell[19, 27] textOpen:[19, 20, "|"] text:[21, 25, "body"] textClose:[26, 27, "|"]
          Text[21, 25] chars:[21, 25, "body"]
  Paragraph[29, 36]
    AttributesNode[29, 35] textOpen:[29, 30, "{"] text:[30, 34, ".red"] textClose:[34, 35, "}"]
      AttributeNode[30, 34] name:[30, 31, "."] value:[31, 34, "red"] isImplicit isClass
````````````````````````````````


### Cond 3.6

Cond 3.6 attributes go to the paragraph

Assigned to paragraph

```````````````````````````````` example Cond 3.6: 1
Some paragraph
 
{.red} Some Text
.
<p>Some paragraph</p>
<p class="red">Some Text</p>
.
Document[0, 34]
  Paragraph[0, 15] isTrailingBlankLine
    Text[0, 14] chars:[0, 14, "Some  … graph"]
  Paragraph[17, 34]
    AttributesNode[17, 23] textOpen:[17, 18, "{"] text:[18, 22, ".red"] textClose:[22, 23, "}"]
      AttributeNode[18, 22] name:[18, 19, "."] value:[19, 22, "red"] isImplicit isClass
    Text[24, 33] chars:[24, 33, "Some Text"]
````````````````````````````````


### Cond 3.7

* non paragraph parent
* Cond 3.7 attributes go to the parent

```````````````````````````````` example Cond 3.7: 1
Some Text **{.red}bold text**
.
<p>Some Text <strong class="red">bold text</strong></p>
.
Document[0, 30]
  Paragraph[0, 30]
    Text[0, 10] chars:[0, 10, "Some Text "]
    StrongEmphasis[10, 29] textOpen:[10, 12, "**"] text:[12, 27, "{.red}bold text"] textClose:[27, 29, "**"]
      AttributesNode[12, 18] textOpen:[12, 13, "{"] text:[13, 17, ".red"] textClose:[17, 18, "}"]
        AttributeNode[13, 17] name:[13, 14, "."] value:[14, 17, "red"] isImplicit isClass
      Text[18, 27] chars:[18, 27, "bold text"]
````````````````````````````````


## Anchor Targets

To allow for customizing id attributes of elements which have their id attribute computed from
their content, attributes have to assign the id attribute to override the computed value.

Cond 4.1 anchor target element get their id attribute from the last attributes element which is
assigned to the element and has an `id` attribute.

### Cond 4.1

```````````````````````````````` example Cond 4.1: 1
# Heading
.
<h1 id="heading">Heading</h1>
.
Document[0, 10]
  Heading[0, 9] textOpen:[0, 1, "#"] text:[2, 9, "Heading"]
    Text[2, 9] chars:[2, 9, "Heading"]
````````````````````````````````


```````````````````````````````` example Cond 4.1: 2
# Heading {#custom-id}
.
<h1 id="custom-id">Heading</h1>
.
Document[0, 23]
  Heading[0, 22] textOpen:[0, 1, "#"] text:[2, 22, "Heading {#custom-id}"]
    Text[2, 9] chars:[2, 9, "Heading"]
    AttributesNode[10, 22] textOpen:[10, 11, "{"] text:[11, 21, "#custom-id"] textClose:[21, 22, "}"]
      AttributeNode[11, 21] name:[11, 12, "#"] value:[12, 21, "custom-id"] isImplicit isId
````````````````````````````````


```````````````````````````````` example(Cond 4.1: 3) options(anchors)
# Heading {#custom-id}
.
<h1><a href="#custom-id" id="custom-id"></a>Heading</h1>
.
Document[0, 22]
  Heading[0, 22] textOpen:[0, 1, "#"] text:[2, 22, "Heading {#custom-id}"]
    AnchorLink[2, 2]
    Text[2, 9] chars:[2, 9, "Heading"]
    AttributesNode[10, 22] textOpen:[10, 11, "{"] text:[11, 21, "#custom-id"] textClose:[21, 22, "}"]
      AttributeNode[11, 21] name:[11, 12, "#"] value:[12, 21, "custom-id"] isImplicit isId
````````````````````````````````


```````````````````````````````` example Cond 4.1: 4
# Heading # {#custom-id}
.
<h1 id="custom-id">Heading #</h1>
.
Document[0, 25]
  Heading[0, 24] textOpen:[0, 1, "#"] text:[2, 24, "Heading # {#custom-id}"]
    Text[2, 11] chars:[2, 11, "Heading #"]
    AttributesNode[12, 24] textOpen:[12, 13, "{"] text:[13, 23, "#custom-id"] textClose:[23, 24, "}"]
      AttributeNode[13, 23] name:[13, 14, "#"] value:[14, 23, "custom-id"] isImplicit isId
````````````````````````````````


```````````````````````````````` example Cond 4.1: 5
Heading {#custom-id}
======================
.
<h1 id="custom-id">Heading</h1>
.
Document[0, 44]
  Heading[0, 43] text:[0, 20, "Heading {#custom-id}"] textClose:[21, 43, "======================"]
    Text[0, 7] chars:[0, 7, "Heading"]
    AttributesNode[8, 20] textOpen:[8, 9, "{"] text:[9, 19, "#custom-id"] textClose:[19, 20, "}"]
      AttributeNode[9, 19] name:[9, 10, "#"] value:[10, 19, "custom-id"] isImplicit isId
````````````````````````````````


```````````````````````````````` example Cond 4.1: 6
Heading {#123-heading}
======================
.
<h1 id="123-heading">Heading</h1>
.
Document[0, 46]
  Heading[0, 45] text:[0, 22, "Heading {#123-heading}"] textClose:[23, 45, "======================"]
    Text[0, 7] chars:[0, 7, "Heading"]
    AttributesNode[8, 22] textOpen:[8, 9, "{"] text:[9, 21, "#123-heading"] textClose:[21, 22, "}"]
      AttributeNode[9, 21] name:[9, 10, "#"] value:[10, 21, "123-heading"] isImplicit isId
````````````````````````````````


## Attributes on Reference

Cond 5.1 attributes on reference definition are applied to the references

Reference definitions do not allow any text to follow the definition. Therefore the only way to
have an attribute assigned to the definition is to place it as the only element of the following
paragraph text.

### Cond 5.1

```````````````````````````````` example Cond 5.1: 1
[test]

[test]: http://example.com 

{style="color:red"}
.
<p><a href="http://example.com" style="color:red">test</a></p>
.
Document[0, 57]
  Paragraph[0, 7] isTrailingBlankLine
    LinkRef[0, 6] referenceOpen:[0, 1, "["] reference:[1, 5, "test"] referenceClose:[5, 6, "]"]
      Text[1, 5] chars:[1, 5, "test"]
  Reference[8, 34] refOpen:[8, 9, "["] ref:[9, 13, "test"] refClose:[13, 15, "]:"] url:[16, 34, "http://example.com"]
  Paragraph[37, 57]
    AttributesNode[37, 56] textOpen:[37, 38, "{"] text:[38, 55, "style=\"color:red\""] textClose:[55, 56, "}"]
      AttributeNode[38, 55] name:[38, 43, "style"] sep:[43, 44, "="] valueOpen:[44, 45, "\""] value:[45, 54, "color:red"] valueClose:[54, 55, "\""]
````````````````````````````````


```````````````````````````````` example Cond 5.1: 2
[reference 1][test] 

[reference 2][test] 

[test]: http://example.com 

{style="color:red"}
.
<p><a href="http://example.com" style="color:red">reference 1</a></p>
<p><a href="http://example.com" style="color:red">reference 2</a></p>
.
Document[0, 93]
  Paragraph[0, 21] isTrailingBlankLine
    LinkRef[0, 19] textOpen:[0, 1, "["] text:[1, 12, "reference 1"] textClose:[12, 13, "]"] referenceOpen:[13, 14, "["] reference:[14, 18, "test"] referenceClose:[18, 19, "]"]
      Text[1, 12] chars:[1, 12, "refer … nce 1"]
  Paragraph[22, 43] isTrailingBlankLine
    LinkRef[22, 41] textOpen:[22, 23, "["] text:[23, 34, "reference 2"] textClose:[34, 35, "]"] referenceOpen:[35, 36, "["] reference:[36, 40, "test"] referenceClose:[40, 41, "]"]
      Text[23, 34] chars:[23, 34, "refer … nce 2"]
  Reference[44, 70] refOpen:[44, 45, "["] ref:[45, 49, "test"] refClose:[49, 51, "]:"] url:[52, 70, "http://example.com"]
  Paragraph[73, 93]
    AttributesNode[73, 92] textOpen:[73, 74, "{"] text:[74, 91, "style=\"color:red\""] textClose:[91, 92, "}"]
      AttributeNode[74, 91] name:[74, 79, "style"] sep:[79, 80, "="] valueOpen:[80, 81, "\""] value:[81, 90, "color:red"] valueClose:[90, 91, "\""]
````````````````````````````````


Ref image

```````````````````````````````` example Cond 5.1: 3
![test]

[test]: <http://example.com/test.png> 

{style="border-color:red"}
.
<p><img src="http://example.com/test.png" alt="test" style="border-color:red" /></p>
.
Document[0, 76]
  Paragraph[0, 8] isTrailingBlankLine
    ImageRef[0, 7] referenceOpen:[0, 2, "!["] reference:[2, 6, "test"] referenceClose:[6, 7, "]"]
      Text[2, 6] chars:[2, 6, "test"]
  Reference[9, 46] refOpen:[9, 10, "["] ref:[10, 14, "test"] refClose:[14, 16, "]:"] urlOpen:[17, 18, "<"] url:[18, 45, "http://example.com/test.png"] urlClose:[45, 46, ">"]
  Paragraph[49, 76]
    AttributesNode[49, 75] textOpen:[49, 50, "{"] text:[50, 74, "style=\"border-color:red\""] textClose:[74, 75, "}"]
      AttributeNode[50, 74] name:[50, 55, "style"] sep:[55, 56, "="] valueOpen:[56, 57, "\""] value:[57, 73, "border-color:red"] valueClose:[73, 74, "\""]
````````````````````````````````


```````````````````````````````` example Cond 5.1: 4
![reference 1][test]

![reference 2][test]

[test]: <http://example.com/test.png> 

{style="border-color:red"}
.
<p><img src="http://example.com/test.png" alt="reference 1" style="border-color:red" /></p>
<p><img src="http://example.com/test.png" alt="reference 2" style="border-color:red" /></p>
.
Document[0, 111]
  Paragraph[0, 21] isTrailingBlankLine
    ImageRef[1, 20] textOpen:[1, 2, "["] text:[2, 13, "reference 1"] textClose:[13, 14, "]"] referenceOpen:[14, 15, "["] reference:[15, 19, "test"] referenceClose:[19, 20, "]"]
      Text[2, 13] chars:[2, 13, "refer … nce 1"]
  Paragraph[22, 43] isTrailingBlankLine
    ImageRef[23, 42] textOpen:[23, 24, "["] text:[24, 35, "reference 2"] textClose:[35, 36, "]"] referenceOpen:[36, 37, "["] reference:[37, 41, "test"] referenceClose:[41, 42, "]"]
      Text[24, 35] chars:[24, 35, "refer … nce 2"]
  Reference[44, 81] refOpen:[44, 45, "["] ref:[45, 49, "test"] refClose:[49, 51, "]:"] urlOpen:[52, 53, "<"] url:[53, 80, "http://example.com/test.png"] urlClose:[80, 81, ">"]
  Paragraph[84, 111]
    AttributesNode[84, 110] textOpen:[84, 85, "{"] text:[85, 109, "style=\"border-color:red\""] textClose:[109, 110, "}"]
      AttributeNode[85, 109] name:[85, 90, "style"] sep:[90, 91, "="] valueOpen:[91, 92, "\""] value:[92, 108, "border-color:red"] valueClose:[108, 109, "\""]
````````````````````````````````


## Random Tests

```````````````````````````````` example(Random Tests: 1) options(no-text-attributes)
Sample text{.class-name}
.
<p class="class-name">Sample text</p>
.
Document[0, 24]
  Paragraph[0, 24]
    Text[0, 11] chars:[0, 11, "Sampl …  text"]
    AttributesNode[11, 24] textOpen:[11, 12, "{"] text:[12, 23, ".class-name"] textClose:[23, 24, "}"]
      AttributeNode[12, 23] name:[12, 13, "."] value:[13, 23, "class-name"] isImplicit isClass
````````````````````````````````


Immediately attached to previous text will apply to the sibling text node

```````````````````````````````` example Random Tests: 2
Sample text{.class-name}
.
<p><span class="class-name">Sample text</span></p>
.
Document[0, 25]
  Paragraph[0, 25]
    TextBase[0, 11] chars:[0, 11, "Sampl …  text"]
      Text[0, 11] chars:[0, 11, "Sampl …  text"]
    AttributesNode[11, 24] textOpen:[11, 12, "{"] text:[12, 23, ".class-name"] textClose:[23, 24, "}"]
      AttributeNode[12, 23] name:[12, 13, "."] value:[13, 23, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example(Random Tests: 3) options(no-text-attributes)
Paragraph{style="color:red"}
.
<p style="color:red">Paragraph</p>
.
Document[0, 28]
  Paragraph[0, 28]
    Text[0, 9] chars:[0, 9, "Paragraph"]
    AttributesNode[9, 28] textOpen:[9, 10, "{"] text:[10, 27, "style=\"color:red\""] textClose:[27, 28, "}"]
      AttributeNode[10, 27] name:[10, 15, "style"] sep:[15, 16, "="] valueOpen:[16, 17, "\""] value:[17, 26, "color:red"] valueClose:[26, 27, "\""]
````````````````````````````````


```````````````````````````````` example Random Tests: 4
Paragraph{style="color:red"}
.
<p><span style="color:red">Paragraph</span></p>
.
Document[0, 29]
  Paragraph[0, 29]
    TextBase[0, 9] chars:[0, 9, "Paragraph"]
      Text[0, 9] chars:[0, 9, "Paragraph"]
    AttributesNode[9, 28] textOpen:[9, 10, "{"] text:[10, 27, "style=\"color:red\""] textClose:[27, 28, "}"]
      AttributeNode[10, 27] name:[10, 15, "style"] sep:[15, 16, "="] valueOpen:[16, 17, "\""] value:[17, 26, "color:red"] valueClose:[26, 27, "\""]
````````````````````````````````


```````````````````````````````` example(Random Tests: 5) options(no-text-attributes)
Paragraph {style="color:red"}
.
<p style="color:red">Paragraph</p>
.
Document[0, 29]
  Paragraph[0, 29]
    Text[0, 9] chars:[0, 9, "Paragraph"]
    AttributesNode[10, 29] textOpen:[10, 11, "{"] text:[11, 28, "style=\"color:red\""] textClose:[28, 29, "}"]
      AttributeNode[11, 28] name:[11, 16, "style"] sep:[16, 17, "="] valueOpen:[17, 18, "\""] value:[18, 27, "color:red"] valueClose:[27, 28, "\""]
````````````````````````````````


```````````````````````````````` example(Random Tests: 6) options(no-text-attributes)
Sample text **bold**{.class-name}
.
<p>Sample text <strong class="class-name">bold</strong></p>
.
Document[0, 33]
  Paragraph[0, 33]
    Text[0, 12] chars:[0, 12, "Sampl … text "]
    StrongEmphasis[12, 20] textOpen:[12, 14, "**"] text:[14, 18, "bold"] textClose:[18, 20, "**"]
      Text[14, 18] chars:[14, 18, "bold"]
    AttributesNode[20, 33] textOpen:[20, 21, "{"] text:[21, 32, ".class-name"] textClose:[32, 33, "}"]
      AttributeNode[21, 32] name:[21, 22, "."] value:[22, 32, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example(Random Tests: 7) options(no-text-attributes)
Sample text **bold** {.class-name}
.
<p class="class-name">Sample text <strong>bold</strong></p>
.
Document[0, 34]
  Paragraph[0, 34]
    Text[0, 12] chars:[0, 12, "Sampl … text "]
    StrongEmphasis[12, 20] textOpen:[12, 14, "**"] text:[14, 18, "bold"] textClose:[18, 20, "**"]
      Text[14, 18] chars:[14, 18, "bold"]
    AttributesNode[21, 34] textOpen:[21, 22, "{"] text:[22, 33, ".class-name"] textClose:[33, 34, "}"]
      AttributeNode[22, 33] name:[22, 23, "."] value:[23, 33, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example Random Tests: 8
Sample<!----> text{.class-name}
.
<p>Sample<!----><span class="class-name"> text</span></p>
.
Document[0, 32]
  Paragraph[0, 32]
    Text[0, 6] chars:[0, 6, "Sample"]
    HtmlInlineComment[6, 13] chars:[6, 13, "<!---->"]
    TextBase[13, 18] chars:[13, 18, " text"]
      Text[13, 18] chars:[13, 18, " text"]
    AttributesNode[18, 31] textOpen:[18, 19, "{"] text:[19, 30, ".class-name"] textClose:[30, 31, "}"]
      AttributeNode[19, 30] name:[19, 20, "."] value:[20, 30, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example Random Tests: 9
Sample text **bold<!----> text{.class-name}**
.
<p>Sample text <strong>bold<!----><span class="class-name"> text</span></strong></p>
.
Document[0, 46]
  Paragraph[0, 46]
    Text[0, 12] chars:[0, 12, "Sampl … text "]
    StrongEmphasis[12, 45] textOpen:[12, 14, "**"] text:[14, 43, "bold<!----> text{.class-name}"] textClose:[43, 45, "**"]
      Text[14, 18] chars:[14, 18, "bold"]
      HtmlInlineComment[18, 25] chars:[18, 25, "<!---->"]
      TextBase[25, 30] chars:[25, 30, " text"]
        Text[25, 30] chars:[25, 30, " text"]
      AttributesNode[30, 43] textOpen:[30, 31, "{"] text:[31, 42, ".class-name"] textClose:[42, 43, "}"]
        AttributeNode[31, 42] name:[31, 32, "."] value:[32, 42, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example(Random Tests: 10) options(no-text-attributes)
Sample text **bold{.class-name}**
.
<p>Sample text <strong class="class-name">bold</strong></p>
.
Document[0, 33]
  Paragraph[0, 33]
    Text[0, 12] chars:[0, 12, "Sampl … text "]
    StrongEmphasis[12, 33] textOpen:[12, 14, "**"] text:[14, 31, "bold{.class-name}"] textClose:[31, 33, "**"]
      Text[14, 18] chars:[14, 18, "bold"]
      AttributesNode[18, 31] textOpen:[18, 19, "{"] text:[19, 30, ".class-name"] textClose:[30, 31, "}"]
        AttributeNode[19, 30] name:[19, 20, "."] value:[20, 30, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example Random Tests: 11
Sample text **bold{.class-name}**
.
<p>Sample text <strong><span class="class-name">bold</span></strong></p>
.
Document[0, 34]
  Paragraph[0, 34]
    Text[0, 12] chars:[0, 12, "Sampl … text "]
    StrongEmphasis[12, 33] textOpen:[12, 14, "**"] text:[14, 31, "bold{.class-name}"] textClose:[31, 33, "**"]
      TextBase[14, 18] chars:[14, 18, "bold"]
        Text[14, 18] chars:[14, 18, "bold"]
      AttributesNode[18, 31] textOpen:[18, 19, "{"] text:[19, 30, ".class-name"] textClose:[30, 31, "}"]
        AttributeNode[19, 30] name:[19, 20, "."] value:[20, 30, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example(Random Tests: 12) options(no-text-attributes)
Sample text **bold {.class-name}**
.
<p>Sample text <strong class="class-name">bold</strong></p>
.
Document[0, 34]
  Paragraph[0, 34]
    Text[0, 12] chars:[0, 12, "Sampl … text "]
    StrongEmphasis[12, 34] textOpen:[12, 14, "**"] text:[14, 32, "bold {.class-name}"] textClose:[32, 34, "**"]
      Text[14, 18] chars:[14, 18, "bold"]
      AttributesNode[19, 32] textOpen:[19, 20, "{"] text:[20, 31, ".class-name"] textClose:[31, 32, "}"]
        AttributeNode[20, 31] name:[20, 21, "."] value:[21, 31, "class-name"] isImplicit isClass
````````````````````````````````


```````````````````````````````` example(Random Tests: 13) options(no-text-attributes)
Sample text ![Sample Image](http://example.com){width=64 height=32}
.
<p>Sample text <img src="http://example.com" alt="Sample Image" width="64" height="32" /></p>
.
Document[0, 67]
  Paragraph[0, 67]
    Text[0, 12] chars:[0, 12, "Sampl … text "]
    Image[12, 47] textOpen:[12, 14, "!["] text:[14, 26, "Sample Image"] textClose:[26, 27, "]"] linkOpen:[27, 28, "("] url:[28, 46, "http://example.com"] pageRef:[28, 46, "http://example.com"] linkClose:[46, 47, ")"]
      Text[14, 26] chars:[14, 26, "Sampl … Image"]
    AttributesNode[47, 67] textOpen:[47, 48, "{"] text:[48, 66, "width=64 height=32"] textClose:[66, 67, "}"]
      AttributeNode[48, 56] name:[48, 53, "width"] sep:[53, 54, "="] value:[54, 56, "64"]
      AttributeNode[57, 66] name:[57, 63, "height"] sep:[63, 64, "="] value:[64, 66, "32"]
````````````````````````````````


```````````````````````````````` example(Random Tests: 14) options(no-text-attributes)
* list item{style="color:red"}
* list item {style="color:blue"}
.
<ul>
  <li style="color:red">list item</li>
  <li style="color:blue">list item</li>
</ul>
.
Document[0, 63]
  BulletList[0, 63] isTight
    BulletListItem[0, 31] open:[0, 1, "*"] isTight
      Paragraph[2, 31]
        Text[2, 11] chars:[2, 11, "list item"]
        AttributesNode[11, 30] textOpen:[11, 12, "{"] text:[12, 29, "style=\"color:red\""] textClose:[29, 30, "}"]
          AttributeNode[12, 29] name:[12, 17, "style"] sep:[17, 18, "="] valueOpen:[18, 19, "\""] value:[19, 28, "color:red"] valueClose:[28, 29, "\""]
    BulletListItem[31, 63] open:[31, 32, "*"] isTight
      Paragraph[33, 63]
        Text[33, 42] chars:[33, 42, "list item"]
        AttributesNode[43, 63] textOpen:[43, 44, "{"] text:[44, 62, "style=\"color:blue\""] textClose:[62, 63, "}"]
          AttributeNode[44, 62] name:[44, 49, "style"] sep:[49, 50, "="] valueOpen:[50, 51, "\""] value:[51, 61, "color:blue"] valueClose:[61, 62, "\""]
````````````````````````````````


```````````````````````````````` example Random Tests: 15
* list item{style="color:red"}
* list item {style="color:blue"}
.
<ul>
  <li><span style="color:red">list item</span></li>
  <li style="color:blue">list item</li>
</ul>
.
Document[0, 64]
  BulletList[0, 64] isTight
    BulletListItem[0, 31] open:[0, 1, "*"] isTight
      Paragraph[2, 31]
        TextBase[2, 11] chars:[2, 11, "list item"]
          Text[2, 11] chars:[2, 11, "list item"]
        AttributesNode[11, 30] textOpen:[11, 12, "{"] text:[12, 29, "style=\"color:red\""] textClose:[29, 30, "}"]
          AttributeNode[12, 29] name:[12, 17, "style"] sep:[17, 18, "="] valueOpen:[18, 19, "\""] value:[19, 28, "color:red"] valueClose:[28, 29, "\""]
    BulletListItem[31, 64] open:[31, 32, "*"] isTight
      Paragraph[33, 64]
        Text[33, 42] chars:[33, 42, "list item"]
        AttributesNode[43, 63] textOpen:[43, 44, "{"] text:[44, 62, "style=\"color:blue\""] textClose:[62, 63, "}"]
          AttributeNode[44, 62] name:[44, 49, "style"] sep:[49, 50, "="] valueOpen:[50, 51, "\""] value:[51, 61, "color:blue"] valueClose:[61, 62, "\""]
````````````````````````````````


empty tight items without attributes should not wrap spans

```````````````````````````````` example(Random Tests: 16) options(no-text-attributes)
paragraph 1

* list item 2
* list item 1{style="color:red"}
.
<p>paragraph 1</p>
<ul>
  <li>list item 2</li>
  <li style="color:red">list item 1</li>
</ul>
.
Document[0, 59]
  Paragraph[0, 12] isTrailingBlankLine
    Text[0, 11] chars:[0, 11, "parag … aph 1"]
  BulletList[13, 59] isTight
    BulletListItem[13, 27] open:[13, 14, "*"] isTight
      Paragraph[15, 27]
        Text[15, 26] chars:[15, 26, "list  … tem 2"]
    BulletListItem[27, 59] open:[27, 28, "*"] isTight
      Paragraph[29, 59]
        Text[29, 40] chars:[29, 40, "list  … tem 1"]
        AttributesNode[40, 59] textOpen:[40, 41, "{"] text:[41, 58, "style=\"color:red\""] textClose:[58, 59, "}"]
          AttributeNode[41, 58] name:[41, 46, "style"] sep:[46, 47, "="] valueOpen:[47, 48, "\""] value:[48, 57, "color:red"] valueClose:[57, 58, "\""]
````````````````````````````````


## Paragraphs

Only attributes, go to paragraph's previous, if none to paragraph's parent

To previous sibling

```````````````````````````````` example Paragraphs: 1
paragraph 1

{style="color:red"}
.
<p style="color:red">paragraph 1</p>
.
Document[0, 33]
  Paragraph[0, 12] isTrailingBlankLine
    Text[0, 11] chars:[0, 11, "parag … aph 1"]
  Paragraph[13, 33]
    AttributesNode[13, 32] textOpen:[13, 14, "{"] text:[14, 31, "style=\"color:red\""] textClose:[31, 32, "}"]
      AttributeNode[14, 31] name:[14, 19, "style"] sep:[19, 20, "="] valueOpen:[20, 21, "\""] value:[21, 30, "color:red"] valueClose:[30, 31, "\""]
````````````````````````````````


To parent

```````````````````````````````` example Paragraphs: 2
{style="color:red"}

paragraph 1
.
<p>paragraph 1</p>
.
Document[0, 33]
  Paragraph[0, 20] isTrailingBlankLine
    AttributesNode[0, 19] textOpen:[0, 1, "{"] text:[1, 18, "style=\"color:red\""] textClose:[18, 19, "}"]
      AttributeNode[1, 18] name:[1, 6, "style"] sep:[6, 7, "="] valueOpen:[7, 8, "\""] value:[8, 17, "color:red"] valueClose:[17, 18, "\""]
  Paragraph[21, 33]
    Text[21, 32] chars:[21, 32, "parag … aph 1"]
````````````````````````````````


To parent, in this case list

```````````````````````````````` example Paragraphs: 3
paragraph 1

* {style="color:red"} list item 1
* list item 2
.
<p>paragraph 1</p>
<ul style="color:red">
  <li>list item 1</li>
  <li>list item 2</li>
</ul>
.
Document[0, 61]
  Paragraph[0, 12] isTrailingBlankLine
    Text[0, 11] chars:[0, 11, "parag … aph 1"]
  BulletList[13, 61] isTight
    BulletListItem[13, 47] open:[13, 14, "*"] isTight
      Paragraph[15, 47]
        AttributesNode[15, 34] textOpen:[15, 16, "{"] text:[16, 33, "style=\"color:red\""] textClose:[33, 34, "}"]
          AttributeNode[16, 33] name:[16, 21, "style"] sep:[21, 22, "="] valueOpen:[22, 23, "\""] value:[23, 32, "color:red"] valueClose:[32, 33, "\""]
        Text[35, 46] chars:[35, 46, "list  … tem 1"]
    BulletListItem[47, 61] open:[47, 48, "*"] isTight
      Paragraph[49, 61]
        Text[49, 60] chars:[49, 60, "list  … tem 2"]
````````````````````````````````


To parent, in this case list

```````````````````````````````` example Paragraphs: 4
paragraph 1

* list item 2
* {style="color:red"} list item 1
.
<p>paragraph 1</p>
<ul style="color:red">
  <li>list item 2</li>
  <li>list item 1</li>
</ul>
.
Document[0, 61]
  Paragraph[0, 12] isTrailingBlankLine
    Text[0, 11] chars:[0, 11, "parag … aph 1"]
  BulletList[13, 61] isTight
    BulletListItem[13, 27] open:[13, 14, "*"] isTight
      Paragraph[15, 27]
        Text[15, 26] chars:[15, 26, "list  … tem 2"]
    BulletListItem[27, 61] open:[27, 28, "*"] isTight
      Paragraph[29, 61]
        AttributesNode[29, 48] textOpen:[29, 30, "{"] text:[30, 47, "style=\"color:red\""] textClose:[47, 48, "}"]
          AttributeNode[30, 47] name:[30, 35, "style"] sep:[35, 36, "="] valueOpen:[36, 37, "\""] value:[37, 46, "color:red"] valueClose:[46, 47, "\""]
        Text[49, 60] chars:[49, 60, "list  … tem 1"]
````````````````````````````````


## Headings Tests

```````````````````````````````` example Headings Tests: 1
Heading
=======
.
<h1 id="heading">Heading</h1>
.
Document[0, 16]
  Heading[0, 15] text:[0, 7, "Heading"] textClose:[8, 15, "======="]
    Text[0, 7] chars:[0, 7, "Heading"]
````````````````````````````````


```````````````````````````````` example Headings Tests: 2
Heading with emoji :+1:
=======================
.
<h1 id="heading-with-emoji">Heading with emoji <img src="/img/plus1.png" alt="emoji people:+1" height="20" width="20" align="absmiddle" /></h1>
.
Document[0, 48]
  Heading[0, 47] text:[0, 23, "Heading with emoji :+1:"] textClose:[24, 47, "======================="]
    Text[0, 19] chars:[0, 19, "Headi … moji "]
    Emoji[19, 23] textOpen:[19, 20, ":"] text:[20, 22, "+1"] textClose:[22, 23, ":"]
      Text[20, 22] chars:[20, 22, "+1"]
````````````````````````````````


```````````````````````````````` example(Headings Tests: 3) options(no-text-attributes)
Heading{#id1} with multiple{#id2} anchors{#id3}
===============================================
.
<h1 id="id3">Heading with multiple anchors</h1>
.
Document[0, 95]
  Heading[0, 95] text:[0, 47, "Heading{#id1} with multiple{#id2} anchors{#id3}"] textClose:[48, 95, "==============================================="]
    Text[0, 7] chars:[0, 7, "Heading"]
    AttributesNode[7, 13] textOpen:[7, 8, "{"] text:[8, 12, "#id1"] textClose:[12, 13, "}"]
      AttributeNode[8, 12] name:[8, 9, "#"] value:[9, 12, "id1"] isImplicit isId
    Text[13, 27] chars:[13, 27, " with … tiple"]
    AttributesNode[27, 33] textOpen:[27, 28, "{"] text:[28, 32, "#id2"] textClose:[32, 33, "}"]
      AttributeNode[28, 32] name:[28, 29, "#"] value:[29, 32, "id2"] isImplicit isId
    Text[33, 41] chars:[33, 41, " anchors"]
    AttributesNode[41, 47] textOpen:[41, 42, "{"] text:[42, 46, "#id3"] textClose:[46, 47, "}"]
      AttributeNode[42, 46] name:[42, 43, "#"] value:[43, 46, "id3"] isImplicit isId
````````````````````````````````


```````````````````````````````` example Headings Tests: 4
Heading{#id1} with multiple{#id2} anchors{#id3}
===============================================
.
<h1 id="heading-with-multiple-anchors"><span id="id1">Heading</span><span id="id2"> with multiple</span><span id="id3"> anchors</span></h1>
.
Document[0, 96]
  Heading[0, 95] text:[0, 47, "Heading{#id1} with multiple{#id2} anchors{#id3}"] textClose:[48, 95, "==============================================="]
    TextBase[0, 7] chars:[0, 7, "Heading"]
      Text[0, 7] chars:[0, 7, "Heading"]
    AttributesNode[7, 13] textOpen:[7, 8, "{"] text:[8, 12, "#id1"] textClose:[12, 13, "}"]
      AttributeNode[8, 12] name:[8, 9, "#"] value:[9, 12, "id1"] isImplicit isId
    TextBase[13, 27] chars:[13, 27, " with … tiple"]
      Text[13, 27] chars:[13, 27, " with … tiple"]
    AttributesNode[27, 33] textOpen:[27, 28, "{"] text:[28, 32, "#id2"] textClose:[32, 33, "}"]
      AttributeNode[28, 32] name:[28, 29, "#"] value:[29, 32, "id2"] isImplicit isId
    TextBase[33, 41] chars:[33, 41, " anchors"]
      Text[33, 41] chars:[33, 41, " anchors"]
    AttributesNode[41, 47] textOpen:[41, 42, "{"] text:[42, 46, "#id3"] textClose:[46, 47, "}"]
      AttributeNode[42, 46] name:[42, 43, "#"] value:[43, 46, "id3"] isImplicit isId
````````````````````````````````


```````````````````````````````` example(Headings Tests: 5) options(no-text-attributes)
Heading{#id1} with multiple{#id2 style="color:red"} anchors{#id3}
=================================================================
.
<h1 id="id3" style="color:red">Heading with multiple anchors</h1>
.
Document[0, 131]
  Heading[0, 131] text:[0, 65, "Heading{#id1} with multiple{#id2 style=\"color:red\"} anchors{#id3}"] textClose:[66, 131, "================================================================="]
    Text[0, 7] chars:[0, 7, "Heading"]
    AttributesNode[7, 13] textOpen:[7, 8, "{"] text:[8, 12, "#id1"] textClose:[12, 13, "}"]
      AttributeNode[8, 12] name:[8, 9, "#"] value:[9, 12, "id1"] isImplicit isId
    Text[13, 27] chars:[13, 27, " with … tiple"]
    AttributesNode[27, 51] textOpen:[27, 28, "{"] text:[28, 50, "#id2 style=\"color:red\""] textClose:[50, 51, "}"]
      AttributeNode[28, 32] name:[28, 29, "#"] value:[29, 32, "id2"] isImplicit isId
      AttributeNode[33, 50] name:[33, 38, "style"] sep:[38, 39, "="] valueOpen:[39, 40, "\""] value:[40, 49, "color:red"] valueClose:[49, 50, "\""]
    Text[51, 59] chars:[51, 59, " anchors"]
    AttributesNode[59, 65] textOpen:[59, 60, "{"] text:[60, 64, "#id3"] textClose:[64, 65, "}"]
      AttributeNode[60, 64] name:[60, 61, "#"] value:[61, 64, "id3"] isImplicit isId
````````````````````````````````


```````````````````````````````` example Headings Tests: 6
Heading{#id1} with multiple{#id2 style="color:red"} anchors{#id3}
=================================================================
.
<h1 id="heading-with-multiple-anchors"><span id="id1">Heading</span><span id="id2" style="color:red"> with multiple</span><span id="id3"> anchors</span></h1>
.
Document[0, 132]
  Heading[0, 131] text:[0, 65, "Heading{#id1} with multiple{#id2 style=\"color:red\"} anchors{#id3}"] textClose:[66, 131, "================================================================="]
    TextBase[0, 7] chars:[0, 7, "Heading"]
      Text[0, 7] chars:[0, 7, "Heading"]
    AttributesNode[7, 13] textOpen:[7, 8, "{"] text:[8, 12, "#id1"] textClose:[12, 13, "}"]
      AttributeNode[8, 12] name:[8, 9, "#"] value:[9, 12, "id1"] isImplicit isId
    TextBase[13, 27] chars:[13, 27, " with … tiple"]
      Text[13, 27] chars:[13, 27, " with … tiple"]
    AttributesNode[27, 51] textOpen:[27, 28, "{"] text:[28, 50, "#id2 style=\"color:red\""] textClose:[50, 51, "}"]
      AttributeNode[28, 32] name:[28, 29, "#"] value:[29, 32, "id2"] isImplicit isId
      AttributeNode[33, 50] name:[33, 38, "style"] sep:[38, 39, "="] valueOpen:[39, 40, "\""] value:[40, 49, "color:red"] valueClose:[49, 50, "\""]
    TextBase[51, 59] chars:[51, 59, " anchors"]
      Text[51, 59] chars:[51, 59, " anchors"]
    AttributesNode[59, 65] textOpen:[59, 60, "{"] text:[60, 64, "#id3"] textClose:[64, 65, "}"]
      AttributeNode[60, 64] name:[60, 61, "#"] value:[61, 64, "id3"] isImplicit isId
````````````````````````````````


```````````````````````````````` example Headings Tests: 7
Heading{#id1} with multiple{#id2 style="color:red"} anchors{#id3} {#id4}
========================================================================
.
<h1 id="id4"><span id="id1">Heading</span><span id="id2" style="color:red"> with multiple</span><span id="id3"> anchors</span></h1>
.
Document[0, 146]
  Heading[0, 145] text:[0, 72, "Heading{#id1} with multiple{#id2 style=\"color:red\"} anchors{#id3} {#id4}"] textClose:[73, 145, "========================================================================"]
    TextBase[0, 7] chars:[0, 7, "Heading"]
      Text[0, 7] chars:[0, 7, "Heading"]
    AttributesNode[7, 13] textOpen:[7, 8, "{"] text:[8, 12, "#id1"] textClose:[12, 13, "}"]
      AttributeNode[8, 12] name:[8, 9, "#"] value:[9, 12, "id1"] isImplicit isId
    TextBase[13, 27] chars:[13, 27, " with … tiple"]
      Text[13, 27] chars:[13, 27, " with … tiple"]
    AttributesNode[27, 51] textOpen:[27, 28, "{"] text:[28, 50, "#id2 style=\"color:red\""] textClose:[50, 51, "}"]
      AttributeNode[28, 32] name:[28, 29, "#"] value:[29, 32, "id2"] isImplicit isId
      AttributeNode[33, 50] name:[33, 38, "style"] sep:[38, 39, "="] valueOpen:[39, 40, "\""] value:[40, 49, "color:red"] valueClose:[49, 50, "\""]
    TextBase[51, 59] chars:[51, 59, " anchors"]
      Text[51, 59] chars:[51, 59, " anchors"]
    AttributesNode[59, 65] textOpen:[59, 60, "{"] text:[60, 64, "#id3"] textClose:[64, 65, "}"]
      AttributeNode[60, 64] name:[60, 61, "#"] value:[61, 64, "id3"] isImplicit isId
    AttributesNode[66, 72] textOpen:[66, 67, "{"] text:[67, 71, "#id4"] textClose:[71, 72, "}"]
      AttributeNode[67, 71] name:[67, 68, "#"] value:[68, 71, "id4"] isImplicit isId
````````````````````````````````


## TOC

Default rendering with emphasis

```````````````````````````````` example TOC: 1
[TOC] 

# Heading **some bold** 1 {#heading-1}
## Heading 1.1 _some italic_ {#heading-2}
### Heading 1.1.1
### Heading 1.1.2  **_some bold italic_** {#heading-3}
.
<ul>
  <li><a href="#heading-2">Heading 1.1 <em>some italic</em></a>
    <ul>
      <li><a href="#heading-111">Heading 1.1.1</a></li>
      <li><a href="#heading-3">Heading 1.1.2  <strong><em>some bold italic</em></strong></a></li>
    </ul>
  </li>
</ul>
<h1 id="heading-1">Heading <strong>some bold</strong> 1</h1>
<h2 id="heading-2">Heading 1.1 <em>some italic</em></h2>
<h3 id="heading-111">Heading 1.1.1</h3>
<h3 id="heading-3">Heading 1.1.2  <strong><em>some bold italic</em></strong></h3>
.
Document[0, 162]
  TocBlock[0, 7] openingMarker:[0, 1] tocKeyword:[1, 4] closingMarker:[4, 5]
  Heading[8, 46] textOpen:[8, 9, "#"] text:[10, 46, "Heading **some bold** 1 {#heading-1}"]
    Text[10, 18] chars:[10, 18, "Heading "]
    StrongEmphasis[18, 31] textOpen:[18, 20, "**"] text:[20, 29, "some bold"] textClose:[29, 31, "**"]
      Text[20, 29] chars:[20, 29, "some bold"]
    Text[31, 33] chars:[31, 33, " 1"]
    AttributesNode[34, 46] textOpen:[34, 35, "{"] text:[35, 45, "#heading-1"] textClose:[45, 46, "}"]
      AttributeNode[35, 45] name:[35, 36, "#"] value:[36, 45, "heading-1"] isImplicit isId
  Heading[47, 88] textOpen:[47, 49, "##"] text:[50, 88, "Heading 1.1 _some italic_ {#heading-2}"]
    Text[50, 62] chars:[50, 62, "Headi …  1.1 "]
    Emphasis[62, 75] textOpen:[62, 63, "_"] text:[63, 74, "some italic"] textClose:[74, 75, "_"]
      Text[63, 74] chars:[63, 74, "some  … talic"]
    AttributesNode[76, 88] textOpen:[76, 77, "{"] text:[77, 87, "#heading-2"] textClose:[87, 88, "}"]
      AttributeNode[77, 87] name:[77, 78, "#"] value:[78, 87, "heading-2"] isImplicit isId
  Heading[89, 106] textOpen:[89, 92, "###"] text:[93, 106, "Heading 1.1.1"]
    Text[93, 106] chars:[93, 106, "Headi … 1.1.1"]
  Heading[107, 161] textOpen:[107, 110, "###"] text:[111, 161, "Heading 1.1.2  **_some bold italic_** {#heading-3}"]
    Text[111, 126] chars:[111, 126, "Headi … 1.2  "]
    StrongEmphasis[126, 148] textOpen:[126, 128, "**"] text:[128, 146, "_some bold italic_"] textClose:[146, 148, "**"]
      Emphasis[128, 146] textOpen:[128, 129, "_"] text:[129, 145, "some bold italic"] textClose:[145, 146, "_"]
        Text[129, 145] chars:[129, 145, "some  … talic"]
    AttributesNode[149, 161] textOpen:[149, 150, "{"] text:[150, 160, "#heading-3"] textClose:[160, 161, "}"]
      AttributeNode[150, 160] name:[150, 151, "#"] value:[151, 160, "heading-3"] isImplicit isId
````````````````````````````````


## Trailing Spaces

```````````````````````````````` example Trailing Spaces: 1
text {attribute=value   }
.
<p attribute="value">text</p>
.
Document[0, 26]
  Paragraph[0, 26]
    Text[0, 4] chars:[0, 4, "text"]
    AttributesNode[5, 25] textOpen:[5, 6, "{"] text:[6, 21, "attribute=value"] textClose:[24, 25, "}"]
      AttributeNode[6, 21] name:[6, 15, "attribute"] sep:[15, 16, "="] value:[16, 21, "value"]
````````````````````````````````


## Translator Encoding

```````````````````````````````` example Translator Encoding: 1
text {#_1_}{#_2_}
.
<p><span id="_2_">text </span></p>
.
Document[0, 18]
  Paragraph[0, 18]
    TextBase[0, 5] chars:[0, 5, "text "]
      Text[0, 5] chars:[0, 5, "text "]
    AttributesNode[5, 11] textOpen:[5, 6, "{"] text:[6, 10, "#_1_"] textClose:[10, 11, "}"]
      AttributeNode[6, 10] name:[6, 7, "#"] value:[7, 10, "_1_"] isImplicit isId
    AttributesNode[11, 17] textOpen:[11, 12, "{"] text:[12, 16, "#_2_"] textClose:[16, 17, "}"]
      AttributeNode[12, 16] name:[12, 13, "#"] value:[13, 16, "_2_"] isImplicit isId
````````````````````````````````


## Source Position Attribute

```````````````````````````````` example(Source Position Attribute: 1) options(src-pos, no-text-attributes)
Paragraph{style="color:red"}
.
<p md-pos="0-28" style="color:red">Paragraph</p>
.
Document[0, 28]
  Paragraph[0, 28]
    Text[0, 9] chars:[0, 9, "Paragraph"]
    AttributesNode[9, 28] textOpen:[9, 10, "{"] text:[10, 27, "style=\"color:red\""] textClose:[27, 28, "}"]
      AttributeNode[10, 27] name:[10, 15, "style"] sep:[15, 16, "="] valueOpen:[16, 17, "\""] value:[17, 26, "color:red"] valueClose:[26, 27, "\""]
````````````````````````````````


```````````````````````````````` example(Source Position Attribute: 2) options(src-pos)
Paragraph{style="color:red"}
.
<p md-pos="0-28"><span style="color:red">Paragraph</span></p>
.
Document[0, 28]
  Paragraph[0, 28]
    TextBase[0, 9] chars:[0, 9, "Paragraph"]
      Text[0, 9] chars:[0, 9, "Paragraph"]
    AttributesNode[9, 28] textOpen:[9, 10, "{"] text:[10, 27, "style=\"color:red\""] textClose:[27, 28, "}"]
      AttributeNode[10, 27] name:[10, 15, "style"] sep:[15, 16, "="] valueOpen:[16, 17, "\""] value:[17, 26, "color:red"] valueClose:[26, 27, "\""]
````````````````````````````````


