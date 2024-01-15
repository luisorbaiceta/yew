---
title: "Callbacks"
description: "ComponentLink y Callbacks"
---

## API `Scope<_>` del componente

El componente "`Scope`" es el mecanismo por el cual los componetes son capaces de crear callbacks y actualizarse a sí mismos
usando mensajes. Obtenemos una referencia a este al llamar a `link()` en el objeto de contexto pasado al componente.

### `send_message`

Envía un mensaje al componente.
Los mensajes son manejados por el método `update` el cual determina si el componente debe volver a renderizarse.

### `send_message_batch`

Envía múltiples mensajes al componente al mismo tiempo.
Esto es similar a `send_message` pero si cualquiera de los mensajes ocasiona que el método `update` regrese `true`,
el componente volverá a renderizarse después que todos los mensajes en el lote han sido procesados.

Si el vector proporcionado está vacío, esta función no hace nada.

### `callback`

Crea un callback que enviará un mensaje al componente cuando sea ejecutado.
Internamente, llamará a `send_message` con el mensaje retornado por el closure proporcionado.

Existe un método diferente llamado `callback_once` el cual acepta un `FnOnce` en lugar de `Fn`.
Deberías usar este con cuidado, ya que el callback resultante entrará en pánico si se ejecuta más de una vez.

```rust
use yew::{html, Component, Context, Html};

enum Msg {
    Text(String),
}

struct Comp;

impl Component for Comp {

    type Message = Msg;
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        // Create a callback that accepts some text and sends it 
        // to the component as the `Msg::Text` message variant.
        // highlight-next-line
        let cb = ctx.link().callback(|text: String| Msg::Text(text));
            
        // The previous line is needlessly verbose to make it clearer.
        // It can be simplified it to this:
        // highlight-next-line
        let cb = ctx.link().callback(Msg::Text);
            
        // Will send `Msg::Text("Hello World!")` to the component.
        // highlight-next-line
        cb.emit("Hello World!".to_owned());

        html! {
            // html here
        }
    }
}
```

### `batch_callback`

Crea un callback que enviará un lote de mensajes al componente cuando este se ejecute.
La diferencia con `callback` es que el closure proporcionado a este método no tiene un mensaje de retorno.
En su lugar, el closure puede regresar `Vec<Msg>` o `Option<Msg>` donde `Msg` es el tipo de mensaje del componente.

`Vec<Msg>` es tratado como un lote de mensajes y usa `send_message_batch` internamente.

`Option<Msg>` llama a `send_message` si es `Some`. Si el valor es `None`, no pasa nada.
Esto puede usarse en casos donde, dependiendo la situación, no se requiere una actualización.

Esto se logra usando la característica `SendAsMessage` la cual sólo es implementada para estos tipos.
Puedes implementar `SendAsMessage` para tus propios tipos lo que te permite usarlos en `batch_callback`.

Como `callback`, este método también tiene una contraparte `FnOnce`, `batch_callback_once`.
Aplican las mismas restricciones como en `callback_once`.

## Callbacks

_\(Esto podría necesitar su propia página corta.\)_

Los callback son usados para comunicarse con los servicios, agentes y componentes padre dentro de Yew.
Internamente, su tipo es sólo `Fn` envuelto en `Rc` para permitirles ser clonados.

Tienen una función `emit` que toma su tipo `<IN>` como un argumento y lo convierte a un mensaje esperado por su destino. Si se proporciona un callback de un padre en los props a un componente hijo, el hijo puede llamar a `emit` en el callback en su hook `update` de su ciclo de vida para enviar un mensaje de vuelta a su padre. Los Closures o Functions proporcionados como props dentro de la macro `html!` son convertidos automáticamente a Callbacks.

Un uso simple de un callback podría verse como esto:

```rust
use yew::{html, Component, Context, Html};

enum Msg {
    Clicked,
}

struct Comp;

impl Component for Comp {

    type Message = Msg;
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        // highlight-next-line
        let onclick = ctx.link().callback(|_| Msg::Clicked);
        html! {
            // highlight-next-line
            <button {onclick}>{ "Click" }</button>
        }
    }
}
```

La función pasada a `callback` siempre debe tomar un parámetro, el manejador `onclick` requiere una función la cual toma un parámetro del tipo `MouseEvent`. El manejador puede decidir qué tipo de mensaje debe enviar al componente. Este mensaje es programado para el pŕoximo ciclo de actualización incondicionalmente.

Si necesitas un callback que podría no necesitar causar una actualización, usa `batch_callback`.

```rust
use yew::{events::KeyboardEvent, html, Component, Context, Html};

enum Msg {
    Submit,
}

struct Comp;

impl Component for Comp {

    type Message = Msg;
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        // highlight-start
        let onkeypress = ctx.link().batch_callback(|event: KeyboardEvent| {
            if event.key() == "Enter" {
                Some(Msg::Submit)
            } else {
                None
            }
        });
        
        html! {
            <input type="text" {onkeypress} />
        }
        // highlight-end
    }
}
```

## Ejemplos relevantes

- [Contador](https://github.com/yewstack/yew/tree/master/examples/counter)
- [Temporizador](https://github.com/yewstack/yew/tree/master/examples/timer)
