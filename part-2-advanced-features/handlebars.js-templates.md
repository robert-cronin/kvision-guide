# Handlebars.js templates

Support for [Handlebars.js](https://handlebarsjs.com/) templates is contained in kvision-handlebars module and is based on webpack's [handlebars loader](https://github.com/pcardune/handlebars-loader). Templates are automatically transformed to JavaScript functions during the build process of the application.

You put your template files \(with `*.hbs` extension\) into `src/main/resources/hbs` directory of your application. Then in your code you can just `require` your `*.hbs` files. All components which render textual content \(`pl.treksoft.kvision.html.Tag` class and subclasses\) have `template` and `templates` properties. The second one allows you to define different templates for other supported languages \(see. [Internationalization](../part-1-fundamentals/internationalization.md)\).

```kotlin
div {
    template = require("hbs/template.hbs")
}

div {
    templates = mapOf(
        "en" to require("hbs/template.en.hbs"),
        "pl" to require("hbs/template.pl.hbs")
    )
}
```

To actually call the template function and render its output in your app, you need to set some data by setting `templateData` property. It's required to use plain JavaScript object. To generate one from your data  you can use `obj` function builder.

```kotlin
import pl.treksoft.kvision.utils.obj

div {
    template = require("hbs/template.hbs")
    templateData = obj {
        name = "Alan"
        hometown = "Somewhere, TX"
        kids = arrayOf(obj {
            name = "Jimmy"
            age = "12"
        }, obj {
            name = "Sally"
            age = "5"
        })
    }
}
```

You can also use `toObj` extension function if you store your data in a class \(or classes\) with `@Serializable` annotation.

```kotlin
import kotlinx.serialization.Serializable
import pl.treksoft.kvision.utils.JSON.toObj

@Serializable
data class Kid(val name: String, val age: Int)
@Serializable
data class Person(val name: String, val hometown: String, val kids: List<Kid>)

val person = Person("Alan", "Somwhere, TX", listOf(Kid("Jimmy", 12), Kid("Sally", 5)))

div {
    template = require("hbs/template.hbs")
    templateData = person.toObj()
}

```


