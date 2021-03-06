# Common code

## Build configuration

The common module must declare the dependencies on kvision-common-types and kvision-common-remote modules.

{% code-tabs %}
{% code-tabs-item title="build.gradle" %}
```groovy
apply plugin: 'kotlin-platform-common'
apply plugin: 'kotlinx-serialization'

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-common:${kotlinVersion}"
    compile "pl.treksoft:kvision-common-types:${kvisionVersion}"
    compile "pl.treksoft:kvision-common-remote:${kvisionVersion}"
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Implementation

The common module is the place where you define how your remote services should look and how they should work. You can define as many services as you wish, and they can have as many methods as you need. It's a good practice to split your services based on their context and functions.

{% hint style="info" %}
Note: When using authentication on the server side, you can usually apply different authentication options to different services.
{% endhint %}

Every service definition in the common module is built with three elements: an interface definition, an expected class declaration and a `KVServiceManager` object definition. 

### An interface

Designing the interface is probably the most important step, and during this process you have to stick to some important rules.

#### A method must be suspending

This requirement allows the framework to translate asynchronous calls into synchronous-like code.

#### A method must have from zero to five parameters

This is the restriction of the current version of the framework. It may change in the future.

#### A method can't return nullable value

`Unit` return type is not supported as well.

#### Method parameters and return value must be of supported types

Supported types are:

* all basic Kotlin types \(`String`, `Boolean`, `Int`, `Long`, `Short`, `Char`, `Byte`,  `Float`, `Double`\)
* `Enum` class defined in common code
* `pl.treksoft.kvision.types.Date`, which is automatically mapped to `kotlin.js.Date` on the client side and `java.util.Date` on the server side
* any class defined in the common code with `@Serializable` annotation
* a `List<T>`, where T is one of the above types
* a `T?`, where T is one of the above types \(allowed only as method parameters - see previous rule\)

{% hint style="info" %}
Note: Default parameters values are supported.
{% endhint %}

Even with the above restrictions, the set of supported types is quite rich and you should be able to model almost any use case for your applications. With the help of `@Serializable` annotation you can always wrap any data structure into a serializable data class. It's also a simple way to pass around the parameters count limit.

{% hint style="info" %}
Note: You have to add `@ContextualSerialization` annotations to your `Date` fields in order to explicitly allow serialization with the KVision context. You can also use `@file:ContextualSerialization(Date::class)` file annotation if you want to keep your model classes cleaner. You can find more information about this annotation in the [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/custom_serializers.md#contextualserialization-annotation) documentation.
{% endhint %}

With an interface defined in the common module, the type safety of your whole application is forced at a compile time. Any incompatibility between the client and the server code will be marked as a compile-time error.

```kotlin
@Serializable
data class Address(
    val id: Int? = 0,
    val firstName: String? = null,
    val lastName: String? = null,
    val email: String? = null,
)

enum class Sort {
    FN, LN, E
}

interface IAddressService {
    suspend fun getAddressList(search: String?, sort: Sort): List<Address>
    suspend fun addAddress(address: Address): Address
    suspend fun updateAddress(id: Int, address: Address): Address
    suspend fun deleteAddress(id: Int): Boolean
}
```

### An expected class

An expected class declaration have to implement your service interface. The compiler forces you to implement this class in both the client and the server module. 

```kotlin
expect class AddressService : IAddressService
```

### `KVServiceManager` object

By instantiating `KVServiceManager` object you actually define connections between the client and the server code. These connections are created by a series of calls to the `bind` methods, which must be called inside the initialization block of the object - one call for every method.

```kotlin
object AddressServiceManager : KVServiceManager<AddressService>(AddressService::class) {
    init {
        GlobalScope.launch(start = CoroutineStart.UNDISPATCHED) {
            bind(IAddressService::getAddressList)
            bind(IAddressService::addAddress)
            bind(IAddressService::updateAddress)
            bind(IAddressService::deleteAddress)
        }
    }
}
```

{% hint style="info" %}
Note: Because of the bug in the Kotlin/JS compiler \([KT-27855](https://youtrack.jetbrains.com/issue/KT-27855)\) at the moment it is necessary to wrap these calls into a special `GlobalScope.launch` coroutine builder.
{% endhint %}

The `bind` method saves the information what endpoint should be called and how to process parameters when the method is being called from the client code. At the same time it attaches the service implementation as a handler for this endpoint on the server side, taking care of processing parameters and the result value.

Typically you do not have to use any parameters of the `bind` method, other then the callable references to your service methods. By default KVision will use HTTP POST server calls and automatically generated endpoint names. But you can also change the HTTP method and the endpoint URL with additional parameters of `bind`.

```kotlin
object AddressServiceManager : KVServiceManager<AddressService>(AddressService::class) {
    init {
        GlobalScope.launch(start = CoroutineStart.UNDISPATCHED) {
            bind(IAddressService::getAddressList, HttpMethod.POST)
            bind(IAddressService::addAddress, HttpMethod.PUT, "addAddress")
            bind(IAddressService::updateAddress, HttpMethod.POST, "updateAddress")
            bind(IAddressService::deleteAddress, HttpMethod.DELETE, "removeAddress")
        }
    }
}
```

{% hint style="info" %}
Note: All KVision endpoint names \(even those with user defined names\) are prefixed with "/kv/" to avoid potential conflicts with other endpoints.
{% endhint %}

{% hint style="info" %}
Note: HTTP GET can be used only for methods without parameters.
{% endhint %}

Properly initialized service manager object will be used in both the client and the server modules.

