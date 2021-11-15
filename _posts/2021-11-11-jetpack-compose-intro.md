---
layout: post
title:  "Jetpack Compose - an introduction to state"
date:   2021-11-14 22:00:00 -0300
categories: android jetpack-compose
description: A quick look into state management in Jetpack Compose
image: jetpack_compose_post_og_image.png
---

<meta name="twitter:card" content="summary">
<meta name="twitter:creator" content="@nahidalgo01">
{% if page.title %}
  <meta name="twitter:title" content="{{ page.title }}">
{% else %}
  <meta name="twitter:title" content="{{ site.title }}">
{% endif %}
{% if page.url %}
  <meta name="twitter:url" content="{{ site.url }}{{ page.url }}">
{% endif %}
{% if page.description %}
  <meta name="twitter:description" content="{{ page.description }}">
{% else %}
  <meta name="twitter:description" content="{{ site.description }}">
{% endif %}
{% if page.image %}
  <meta name="twitter:image:src" content="{{ site.url }}/assets/{{ page.image }}">
{% else %}
  <meta name="twitter:image:src" content="{{ site.url }}/assets/logo.png">
{% endif %}

<meta content="{{ site.title }}" property="og:site_name">
{% if page.title %}
  <meta content="{{ page.title }}" property="og:title">
{% else %}
  <meta content="{{ site.title }}" property="og:title">
{% endif %}
{% if page.title %}
  <meta content="article" property="og:type">
{% else %}
  <meta content="website" property="og:type">
{% endif %}
{% if page.description %}
  <meta content="{{ page.description }}" property="og:description">
{% else %}
  <meta content="{{ site.description }}" property="og:description">
{% endif %}
{% if page.url %}
  <meta content="{{ site.url }}{{ page.url }}" property="og:url">
{% endif %}
{% if page.date %}
  <meta content="{{ page.date | date_to_xmlschema }}" property="article:published_time">
  <meta content="{{ site.url }}/about/" property="article:author">
{% endif %}
{% if page.image %}
  <meta content="{{ site.url }}/assets/{{ page.image }}" property="og:image">
{% else %}
  <meta content="{{ site.url }}/assets/logo.png" property="og:image">
{% endif %}
{% if page.categories %}
  {% for category in page.categories limit:1 %}
  <meta content="{{ category }}" property="article:section">
  {% endfor %}
{% endif %}
{% if page.tags %}
  {% for tag in page.tags %}
  <meta content="{{ tag }}" property="article:tag">
  {% endfor %}
{% endif %}

<img src="{{ page.image }}">

## What is Jetpack Compose
Jetpack Compose is the new declarative UI Toolkit for Android built with Kotlin. It represents a meaningful change in how Android apps' UIs are developed. It works with composable functions, which are functions that, given data as input, convert it into UI. All of the composable functions must be annotated with `@Compose`, which shows the compiler it is going to convert data into UI. The Hello World of Compose is:

```kotlin
@Composable
fun Greeting(name: String) {
	Text("Hello, $name")
}
```

From the example, it's possible to note that given a name as input, the `Text()` function is called, which itself is a composable function. The `Text()` function is responsible for creating the text UI element itself on the screen. The `name` parameter passed as input, is the data that will be used to describe our UI. Also, composable functions don't need to return anything as it only describes what the UI needs to look like with a given state. The state is the key to building great and responsive reactive UIs.

In this article, I'll take a shallow look at how the state works with Compose and how Compose integrates with the other Jetpack tools.

## Why do you talk so much about state
State isn't always a concept that's easy to understand. But keep in mind that "state", in the context of Jetpack Compose, is only a short word for the data used to build the UI. When the state changes, the data changes and so does the UI. In the `Greeting` example above, when the name changes, the `Greeting` composable function is rebuilt to display the new name.

A simple example is a product catalog screen having three states: Products, Loading, and Error. When the app logic fetches products shown to the user, we describe the UI to show these products as a list of Product cards, for example. If the app logic is still loading the products, we can display a circular progress indicator showing the user that the data isn't loaded yet. If the app logic encounters an error when fetching the products, the Error state can be triggered, and we show an error message to the user.

## How the UI reacts to state (and what triggers state changes)

If we think of an app more abstractly, we can conclude that it has four main parts: State, UI Events, the UI itself, and the User. When the user clicks on a button it triggers a UI Event. That UI Event then causes a change on the State by executing business logic, on a ViewModel, for example. The new generated State triggers the UI to rebuild. The newly generated UI is shown to the user, and the cycle continues.

![Jetpack Compose Architecture](/assets/JetpackArchitecture.png)

