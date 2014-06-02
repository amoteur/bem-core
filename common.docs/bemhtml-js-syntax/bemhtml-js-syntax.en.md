# JS syntax of BEMHTML. Migration Guide

<a name="intro"></a>
## Introduction

This article is intended for web developers and HTML layout designers, which use [BEM methodology](http://bem.info/method/) and [BEMHTML template engine](http://bem.info/libs/bem-core/1.0.0/bemhtml/reference/).

The article describes:

* [compatibility](#compat) of BEMHTML templates, provided with different syntax;
* [template execution environment](#runmode) settings;
* [BEMHTML templates processing](#run) processing flow;
* BEMHTML template engine standard operations implementation using [JavaScript syntax](#sintax);
* templates [transform algorithm](#steps): from abbreviated syntax to JS.

This article doesn't contain any information about setting development environment or template compilation procedure, BEHTML template engine features, or [BEMJSON](http://ru.bem.info/libs/bem-core/1.0.0/bemhtml/reference/#bemjson) input data format.


<a name="general"></a>
## General information

Starting from a [bem-core](https://github.com/bem/bem-core/tree/v1) library version 1.0.0 you can execute BEMHTML templates that were written using JavaScript syntax.
The library supports two template syntax types: **acronym** and a JS syntax.
Since the `bem-core` library introduction template acronym syntax is considered deprecated.

JacaScript syntax of BEMHTML templates has the following advantages:

* development environments and tools support (because you code in JavaScript);
  * code highlight;
  * JSHint, JSLint etc;
* fast [compilation](#run), especially in dev execution environment;
* besides, template code can run directly in [dev execution environment](#runmode), which makes debugging simpler.

All the major features of BEMHTML template engine are still relevant when using JS syntax.


***


<a name="install"></a>
## Work start

BEMHTML templates which use JS syntax can be executed in all BEM platform components that use bem-core library.

To migrate to JS syntax you can:

* use project-stub version that uses bem-core library ([bem-core](https://github.com/bem/project-stub/tree/bem-core) branch).

* install all the packages needed by yourself: [bem-xjst](https://github.com/bem/bem-xjst), [bemhtml-compat](https://github.com/bem/bemhtml-compat), [BEMHTML API v2](https://github.com/bem/bem-core/blob/v1/.bem/techs/bemhtml.js) module from bem-tools package.

BEMHTML technology module, which supports JS syntax, implemented using [API v2 технологии](http://ru.bem.info/tools/bem/bem-tools/tech-modules/#api) from bem-tools. To use it you need a bem-tools package version 0.6.4 or higher.


***


<a name="compat"></a>
## Templates compatibility

BEMHTML templates implemented using different syntax can be used in one project.

During the execution a template engine translates acronym-syntax templates into JS syntax. Syntax conversion is performed by [bemhtml-compat](https://github.com/bem/bemhtml-compat) module. For more information read [template execution](#run).

Template syntax is automatically detected by a template engine on compile time.

To make it easier to distinguish template files with different syntax different suffixes can be used:

* for acronym-syntax - `.bemhtml`;
* for JS syntax - `.bemhtml.xjst`.

**Important:** template file can't use two syntax types at the same time.


***


<a name="sintax"></a>
## BEMHTML templates JavaScript syntax

To simplify JavaScript-syntax BEMHTML templates creation, [bem-xjst](https://github.com/bem/bem-xjst) module is used.

BEM-XJST is a BEM oriented helpers kit, which extends standard XJST-syntax.

It allows JS syntax BEMHTML templates to use:

* helpers for writing subpredicates, related to BEM subject domain;
* Helpers for a subpredicate detection based on a mode;
* helpers for XJST constructions `apply` and `applyNext` using default mode;
* `applyCtx` construction.

BEMXJST is a superset of a [XJST template language](https://github.com/veged/xjst), which in turn is a Javascript superset.

BEM-XJST uses canonical XJST-syntax, extended by rules, related to BEM subject domain. This kind of implementation allows BEMHTML templates with JS syntax to be executed in dev environment without preliminarily compilation.

**NB:** `apply` and `applyNext` methods behavior is extended in BEM-XJST compared to XJST. Methods can take a string literal of a statement that can be cast to a string, instead of assignment statements. It means "set string as a mode".

For example `apply('content')` can be written like `apply({ _mode: 'content' })`.


***


<a name="template"></a>

#### Template

Template consists of two parts - `predicate` and `body`.
Every predicate is a list of one or more subpredicates (conditions).

Template predicate is true only when all the subpredicates are true.

In acronym-syntax predicate and body are divided by a colon, subpredicates are divided by commas.

```
subpredicate1, subpredicate2, subpredicate3: body
```

In JS syntax `match` keyword is introduced for a template description.

`match` keyword is a helper method, which have a subpredicate list as an argument. Method returns function which takes a template body as an argument

```js
match(subpredicate1, subpredicate2, subpredicate3)(body);
```

For example:

```js
match(this.block === 'link', this._mode === 'tag', this.ctx.url)('a');
```

The same subpredicate list can be chained:

```js
match(this.block === 'link').match(this._mode === 'tag').match(this.ctx.url)('a');
```

The examples above are identical and will look like this in acronym-syntax:

```js
block link, tag, this.ctx.url: 'a'
```

----
**NB:** if there's more than one template defined for a context, the one listed the last in BEMHTML file will have a higher priority.
Specific templates should be defined after those more general ones.


***


<a name="subpredicate"></a>

#### Subpredicates

Template predicate is a condition list, describing template application moment. Template subpredicate is a simple condition.

BEMHTML have the following condition types:

* input BEM-tree match;
* mode;
* arbitrary condition.

##### Input BEM-tree match

Input BEM-TREE match conditions allow you to describe a template applicability using BEM related entities: blocks and elements names, modifiers names and values.

The following keywords are used in predicates for BEM entities:

BEM entity – **Block**.
***

Keyword – `block`.

Arguments:
* block name

Acronym-syntax example:
`block b-menu` or `block 'b-menu'` or `block 'b' + '-menu'`

JS syntax example:
`block('b-menu')` or `block('b' + '-menu')`


***

BEM entity – **Element**.
***

Keyword - `elem`.

Arguments:
* element name

Simplified syntax example:

```
block b-menu, elem item
```

JS syntax example:

```js
block('b-menu').elem('item')
```

***

BEM entity – **Block modifier**.

Keyword - `mod`.

Arguments:
* block modifier name
* block modifier value

Simplified syntax example:

```
block b-head-logo, mod size big
```

JS syntax example:


```js
block('b-head-logo').mod('size', 'big')
```

***

BEM entity – Element modifier

Keyword - `elemMod`

Arguments
* element modifier name
* element modifier value

Simplified syntax example:

```
block b-head-logo, elem text, elemMod size big
```

JS syntax example:

```js
block('b-head-logo')(elem('text').elemMod('size', 'big'))
```

***


**NB:** identifiers for blocks, elements and modifiers build a string, consisting of latin letters and hyphen. Any JS statement can be used as an identifier, which will be cast to a string. For example `block('b-head-logo')` is the same as `block('b-' + 'head' + '-logo')`.

JS syntax keywords related to BEM subject domain are used for reduced subpredicate definition.
In particular, they allow to omit `match` keyword for BEM subpredicates.

The following predicates are equal:

```js
match(this.block === 'foo').match(this.elem === 'bar')
```

```js
block('foo').elem('bar')
```

To run templates without compilation an `elemMatch` keyword was added. It is used for arbitrary element subpredicate notation:

```js
block('my-block')
    .elemMatch(function() { return this.elem === 'e1' || this.elem === 'e2'  })
        .tag()('span')
```

The thing is, during the processing templates without subpredicates, that describe elements, have a `!this.elem` subpredicate automatically added to them. It allows to avoid block template to be applied to the elements of the same block.

As you can see the example above will not work without `elemMatch`:

```js
block('my-block')
    .match(function() { return this.elem === 'e1' || this.elem === 'e2'  })
        .tag()('span')
```

During the processing `!this.elem` subpredicate will be added to it.


***


<a name="moda"></a>

##### The mode

The name of one of the [standard mode](http://ru.bem.info/libs/bem-core/1.0.0/bemhtml/reference/#standardmoda) can be used as a subpredicate. It means that the predicate will be true when a corresponding mode is set.

The following keywords are used for standard mode validation:

* `def`
* `tag`
* `attrs`
* `bem`
* `mix`
* `cls`
* `js`
* `jsAttr`
* `content`

To define a subpredicate using one of the standard modes in JS syntax, helper corresponding to keyword can be used.

For example ``tag()('span')`` is the same as ``match(this._mode === 'tag')('span')``

When using simplified syntax, any subpredicate consisting only from identifier (`[a-zA-Z0-9-]+`) is interpreted as a non-standard mode's name.
For example, subpredicate `my-mode` is equal to ``this._mode === 'my-mode'``.

In JS syntax to define a subpredicate using a non-standard mode a `mode` keyword is used. It's a helper method similar to `match` construction. It takes a string as an argument and returns a function for template's body.
Considering all this `mode('my-mode')` is the same as ``this._mode === 'my-mode'``.


***


##### Arbitrary conditions
<a name="arbitrary_condition"></a>

Arbitrary conditions take into account not related to BEM data matches.
Any JavaScript statement can be used as an arbitrary condition. It'll be cast to boolean value.

***
**NB:** it is preferable to write down an arbitrary conditions using a canonical XJST form:

```js
predicate statement === value
```

where:

* `predicate statement` – is any JavaScript statement, cast to boolean value;
* `value` – any JavaScript statement.

In JS syntax to write down any predicate statement `match` keyword is used. For example:

```js
match(this.ctx.url)(
        tag()('a'), // Passes a string with a tag 'a' as an argument to a "tag" mode.
        attrs()({ href: this.ctx.url }) // Passes a hash with attributes as an argument to an "attrs" mode
    )`
```

Arbitrary subpredicate `this.ctx.url` will be true when an `url` field in a context will have a value assigned. In this case template's body will be executed.


***


<a name="body"></a>

#### Temolate's body

Template's body is an expression, and a result of it's execution is used to generate an HTML output.
All of the following can be a template's body:

* JavaScript statement:
    * simplified syntax:

```
predicate: JS statement
```

    * JS syntax:

```js
match(predicate)(JS statement)
```

* Block of a JavaScript code:
    * simplified syntax:

```
predicate: { JS code }
```

    * JS syntax:

```js
match(predicate)(function() { JS code })
```

In JS syntax template's body is passed as an argument to a function, which was returned by a `match` method and by helpers for BEM entity or mode.

Syntax:

```
standard-mode()(body)

mode('non-standard-mode')(body)

BEM-entity('entity-name')(body)

match(arbitrary predicate)(body)

**NB:** it's important to remember that template's body is passed to a function returned by helper method, not to a helper itself.

Wrong:

```js
block('b1').tag('span')
```

Right:

```js
block('b1').tag()('span')
```

***

#### XJST expressions

For templates execution in a modified contex an [XJST expressions](http://ru.bem.info/libs/bem-core/1.0.0/bemhtml/reference/#xjst) can be used in a templates implemented using JS syntax.

They work the same way as if they were in a templates implemented in simplified syntax.

#### Templates nesting

If there's a few templates using the same subpredicates, they can be written down as a nested structure to reduce the code duplication.

Curly braces are used to indicate nesting in a simplified syntax. They begin after the predicate's common part. Inside there is a block of code, containing different predicates parts and corresponding template bodies.

```
subpredicate1 {
  subpredicate2: body1
  subpredicate3: body2
}
```

This expression is equal to:

```
subpredicate 1, subpredicate 2: body1
subpredicate 1, subpredicate 3: body2
```

In JS syntax nested subpredicates are written in a template's body. In other words, subpredicates are passed as an arguments to a function, returned by a `match` method and helpers of a BEM entities and modes.

For example, template in a simplified syntax:

```js
this.block === 'link' {
    this._mode === 'tag': 'a'
    this._mode === 'attrs': { href: this.ctx.url }
}
```

can be written like this:

```js
match(this.block === 'link')(
   match(this._mode === 'tag')('a'),
   match(this._mode === 'attrs')({ href: this.ctx.url })
)
```
It's equal to the following expression:

```js
match(this.block === 'link').match(this._mode === 'tag')('a');
match(this.block === 'link').match(this._mode === 'attrs')({ href: this.ctx.url });
```


***

To make an expression simpler, you can use helpers for BEM entities and standard modes names.

Previous example can be written as:

```js
block('link')(
    tag()('a'),
    attrs()({ href: this.ctx.url })
)
```


***

BEMHTML template can contain a template body and subtemplates at the same nesting level.

In a simplified syntax for this feature `true` keyword was used:

```
block link, tag, this.ctx.url {
    true: 'a'
    mods not-link yes: 'span'
}
```

When using JS syntax a template's body is passed as an argument to a function returned by helper. It must be the first argument, and subtemplates can be passed after it:

```js
block('link').tag().match(this.ctx.url)(
    'a',
    mod('not-link', 'yes')('span')
)
```

***

Template's nesting depth is not limited:

```js
block('link')(
    tag()('span'),
    match(this.ctx.url)(
        tag()('a'),
        attrs()({ href: this.ctx.url })
    )
)
```

**NB:** it is not recommended to use ternary operators or JavaScript conditional operators to write nested templates. This kind of expressions will not be optimized in production execution environment.


<a name="runmode"></a>
## Template execution environment

BEMHTML template engine can work in two different modes, depending on the **execution environment settings**. The engine itself supports two execution environments:

* development environment (dev-environment);
* production environment.

The main difference is that in a production environment a template XJST translation takes place, and it gives optimized JavaScript as a result. It increases project build time because of templates compilation, but makes it work faster at runtime.

Execution environment is chosen by a template engine depending on a `process.env.BEMHTML_ENV` environment variable value. When the value is `development` it uses a development environment, and a production environment in all other cases.

The choice of an environment affect a template execution flow, be it a simplified syntax or a JS syntax implementation.


<a name="run"></a>
## BEMHTML templates processing

Template engine processes BEMHTML templates in two stages:

* compilation;
* execution.


<a name="runpre"></a>
### Templates compilation

Templates are compiled differently depending on execution environment settings and template syntax.


<a name="runclassic"></a>
#### Simplified syntax

No matter what an execution environment setting are, the following step are performed:

* all the BEMHTML templates present in a build are stored in a bundle file;
* templates in a bundle file are converted to XJST syntax.

**In a production development environment:**

XJST translation is performed, which results in an optimized template JavaScript code generation.


<a name="runjs"></a>
#### JavaScript syntax

No matter what an execution environment setting are, the following steps are performed:

* all the BEMHTML templates present in a build are stored in a bundle file.

**In a production development environment:**

XJST translation is performed, which results in an optimized template JavaScript code generation.


<a name="runmain"></a>
### Templates execution

JavaScript code that was obtained on a template compile time is executed the same way for all syntax and settings variation:

* template engine takes a BEM-tree as an input data in [BEMJSOM](http://ru.bem.info/libs/bem-core/1.0.0/bemhtml/reference/#bemjson) format;
* sequentially go through an input BEM-tree nodes;
    * data structure called [context](http://ru.bem.info/libs/bem-core/1.0.0/bemhtml/reference/#context) is built during the BEMJSON tree processing;
* HTML output is generated in cycle for every BEM-entity;
    * HTML output is recursively generated for every nested BEM entity;
    * writing to HTML result fragments buffer is performed, element by element.


***

<a name="table"></a>
### Template engine standard operations in different syntax table


| Operation | Simplified syntax | JS syntax |
| ------------- |-------------|------------- |-------------|------------- |-------------|
| BEM entity match | `block b-my-block : body` | `block('b-my-block')(body)` |
| Standard mode match | `tag : 'a'`  | `tag()('a')` |
| Non-standard modifier match |  `custom-mode : body` | `mode('custom-mode')(body)`  |
| Arbitrary condition match | `block link, this.ctx.url, tag: 'a'` | `block('link').match(this.ctx.url).tag()('a')`  |


<a name="steps"></a>
### Simplified syntax to JS syntax conversion algorythm

Templates implemented in a simplified syntax can be converted to use a JS syntax with the following simple transformation.


<a name="steps-table"></a>
#### Step-by-step template convertion table
Template conversion algorithm can be briefly described as a table below.

| Step number   | Stage       | Description  | Conversion pattern |
| ------------- |-------------|------------- |-------------|------------- |-------------|
|  1  | BEM-oriented subpredicates conversion | Surround the BEM-entities names with a single quotation marks | `b1` → `'b1'` |
|  2 |  | Replace a BEM entities abbreviations with helpers | `block 'b1'` → `block('b1')` |
|  3 |  | Replace all commas separating subpredicates with dots | `,`  → `.` |
|  4 | Arbitrary subpredicates conversion | Wrap an arbitrary subpredicate in `match` helper | `arbitrary-subpredicate` -> `match(arbitrary-subpredicate)` |
|  5 | Modes' subpredicate conversion | Replace standard modes names with helpers | `tag` → `tag()` |
|  6 |  | Wrap remaining subpredicated in a `mode` helper and add apostrophes | `some-mode` → `mode('some-mode')` |
|  7 | Template body and nested expressions conversion | Wrap a template's body in a parentheses and remove preceding colon | ` : ...`  →  `(...)` |
|  8 |  | Divide nested templates by commas | `block('b1'){ tag()('a') elem('e1').tag('b') }` → `block('b1'){ tag()('a'), elem('e1').tag('b') }` |
|  9 |  | Replace nesting curly braces with parentheses | `block('b1'){ tag()('a'), elem('e1').tag('b') }` → `block('b1')(tag()('a'), elem('e1').tag('b'))` |


<a name="steps-examples"></a>
#### Template convertion example

**Template 1.** Sets an `img` tag for the `logo` block.

Simplified syntax:

```
block logo {
  tag: 'img'
}
```


JS syntax::

```js
block('logo').tag()('img')
```

***


**Template 2.** Sets and `img` tag and corresponding attribute set for the `logo` block.

Simplified syntax:

```
block logo {
  tag: 'img'
  attrs: ({alt: 'logo', href: 'http://...'})
}
```


JS syntax:

```js
block('logo')(
  tag()('img'),
  attrs()({alt: 'logo', href: 'http://...'})
)
```

***


**Template 3.** Sets an `html` tag for a `b-page` block and forbids class generation with the BEM entity's name.

Simplified syntax:

```
block b-page {
  tag: 'html'
  bem: false
}
```

JS syntax:

```js
block('b-page')(
  tag()('html'),
  bem()(false)
)
```

***


**Template 4.** Sets a tag for a `b-text` block elements which was defined in an input BEMJSON. If there's an `id` field defined for an element in an input data it sets an `id` attribute it's value.

Simplified syntax:

```
block b-text {

    this.elem, tag: this.ctx.elem

    this.elem, this.ctx.id, attrs: { id: this.ctx.id  }

}
```


JS syntax:

```js
block('b-text')(

    elemMatch(this.elem).tag()(this.ctx.elem),

    elemMatch(this.elem).match(this.ctx.id).attrs()({ id: this.ctx.id  })

)
```


***


**Template 5.** Sets a `span` tag for a `b-bla` block. If there's a `o-mode` modifier set to `v2` in an input data, changes tag to an `a`. After that the `m2` element modifier with `v2` value is added to the block, indicating that the block contains JavaScript.

Simplified syntax:

```
block b-bla {
  tag:'span'
  mod 0-mode v2, tag:'a'
  mix: [ { elemMods: { m2: 'v2' }} ]
  js: true
}
```


JS syntax:

```js
block('b-bla')(
  tag()('span'),
  mod('0-mode', 'v2').tag()('a'),
  mix()([{ mods: { m2: 'v2' } }]),
  js()(true)
)
```

***


**Template 6.** Wraps `b-inner` block in `b-wrapper` block. Input BEMJSON fragment in a `default` mode is substituted by `b-wrapper` block, containing the original input data.

Simplified syntax:

```
block b-inner, default: applyCtx({ block: 'b-wrapper', content: this.ctx })
```


JS syntax:


While using a fragment of input BEMJSON `this.ctx` with an `applyCtx` structure in dev-environment, endless cycle is possible during the execution. To avoid it you need to add a flag, indicating that template was already processed, and subpredicate to check flag's value:

```js
block('b-inner')(def()
    .match(!this.ctx._wrapped)(function() {
            var ctx = this.ctx;
            ctx._wrapped=true;
            applyCtx({ block: 'b-wrapper', content: ctx })
   })
)
```


To avoid declaring a local variable XJST expression `local` can be used to add the flag preventing endless cycle. It allows to execute template in a modified context:

```js
block('b-inner')(def()
    .match(!this.ctx._wrapped)(function() {
            local({ 'ctx._wrapped': true })(applyCtx({ block: 'b-wrapper', content: this.ctx }))
   }))
```

***


**Template 7.** Sets the `span` tag by default for the element `e1` of a `b-bla` block. If there's an `url` field defined in an input data, changes the tag to `a` sets the field content as a `href` attribute value.
In case of a match with a non-standard mode `reset` a `href` attribute value is set to `undefined`.

Simplified syntax:

```
block b-link, elem e1 {
  tag: 'span'
  this.ctx.url {
     tag: 'a'
     attrs: { href: this.ctx.url }
     reset {
         attrs: { href: undefined }
      }
   }
}
```


JS syntax:

```js
block('b-link').elem('e1') (
  tag()('span'),
  match(this.ctx.url)(
     tag()('a'),
     attrs()({ href: this.ctx.url }),
     mode('reset')(
         attrs()({ href: undefined })
      )
   )
)
```

***
