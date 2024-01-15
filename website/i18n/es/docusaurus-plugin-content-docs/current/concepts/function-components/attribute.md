---
title: "#[function_component]"
description: "El atributo #[function_component]"
---

`#[function_component(_)]` convierte una función normal de Rust en un componente de función.
Las funciones con el atributo tienen que retornar `Html` y pueden tomar un sólo parámetro para el tipo de props que el componente debe aceptar.
El tipo de parámetro necesita ser una referencia a un tipo `Properties` (ej. `props: &MyProps`).
Si la función no tiene ningún parámetro el componente resultante no acepta ningún props.

El atributo no reemplaza tú función original con un componente. Necesitas proveer un nombre como entrada al atributo el cual será el identificador del componente.

Asumiendo que tienes una función llamada `chat_container` y que agregas el atributo `#[function_component(ChatContainer)]`, puedes usar el componente así:

```rust
use yew::{function_component, html, Html};

#[function_component(ChatContainer)]
pub fn chat_container() -> Html {
    html! {
        // chat container impl
    }
}

html! {
    <ChatContainer />
};
```

## Ejemplo

<!--DOCUSAURUS_CODE_TABS-->
<!--With props-->

```rust
use yew::{function_component, html, Properties};

#[derive(Properties, PartialEq)]
pub struct RenderedAtProps {
    pub time: String,
}

#[function_component(RenderedAt)]
pub fn rendered_at(props: &RenderedAtProps) -> Html {
    html! {
        <p>
            <b>{ "Rendered at: " }</b>
            { &props.time }
        </p>
    }
}
```

<!--Without props-->

```rust
use yew::{function_component, html, use_state, Callback};

#[function_component(App)]
fn app() -> Html {
    let counter = use_state(|| 0);

    let onclick = {
        let counter = counter.clone();
        Callback::from(move |_| counter.set(*counter + 1))
    };

    html! {
        <div>
            <button {onclick}>{ "Increment value" }</button>
            <p>
                <b>{ "Current value: " }</b>
                { *counter }
            </p>
        </div>
    }
}
```

<!--END_DOCUSAURUS_CODE_TABS-->

## Componentes de funciones genéricas

El atributo `#[function_component(_)]` también funciona con funciones genéricas para crear componentes genéricos.

```rust
use std::fmt::Display;
use yew::{function_component, html, Properties};

#[derive(Properties, PartialEq)]
pub struct Props<T>
where
    T: PartialEq,
{
    data: T,
}

#[function_component(MyGenericComponent)]
pub fn my_generic_component<T>(props: &Props<T>) -> Html
where
    T: PartialEq + Display,
{
    html! {
        <p>
            { &props.data }
        </p>
    }
}

#[derive(PartialEq)]
struct Foo;

impl Display for Foo {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        f.write_str("foo")
    }
}

// used like this
html! {
    <MyGenericComponent<i32> data=123 />
};

// or
let foo = Foo;

html! {
    <MyGenericComponent<Foo> data={foo} />
};
```