Notice that this process is a cycle, a natural one, and easy to understand. This concept is explained brilliantly and in more depth by [Andre 'Staltz' Medeiros](https://staltz.com) in his ["What if the user was a function?" JSConf Budapest 2015 presentation](https://youtu.be/1zj7M1LnJV4). In his presentation, Andre talks about the Model View Intent (MVI) architecture but the MVVM architecture together with Compose approaches MVI. The intent is like the UI Events. The model is the ViewModel. And the view is the Compose components.

## State in composables

In compose, State is deeply integrated with the Composition, which is the description of the UI to be built. A composable can be a stateful one if it initializes the state inside itself. A stateful component triggers the redrawing process of every composable function that references the state every time the state it holds changes. For example, let's imagine a simple send message component, which has a text field and a button that performs the sending process.

```kotlin
@Composable
fun SendMessageTextField(
    onSendMessage: (String) -> Unit
) {
    val text = mutableStateOf('')

    Row {
        TextField(value = text.value, onValueChange = { text.value = it })
        IconButton(onClick = onSendMessage(text)) {
            Icon(imageVector = Icons.Filled.Send, "Send message")
        }
    }
}
```

In the above example, `text` is a state of the component, and it holds a string value. `text` is mutable, and every time the `onValueChange` of the `TextField` is called, we assign it to the user's input. The MutableState in Kotlin is defined as follows:

```kotlin
interface MutableState<T> : State<T> {
    override var value: T
}
```

Even though `val text = mutableStateOf('')` has the intended effect of creating a state, it has two main problems:
- The state is not persisted through recomposition, that means, every time the component is rebuilt, the state goes back to the default value.
- Every time you need the value, it's necessary to use `.value` to get and set it.

We can solve both problems by using the `remember` composable and its `by` delegate syntax. The following code solves it:

```kotlin
@Composable
fun SendMessageTextField(
    onSendMessage: (String) -> Unit
) {
    val text by remeber { mutableStateOf('') }

    Row {
        TextField(value = text, onValueChange = { text = it })
        IconButton(onClick = onSendMessage(text)) {
            Icon(imageVector = Icons.Filled.Send, "Send message")
        }
    }
}
```

With this implementation, the state is persisted through recomposition but still, if there are configuration changes (screen rotation, for example), the state is lost again. If we need to persist the state even through configuration changes, the solution is to use `rememberSaveable`.



```kotlin
@Composable
fun SendMessageTextField(
    onSendMessage: (String) -> Unit
) {
    val text by remeberSaveable { mutableStateOf('') }

    Row {
        TextField(value = text, onValueChange = { text = it })
        IconButton(onClick = onSendMessage(text)) {
            Icon(imageVector = Icons.Filled.Send, "Send message")
        }
    }
}
```

The `rememberSaveable` composable stores the state in memory and persists even when the view is destroyed and rebuilt. Also, using the `by` delegate allows us to use the state name directly when we want to get or set it.

The problem with stateful components is that they are more difficult to test and less reusable. The component controls its state and that makes it harder to do checks such as input validation. A better architecture is proposed next.

## The basic Jetpack Compose architecture

Jetpack Compose, together with the other Jetpack tools, makes for an interesting architecture. With Compose components, we can describe the UI and, for every interaction that the user performs with it, we can expose a function. That function is our UI Event which can call a method from a ViewModel. The ViewModel holds the State and can modify it based on the UI Events received. The new State is then listened to by the UI, which reconstructs itself. Using the same previous composable, let's show how we can improve it.

```kotlin
@Composable
fun SendMessageTextField(
    text: String,
    onTextChanged: (String) -> Unit,
    onSendMessage: () -> Unit
) {
    Row {
        TextField(value = text, onValueChange = onTextChange)
        IconButton(onClick = onSendMessage) {
            Icon(imageVector = Icons.Filled.Send, "Send message")
        }
    }
}
```

On the example, we receive the text (or state) that we want to show and expose the UI events which can be handled in the ViewModel. This process is what is called state hoisting. State hoisting is the act of moving the state outside of the composable, making it stateless. This approach leads to better component reusability, testability, decoupling, and control over it. Hoisting is the recommended approach by the Android team to create components.

```kotlin
class MessageScreenViewModel() @ViewModelInject constructor(): ViewModel() {

    val uiState = LiveData ... // the state which will trigger UI changes

    fun getMessages() {
        // receives the new messages
        // and updates the uiState
    }

    fun onSendMessage() {
        // Sends the message
    }

    fun onTextChanged(name: String) {
        // performs action when the text changes
        // and updates uiState
    }
}
```

On the ViewModel side, we can create the methods that handle our UI Events and act according to our app logic. The UI State is a stream that can be listened to by the UI and can be a LiveData or StateFlow, the latter being the recommended approach by Google, that is observed by our UI.

```kotlin
@Composable
fun MessageScreen(
    viewModel: MessageScreenViewModel
) {
    val uiState = viewModel.uiState.observeAsState()

    SendMessageTextField(
        name = uiState.name,
        onTextChanged = viewModel.onTextChanged
        onSendMessage = viewModel.onSendMessage
    )
}
```

Here is where our `SendMessageTextField` state was hoisted to. This screen composable takes care of the state of the whole screen, and if we happened to have more composables, they could be organized and built with the `uiState`. The `observeAsState()` method is a special method to access the `ViewModel`'s `LiveData` as State of the composable itself. There are also analog methods for Flow and RxJava streams. When the stream receives a new value, the `uiState` changes and so the UI is rebuilt if needed. This is what is called recomposition. See that it's not necessary to use the `remember` composable here because the state comes from the ViewModel.

## Recomposition

"Ok, but all that UI rebuilding sure costs a lot of processing power", you would say. And you would indeed be correct. Rebuilding an entire UI can be very expensive. But, the Android team gave special attention to it. The result was what is called recomposition.

When the State data changes, the composable functions are called and redrawn, only if needed. Compose is smart enough to not redraw a component that hasn't had its data changed. For example:

```kotlin
@Composable
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("I've been clicked $clicks times")
    }
}
```

Given that the caller of this component updates the value of `clicks` every time `onClick` is called, compose only redraws (recompose) the `Text` composable function, because it is the only composable that uses the changed value. Other composables that does not reference the values that were updated, won't be recomposed.

## Conclusion

All of this info on state, can lead you into building great reactive UI using Jetpack Compose. The Jetpack tools integrate beautifully with Compose. In future articles, I'll build real world UIs to play around and see what is possible with Compose. I challenge you to do the same! If you like Jetpack Compose, I strongly recommend you to read more about it in the references below.

## References

[Thinking in Compose - Android Developers](https://developer.android.com/jetpack/compose/mental-model)

[Managing state - Android Developers](https://developer.android.com/jetpack/compose/state)