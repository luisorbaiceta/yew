---
title: "Hooks personalizados"
description: "Define tus propis Hooks"
---

## Definiendo Hooks personalizados

La lógica con estado del componente puede ser extraída en una función usable al crear Hooks personalizados.

Considera que tenemos un componente que se suscribe a un agente y muestra los mensajes enviados a este.

```rust
use yew::{function_component, html, use_effect, use_state, Callback};
use yew_agent::Bridged;
// EventBus is an implementation yew_agent::Agent
use website_test::agents::EventBus;


#[function_component(ShowMessages)]
pub fn show_messages() -> Html {
    let state = use_state(Vec::new);

    {
        let state = state.clone();
        use_effect(move || {
            let producer = EventBus::bridge(Callback::from(move |msg| {
                let mut messages = (*state).clone();
                messages.push(msg);
                state.set(messages)
            }));

            || drop(producer)
        });
    }

    let output = state.iter().map(|it| html! { <p>{ it }</p> });
    html! { <div>{ for output }</div> }
}
```

Existe un problema con este código: la lógica no puede ser reutilizada por otro componente.
Si construimos otro componente que realice seguimiento de los mensajes, en lugar de copiar el código modemos mover la lógica en un hook personalizado.

Comenzaremos creando una nueva función llamada `use_subscribe`.
El prefijo `use_`por convención denota que una función es un hook.
Esta función no tomará argumentos y retornará `Rc<RefCell<Vec<String>>>`.
```rust
use std::{cell::RefCell, rc::Rc};

fn use_subscribe() -> Rc<RefCell<Vec<String>>> {
    todo!()
}
```

Este es un hook simple que puede ser creado al combinar otros hooks. Para este ejemplo, tendremos dos hooks predefinidos.
Usaremos el hook `use_state`para almacenar el `Vec` para mensajes, así se persisten entre la renderización de componentes.
También usaremos `use_effect` para suscribirnos al `EventBus` `Agent` así la suscripción puede ser asociada al ciclo de vida del componente.

```rust
use std::collections::HashSet;
use yew::{use_effect, use_state, Callback};
use yew_agent::Bridged;
// EventBus is an implementation yew_agent::Agent
use website_test::agents::EventBus;

fn use_subscribe() -> Vec<String> {
    let state = use_state(Vec::new);

    let effect_state = state.clone();

    use_effect(move || {
        let producer = EventBus::bridge(Callback::from(move |msg| {
            let mut messages = (*effect_state).clone();
            messages.push(msg);
            effect_state.set(messages)
        }));
        || drop(producer)
    });

    (*state).clone()
}
```

Aunque este enfoque funciona en la mayoría de los casos, no puede ser usado para escribir hooks primitivos como los hooks predefinidos que ya hemos visto.

### Escribiendo hooks primitivos

La función `use_hook` se usa para escribir dichos hooks. Mira los docs en [docs.rs](https://docs.rs/yew/0.18.0/yew-functional/use_hook.html) para la documentación
y el directorio `hooks` para ver las implementaciones de los hooks predefinidos.
