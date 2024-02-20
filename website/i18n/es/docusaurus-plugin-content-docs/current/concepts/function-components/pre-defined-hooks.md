---
title: "Hooks predefinidos"
description: "Los hooks predefinidos que vienen con Yew"
---

## `use_state`

`use_state` se usa para manejar el estado en un componente de función.
Este devuelve un objeto `UseStateHandle` el cual se `desreferencia` al valor actual
y provee un método `set`para actualizar el valor.

El hook toma una función como entrada la cual determina el estado inicial.
Este valor permanece actualizado en las rederizaciones subsecuentes.

La función setter se garantiza sea la misma a través de todo el ciclo de vida
del componente. Puedes fácilmente omitir el `UseStateHandle` de los
dependientes de `use_effect_with_deps` si sólo pretendes asignar
los valores desde el hook.

Este hook siempre lanzará una nueva renderización al recibir un nuevo estado. Utiliza [`use_state_eq`](#use_state_eq) si quieres que el componente sólo vuelva a renderizar cuando cambie el estado.

### Ejemplo

```rust
use yew::{Callback, function_component, html, use_state};

#[function_component(UseState)]
fn state() -> Html {
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

:::caution

El valor retenido en el manejo reflejará el valor al momento que el manejo
es retornado por `use_state`. Es posible que el manejo
no desreferencie a un valor actualizado si lo estás manipulando en un
hook `use_effect_with_deps`. Puedes registrar el estado para los
dependientes y así el hook puede estar actualizado cuando el valor cambie.

:::

## `use_state_eq`

Este hook tiene el mismo efecto que `use_state` pero sólo lanzará una
actualización de renderización cuando el setter reciba un valor donde `prev_state != next_state`.

Este hook requiere que el objeto de estado implemente `PartialEq`.

## `use_ref`

`use_ref` se usa para obtener una referencia inmutable  a un valor.
Su estado persiste a través de las renderizaciones.

`use_ref` puede ser útil para mantener las cosas en el alcance para el tiempo de vida del componente, siempre y cuando 
no almacenes un clon del `Rc` resultante en cualquier lugar que sobreviva al componente.

Si necesitas una referencia mutable, considera usar [`use_mut_ref`](#use_mut_ref).
Si necesitas que el componente se vuelva a renderizar en el cambio de estado, considera usar [`use_state`](#use_state).

```rust
// EventBus is an implementation of yew_agent::Agent
use website_test::agents::EventBus;
use yew::{function_component, html, use_ref, use_state, Callback};
use yew_agent::Bridged;

#[function_component(UseRef)]
fn ref_hook() -> Html { 
    let greeting = use_state(|| "No one has greeted me yet!".to_owned());

    {
        let greeting = greeting.clone();
        use_ref(|| EventBus::bridge(Callback::from(move |msg| {
            greeting.set(msg);
        })));
    }

    html! {
        <div>
            <span>{ (*greeting).clone() }</span>
        </div>
    }
}
```

## `use_mut_ref`

`use_mut_ref` se usa para obtener una referencia mutable a un valor.
Su estado persiste a través de las renderizaciones.

Es importante destacar que no obtienes notificación de los cambios de estado.
Si necesitas que tu componente sea renderizado nuevamente al cambiar de estado, considera usar [`use_state`](#use_state).

### Ejemplo

```rust
use web_sys::HtmlInputElement;
use yew::{
    events::Event,
    function_component, html, use_mut_ref, use_state,
    Callback, TargetCast,
};

#[function_component(UseMutRef)]
fn mut_ref_hook() -> Html {
    let message = use_state(|| "".to_string());
    let message_count = use_mut_ref(|| 0);

    let onclick = Callback::from(move |_| {
        let window = gloo_utils::window();

        if *message_count.borrow_mut() > 3 {
            window.alert_with_message("Message limit reached").unwrap();
        } else {
            *message_count.borrow_mut() += 1;
            window.alert_with_message("Message sent").unwrap();
        }
    });

    let onchange = {
        let message = message.clone();
        Callback::from(move |e: Event| {
            let input: HtmlInputElement = e.target_unchecked_into();
            message.set(input.value());
        })
    };

    html! {
        <div>
            <input {onchange} value={(*message).clone()} />
            <button {onclick}>{ "Send" }</button>
        </div>
    }
}
```

## `use_node_ref`

`use_node_ref` se usa para obtener un `NodeRef` que persiste a través de las renderizaciones.

Cuando renderizas elementos condicionalmente puedes usar `NodeRef` en conjunto con `use_effect_with_deps`
para realizar acciones cada vez que un elemento es renderizado y justo antes que sea removido del DOM.

### Ejemplo

```rust
use web_sys::HtmlInputElement;
use yew::{
    function_component, functional::*, html,
    NodeRef
};

#[function_component(UseRef)]
pub fn ref_hook() -> Html {
    let input_ref = use_node_ref();
    let value = use_state(|| 25f64);

    let onclick = {
        let input_ref = input_ref.clone();
        let value = value.clone();
        move |_| {
            if let Some(input) = input_ref.cast::<HtmlInputElement>() {
                value.set(*value + input.value_as_number());
            }
        }
    };

    html! {
        <div>
            <input ref={input_ref} type="number" />
            <button {onclick}>{ format!("Add input to {}", *value) }</button>
        </div>
    }
}
```

## `use_reducer`

`use_reducer` es una alternativa a [`use_state`](#use_state). Se usa para manejar el estado del componente y se utiliza cuando se necesitan realizar acciones complejas en dicho estado.

Acepta una función de estado inicial y retorna un `UseReducerHandle` que desreferencia al estado,
y una función de envío.
La función de envío toma un argumento del tipo `Action`. Al llamarla, la acción y el valor actual
son pasados a la función reducer la cual calcula un nuevo estado el cual es retornado,
y el componente vuelve a ser renderizado.

Se garantiza que la función de envío sea la misma a través del ciclo de vida
del componente. Puedes omitir `UseReducerHandle` de los dependientes 
de `use_effect_with_deps` si sólo tienes la intención de enviar 
valores desde los hooks.

El objeto de estado retornado por la función de estado inicial es requerido
para implementar la característica `Reducible` la cual provee un tipo de `Action` y
una función reducer.

Este hook siempre disparará una nueva renderización al recibir una acción. Ve
[`use_reducer_eq`](#use_reducer_eq) si quieres que el componente únicamente
vuelva a renderizar cuando el estado cambie.

### Ejemplo

```rust
use yew::prelude::*;
use std::rc::Rc;

/// reducer's Action
enum CounterAction {
    Double,
    Square,
}

/// reducer's State
struct CounterState {
    counter: i32,
}

impl Default for CounterState {
    fn default() -> Self {
        Self { counter: 1 }
    }
}

impl Reducible for CounterState {
    /// Reducer Action Type
    type Action = CounterAction;

    /// Reducer Function
    fn reduce(self: Rc<Self>, action: Self::Action) -> Rc<Self> {
        let next_ctr = match action {
            CounterAction::Double => self.counter * 2,
            CounterAction::Square => self.counter.pow(2)
        };

        Self { counter: next_ctr }.into()
    }
}

#[function_component(UseReducer)]
fn reducer() -> Html {
    // The use_reducer hook takes an initialization function which will be called only once.
    let counter = use_reducer(CounterState::default);

   let double_onclick = {
        let counter = counter.clone();
        Callback::from(move |_| counter.dispatch(CounterAction::Double))
    };
    let square_onclick = {
        let counter = counter.clone();
        Callback::from(move |_| counter.dispatch(CounterAction::Square))
    };

    html! {
        <>
            <div id="result">{ counter.counter }</div>

            <button onclick={double_onclick}>{ "Double" }</button>
            <button onclick={square_onclick}>{ "Square" }</button>
        </>
    }
}
```

:::caution

El valor retenido en el manejo reflejará el valor al momento que
el manejo sea retornado por `use_reducer`. Es posible que el manejo no
desreferencie a un valor actual si lo estás moviendo en
un hook `use_effect_with_deps`. Puedes registrar el
estado de los dependientes así el hook puede actualizarse cuando el valor cambie.

:::

## `use_reducer_eq`

This hook has the same effect as `use_reducer` but will only trigger a
re-render when the reducer function produces a value that `prev_state != next_state`.

This hook requires the state object to implement `PartialEq` in addition
to the `Reducible` trait required by `use_reducer`.

## `use_effect`

`use_effect` se usa para engancharse al ciclo de vida del componente.
Similar a `rendered` desde la característica del `Componente`,
`use_effect` toma una función que es llamada después que finaliza la renderización.

La función de entrada tiene que retornar un closure, el drestructor, el cual es llamado cuando el componente es destruido.
El destructor se puede utilizar para limpiar los efectos introducidos y puede apropiarse de los valores para retrasar
eliminarlos hasta que el componente es destruido.

### Ejemplo

```rust
use yew::{Callback, function_component, html, use_effect, use_state};

#[function_component(UseEffect)]
fn effect() -> Html {
    let counter = use_state(|| 0);

    {
        let counter = counter.clone();
        use_effect(move || {
            // Make a call to DOM API after component is rendered
            gloo_utils::document().set_title(&format!("You clicked {} times", *counter));

            // Perform the cleanup
            || gloo_utils::document().set_title("You clicked 0 times")
        });
    }
    let onclick = {
        let counter = counter.clone();
        Callback::from(move |_| counter.set(*counter + 1))
    };

    html! {
        <button {onclick}>{ format!("Increment to {}", *counter) }</button>
    }
}
```

### `use_effect_with_deps`

Algunas veces, se necesita definir manualmente las dependencias para [`use_effect`](#use_effect). En estos casos, usamos `use_effect_with_deps`.

```rust ,no_run
use yew::use_effect_with_deps;

use_effect_with_deps(
    move |_| {
        // ...
        || ()
    },
    (), // dependents
);
```

**Note**: `dependents` deben implementar `PartialEq`.

## `use_context`

`use_context` se usa para consimir [contexts](../contexts.md) en componentes de función.

### Ejemplo

```rust
use yew::{ContextProvider, function_component, html, use_context, use_state};


/// App theme
#[derive(Clone, Debug, PartialEq)]
struct Theme {
    foreground: String,
    background: String,
}

/// Main component
#[function_component(App)]
pub fn app() -> Html {
    let ctx = use_state(|| Theme {
        foreground: "#000000".to_owned(),
        background: "#eeeeee".to_owned(),
    });

    html! {
        // `ctx` is type `Rc<UseStateHandle<Theme>>` while we need `Theme`
        // so we deref it.
        // It derefs to `&Theme`, hence the clone
        <ContextProvider<Theme> context={(*ctx).clone()}>
            // Every child here and their children will have access to this context.
            <Toolbar />
        </ContextProvider<Theme>>
    }
}

/// The toolbar.
/// This component has access to the context
#[function_component(Toolbar)]
pub fn toolbar() -> Html {
    html! {
        <div>
            <ThemedButton />
        </div>
    }
}

/// Button placed in `Toolbar`.
/// As this component is a child of `ThemeContextProvider` in the component tree, it also has access to the context.
#[function_component(ThemedButton)]
pub fn themed_button() -> Html {
    let theme = use_context::<Theme>().expect("no ctx found");

    html! {
        <button style={format!("background: {}; color: {};", theme.background, theme.foreground)}>
            { "Click me!" }
        </button>
    }
}
```
