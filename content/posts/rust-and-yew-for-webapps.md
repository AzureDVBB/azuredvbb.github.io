---
title: "A guide for Simple, Stateful and Interactive Webapps in Rust with Yew v0.20.0"
date: 2023-04-09T19:37:15+02:00
draft: false
---

Rust has been a language of interest and trepadation to me for some time, with it's functional programming and strict rules (coming from python).
The one thing I have identified as the most difficult part in creating my projects is... programming a GUI (graphical user interface).

There are a host of competing frameworks and libraries, each with it's own trade-offs and complexity.
There's even a website to track to track the state of these, humorously called [are we gui yet?](https://www.areweguiyet.com/)

Suffice to say each application is it's own challange and deserves forethough as to what to use and how.

With all that said, my personal projects are much smaller in scope and definately not great lookers.
So I have decided to buld my applications for the web and leverage browsers to do the heavy lifting.

After several days and not insignificant trial, error and ChatGPT voodoo... I decided to write a blog post about how to do it so that others may come across it and have a more seamless experience.

## 0. Prerequisites

### 0.0. Knowledge

This guide expects familiarity with (but provides links to read on) the following topics:
* [Rust's project structure](https://doc.rust-lang.org/stable/book/ch01-02-hello-world.html)
* [Struct](https://doc.rust-lang.org/stable/book/ch05-00-structs.html)'s and [Trait](https://doc.rust-lang.org/stable/book/ch10-02-traits.html)'s
* Cursory knowledge on:
** [Borrow Checker](https://doc.rust-lang.org/stable/book/ch04-01-what-is-ownership.html) (ownership rules)
** [Datatypes](https://doc.rust-lang.org/stable/book/ch03-02-data-types.html)
** [HTML](https://en.wikipedia.org/wiki/HTML) and [CSS](https://en.wikipedia.org/wiki/CSS) syntax (for the gui)
** [preludes](https://doc.rust-lang.org/reference/names/preludes.html)
** [closures](https://doc.rust-lang.org/book/ch13-01-closures.html)
** [match statements](https://doc.rust-lang.org/book/ch06-02-match.html)
* [What macros are](https://doc.rust-lang.org/book/ch19-06-macros.html)
* [functions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#functions) and [methods](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#method-syntax)
* Know how to do things in the terminal

Knowing these apriori is nice, but you can skip this reading and refer to this section if you bump into the terms later on.

### 0.1. Software

It goes without saying that you need rust installed.
And that `cargo` installed binaries are available under your system's `PATH` variable so we can invoke them from the terminal.

To build our webapp and to run a local development server for it we will use [trunk](https://trunkrs.dev/).
```
cargo install trunk
```

We also need to add the [webAssembly (WASM)](https://webassembly.org/) as a build target so we can build our application.

```
rustup target add wasm32-unknown-unknown
```

## 1. Setting up our project

### 1.0. New binary crate

Let's create a new project called **yew-simplified** and go into it's directory.

```
cargo new yew-simplified
cd yew-simplified
```

It's good practice to verify our environment works *before* we start to tinker.
Which we can do simply by running:
```
cargo run
```

That should have printed out a 'Hello World!', confirming everything works.

### 1.1. Adding dependencies

Next we need to edit our `cargo.toml` file to add two new dependencies.
```toml
[dependencies]
stylist = { version = "0.12.0", features = ["yew_integration"] }
yew = { version = "0.20.0", features = ["csr"] }
```

[Yew](https://yew.rs/) is our main web-framework so we can build our GUI.
It is worth noting at this time that there are multiple ways of making a GUI with Yew, but the method I am presenting is, as of this writing, not explained enough that I understood it immediately.

[Stylist](https://crates.io/crates/stylist) on the other hand is one of the easier ways of adding CSS styles to our GUI to make it look pretty.
It also features compile-time checking for valid CSS so it helps during development as well!

### 1.2. Making an empty Landing Page

`trunk` expects a minimal `index.html` file to act as our webapp's landing page and will not build without it.

Let's create it then! It's very important to put it at the crate's root (where `cargo.toml` is) and not inside the `src/` directory.
Then fill it in with some basic HTML like so:
```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="Yew Simplified" charset="utf-8">
    <title>Yew Simplified</title>
  </head>
  <body style="margin: 0; padding: 0;" />
</html>
```

And that's done, now we have our project set up we can write our program.
`Yew` will be filling in the `<body>` tag with HTML during runtime to facilitate re-drawing our application.

We can test that our application works fine by running it with `trunk`:
```
trunk serve --open
```

This will build are (unoptimized) application, downloading and building dependancies along the way.
It will then launch a web-server and using the `--open` flag will open the locally hosted webapp in our system's default browser.

**NOTE:** So long as the `trunk` server is running, any changes to the project's files will prompt an automatic build and refresh on the opened page, so we do not need to do anything to see our changes take effect during development.
Additionally, you can use `trunk build` to build the website without hosting it.

## 2. Defining our application's datastructure

The rest of the logic and GUI we will code directly to our main source file under `src/main.rs`.
Later we can always refactor our code to separate out more complex segments into their own individual modules when and if needed.

So far the file should just hold the main function that the application executes whenever it is run.

```rust
fn main() {
    println!("Hello, world!");
}
```

### 2.1. Defining the GUI emitted messages

[Message passing]() is the primary way the interface will communicate with our Rust code to determine what to do.
If there are no messages, our code should do nothing at all, conserving system resources.

Rust offers a convenient and efficient way of defining messages like this called an `enum`.

Let's create a message that will be emitted when a button is pressed.
We'll add a little bit of complexity by identifying each button with an 'id' number.
This demonstrates how to the GUI can pass some meaningful data back to our Rust code.

```rust
enum AppMessage{
    ButtonPressed(usize),
}
```

**Note:** the message `ButtonPressed(usize)` means that the message `ButtonPressed` holds a value with type `usize`.
While we can substitute `usize` with any other type or struct, in this case we use it because the `usize` represents a pointer for our architecture (32bit, 64bit etc..) and arrays require this exact pointer type when using it to access an element ( for example: `MyArray[elementId]`).

### 2.2. Defining our application state struct

Now we need to actually define a `struct` to hold all of our application's state information, that is all of the persistent data our application needs to hold.

In this we'll just hold some text to display to the user which we will edit and change to show the GUI reacting.
We'll also store an array of texts which we'll use to place appropriately labelled buttons onto the screen.

```rust
struct App {
	displayed_text: String,
    button_texts: Vec<String>,
}
```

**Note:** for `button_texts` we used the type `Vec<String>`, which is just an array that hold values of a single type, `String` type in this case.
This is of course a simplification but for our use case holds true.

### 2.3. Implementing defaults for the application state data

So far we just defined what kind of data our application will store, but when we first load the application we need to initialize them with some default values.

We can accomplish this by implementing the `Default` trait for our `App` struct.

```rust
impl Default for App {
    fn default() -> Self {
        App {
            displayed_text: "Press any button to see a change!".to_string(),
           	button_texts: vec!(
				String::from("I like change"), 
				String::from("I hate change"),
			),
        }
    }
}
```

**Note:** string literals in Rust are of type `&str` (string slice) and need to be converted into `String` types, [you can read as to why with this link](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#memory-and-allocation).
You can see two ways to accomplish this conversions in the code above.

## 3. Using `Yew` and `Stylist` to draw our application to the screen

To reiterate once more, this is just one way of using Yew v0.20.0 to draw a web-based GUI.
This just so happens to be the way that I found works best, is understandable (by me) and I personally use.

Please refer to the [Yew documentation](https://yew.rs/) to learn more approaches.

### 3.0. Adding our dependencies

So far we've not needed to add any dependencies to our `src/main.rs` file.
Now however, we will be going deep into making the web GUI with `Yew` and then making it pretty with `Stylist` to embed CSS directly in our application.

Add the following to the top of `src/main.rs` :

```rust
use stylist::css;
use yew::prelude::*;
```

**Note:** stylist has many ways of applying CSS styles, including some `Yew` integration, it is well worth reading up on it inside it's documentation.
Using the `css!` macro is just one I have found to be simple and understandable.

### 3.1. Implementing the Yew `Component` trait on our App struct

Now to actually integrate our `App` struct with Yew we will need to implement the `Component` trait and it's 3 required methods: 

* `create` 	for initialization (we added default values in **step 2.3.**)
* `update` 	for reacting to GUI sent messages and running our code
* `view` 	for actually building the raw HTML that will be displayed as our GUI

Let's do that now without much functionality.

```rust
impl Component for App {
    type Message = AppMessage;
    type Properties = ();

    fn create(ctx: &Context<Self>) -> Self {
        Self::default()
    }

    fn update(&mut self, ctx: &Context<Self>, msg: Self::Message) -> bool {
		false
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
		html!{
			<p>{"Hello World!"}</p>
		}
    }
}

```

**Note:** the `type` keyword is a [type alias](https://doc.rust-lang.org/reference/items/type-aliases.html).
In this case we are not using `Properties` so we alias that to [the 'unit' type](https://doc.rust-lang.org/std/primitive.unit.html).

### 3.2. Starting the Yew renderer

The final thing to do is to actually call Yew to draw our application in the `main()` function.

This has changed drastically in Yew v0.20.0 compared to previous versions, mostly to support new functionality.
To draw our simple app, edit the `main` function as such:

```rust
fn main() {
	yew::Renderer::<App>::new().render();
}
```

If all went well we now have 'Hello World!' plastered on a blank page!

### 3.3. Using `Yew` and `Stylist` together

**Note:** Once more, there are many ways to do this.
It is just one of the simplest ways I am describing here.
For more information read the `Yew` and `Stylist` documentation.

With that said, let's use the `css!` macro to create a style and apply it to our 'Hello World!' paragraph!
Edit the `view` method as such:

```rust
    fn view(&self, ctx: &Context<Self>) -> Html {
        let hello_style = css!("
           background-color: coral;
        ");

        html!{
            <p class={hello_style}>
                {"Hello World!"}
            </p>
        }
    }
```

Take note how we capture the output of the `css!` macro into a variable and then use it inside Yew's `html!` macro, by putting it in curly braces `{}`.
With Yew we can run code while building the page's HTML and embed the results as valid HTML text.

Also note how we added the style by the paragraphs' `class=` attribute.
This is because Stylist creates unique style classes within the `<head>` of our page, then simply assigns the appropriate classes to our elements.

## 4. Adding interactivity to our application

### 4.1. A single button with a callback

The simplest form of interaction is clicking buttons, without much fanfare let's add one now!

Modify our `view` method as such:

```rust
fn view(&self, ctx: &Context<Self>) -> Html {
        let hello_style = css!("
           background-color: coral;
        ");

        let button_press = ctx.link().callback(|_| AppMessage::ButtonPressed(1));

        html!{
            <div>
                <p class={hello_style}>
                    {"Hello World!"}
                </p>
                <button onclick={button_press}>{ "iButton" }</button>
            </div>
        }
    }
```


Here we created a button event with `ctx.link().callback(...)` , a callback is a function that will be called when a specific event happens, storing the resulting callback in a variable.
This helps separate out the code we write outside of the `html!` so it is easier to read.

In this case the callback will invoke an anonymouse (unnamed) function we defined in a closure and return an `AppMessage` which we can later use to identify what event has fired.
The closure itself takes in a Yew even type (`MouseEvent` in this case) which we do not use, so we ignore it by assigning it to `_` .

**Note:** to read more on Yew context's [see the documentation](https://yew.rs/docs/next/concepts/contexts).
But in this guide, we essentially just use it as a way to pass messages back to our `App` struct.

### 4.2. Making the application display state data

So far the application displayed just a static, unchanging interface.
Now let's add some much needed interactivity.

Firstly, we are going to use our `App` struct's `displayed_text` to put text on the screen, instead of the standard hello world so far.

Let's modify our `view` method once more as follows:

```rust
fn view(&self, ctx: &Context<Self>) -> Html {
        let hello_style = css!("
           background-color: coral;
        ");

        let button_press = ctx.link().callback(|_| AppMessage::ButtonPressed(1));

        html!{
            <div>
                <p class={hello_style}>
                    {self.displayed_text.clone()}
                </p>
                <button onclick={button_press}>{ "iButton" }</button>
            </div>
        }
    }
```

Here we replaced our `"Hello World!"` with a reference to our desired data inside `App` struct.
And just like that we are halfway done!

**Note:** We needed to clone the data we use from `App` as the `html!` macro consumes it, taking ownership.
Because we only have a borrowed reference to it (read-only) we needed to create a clone of it for the macro to consume.

### 4.3. Updating view based on callback `AppMessage`s

Finally we can focus on running application code now we have event callbacks sending messages to run code to!

In this case, let's conditionally run code only when a message comes in that we have a defined behavior for.

Modify the `update` method as follows:

```rust
    fn update(&mut self, ctx: &Context<Self>, msg: Self::Message) -> bool {
        match msg {
            AppMessage::ButtonPressed(id) => {
                self.displayed_text = format!(
                    "You pressed the '{}' button with id '{}', press another!",
                    self.button_texts[id],
                    id,
                );
                true
            }
        }
    }
```

**Note:** in this case we return `true` from the update method when we successfully match against a message.
This signals to Yew that it needs to call the `view` method again to re-draw our screen.
Returning false here (which `match` does for us) will signal Yew that there is no need to re-draw our application.

### 4.4. Dynamically creating many buttons

Lastly, let's add a bit of complexity to our `view` method to dynamically populate the buttons, based on the strings inside `App.button_texts` .

This will demonstrate how to programmatically add elements to our HTML website.

Modify the `view` function as follows:

```rust
    fn view(&self, ctx: &Context<Self>) -> Html {
        let hello_style = css!("
           background-color: coral;
        ");

        let button_style = css!("
            background-color: LightGreen;
        ");

        let button_list = (0..self.button_texts.len())
            .map(|id| {
                html! {
                <button class={ button_style.clone() } onclick={
                ctx
                    .link()
                    .callback(move |_| AppMessage::ButtonPressed(id))
                }>
                    { &self.button_texts[id].to_string() }
                </button>
                }
            })
            .collect::<Html>();

        html!{
            <div>
                <p class={hello_style}>
                    {self.displayed_text.clone()}
                </p>
                { button_list }
            </div>
        }
    }
```

Here we replaced the old button handling logic with the `button_list` variable, that holds a computed HTML section.

We iterate through our `App.button_texts` array, calling the `html!` every time to create a button with a set style, including the in-line closure to generate the callback message.
We also need to call `collect` on it to squash the array into a single HTML definition so we can replace the old button definition within our return HTML document.

## 5. Conclusion

While in no way exhaustive, we saw one way how to make a simple web application with Rust.

From here on out it's just a case of changing the `view` method to make our application prettier and render more things, changing the `update` method to run application code on event messages and adding more of them in the `AppMessage` struct.

We could refactor our code to separate out the logic from `update` or even `view` , but that is left as an excersise to the reader.

**One final note here** is that the `App` struct we have made here is just one, monolithic application.
You could implement more structs as components and have them more self-contained, reducing the amount of re-rendering your browser needs to do to only those elements.
This is beyond the scope of this guide however.

I hope that it was coherent and helpful! For those who didn't follow along, here is our finished `src/main.rs` file in its entierty:

```rust  
use stylist::css;
use yew::prelude::*;

enum AppMessage{
    ButtonPressed(usize),
}

struct App {
    displayed_text: String,
    button_texts: Vec<String>,
}

impl Default for App {
    fn default() -> Self {
        App {
            displayed_text: "Press any button to see a change!".to_string(),
            button_texts: vec!(
                String::from("I like change"),
                String::from("I hate change"),
            ),
        }
    }
}

impl Component for App {
    type Message = AppMessage;
    type Properties = ();

    fn create(ctx: &Context<Self>) -> Self {
        Self::default()
    }

    fn update(&mut self, ctx: &Context<Self>, msg: Self::Message) -> bool {
        match msg {
            AppMessage::ButtonPressed(id) => {
                self.displayed_text = format!(
                    "You pressed the '{}' button with id '{}', press another!",
                    self.button_texts[id],
                    id,
                );
                true
            }
        }
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        let hello_style = css!("
           background-color: coral;
        ");

        let button_style = css!("
            background-color: LightGreen;
        ");

        let button_list = (0..self.button_texts.len())
            .map(|id| {
                html! {
                <button class={ button_style.clone() } onclick={
                ctx
                    .link()
                    .callback(move |_| AppMessage::ButtonPressed(id))
                }>
                    { &self.button_texts[id].to_string() }
                </button>
                }
            })
            .collect::<Html>();

        html!{
            <div>
                <p class={hello_style}>
                    {self.displayed_text.clone()}
                </p>
                {button_list}
            </div>
        }
    }
}
```