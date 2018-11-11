# Basic components

## HTML markup

KVision contains classes for creating typical HTML markup of a web page. The main class for this purpose is [`pl.treksoft.kvision.html.Tag`](https://rjaros.github.io/kvision/api/pl.treksoft.kvision.html/-tag/index.html), which allows you to render any HTML element. This class is also a container, so instances of `Tag` can be nested inside other `Tag` objects. Two subclasses of `Tag` - `Div` and `Label` - are shortcuts to easily render `DIV` and `SPAN` elements. With DSL builders you can declare simple markup as follows:

```kotlin
div {
    label("A simple label")
    label("A label with custom CSS styling") {
        fontFamily = "Times New Roman"
        fontSize = 32.px
        textDecoration = TextDecoration(TextDecorationLine.UNDERLINE, TextDecorationStyle.DOTTED, Col.RED)
    }
    tag(TAG.CODE, "Some text written in <code></code> HTML tag.")
}
```

And it will be rendered as:

```markup
<div>
<span>A simple label</span>
<span style="font-family: Times New Roman; font-size: 32px; text-decroration: underline dotted red">
A label with custom CSS styling
</span>
<code>Some text written in &lt;code&gt;&lt;/code&gt; HTML tag.</code>
</div>
```

You can use the [`pl.treksoft.kvision.html.ListTag`](https://rjaros.github.io/kvision/api/pl.treksoft.kvision.html/-list-tag/index.html) class to create  HTML lists:

```kotlin
div {
    listTag(ListType.UL, listOf("First list element", "Second list element", "Third list element"))
}
```

And you can use classes from the [`pl.treksoft.kvision.table.*`](https://rjaros.github.io/kvision/api/pl.treksoft.kvision.table/index.html) package to create HTML tables:

```kotlin
table(
    listOf("Column 1", "Column 2", "Column 3"),
    setOf(TableType.BORDERED, TableType.CONDENSED, TableType.STRIPED, TableType.HOVER), responsive = true
) {
    row {
        cell("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce nec fringilla turpis.")
        cell("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce nec fringilla turpis.")
        cell("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce nec fringilla turpis.")
    }
    row {
        cell("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce nec fringilla turpis.")
        cell("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce nec fringilla turpis.")
        cell("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce nec fringilla turpis.")
    }
}
```

To create a link use [`pl.treksoft.kvision.html.Link`](https://rjaros.github.io/kvision/api/pl.treksoft.kvision.html/-link/index.html):

```text
div {
    link("A link to Google", "http://www.google.com")
}
```

## Dynamic content

Note that all the generated HTML markup is fully dynamic and is bound to the state of the KVision components. If you change content or styling properties of any visible object, it will automatically re-render the content shown in the browser. These changes can be triggered by any sources \(timers, coroutines, network events\) but probably most often they will be triggered by user interaction.

```kotlin
link("A link to Google", "http://www.google.com").setEventListener<Link> {
    mouseover = {
        self.label = "A link to Microsoft"
        self.url = "http://www.microsoft.com"
    }
}
```

## Rich text

All components which render text content allow to declare such content as "rich text". If you set the `rich` property to `true`, the content will be treated as HTML. The framework takes care of making the output markup valid HTML if it's not. This code:

```kotlin
tag(
    TAG.P,
    "Rich <b>text</b> <i>written</i> with <span style=\"font-family: Verdana; font-size: 14pt\">" +
            "any <strong>forma</strong>tting</span>.",
    rich = true
)
```

will render:

```markup
<p><span>Rich <b>text</b> <i>written</i> with 
<span style="font-family: Verdana; font-size: 14pt;">any <strong>forma</strong>tting
</span>.</span></p>
```

{% hint style="info" %}
Notice the extra `<span>` element surrounding the given content. 
{% endhint %}
