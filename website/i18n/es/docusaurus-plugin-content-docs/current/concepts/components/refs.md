---
title: "Referencias"
description: "Acceso DOM fuera de banda"
---

La palabra reservada `ref` puede ser usada dentro de cualquier elemento HTML o componente para obtener el `Elemento` DOM al
que está adjunto el ítem. Esto puede ser usado para realizar cambios al DOM fuera del método del ciclo de vida de la `vista`.

Esto es útil para conseguir elementos del lienzo, o desplazarse a distintas secciones de la página. 
Por ejemplo, usando un `NodeRef` en el método `rendered` de un componente, te permite realizar llamadas de dibujo a 
un elemento de lienzo después que ha sido renderizado por la `vista`.

La sintaxis es:

```rust
use web_sys::Element;
use yew::{html, Component, Context, Html, NodeRef};

struct Comp {
    node_ref: NodeRef,
}

impl Component for Comp {
    type Message = ();
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self {
            // highlight-next-line
            node_ref: NodeRef::default(),
        }
    }

    fn view(&self, _ctx: &Context<Self>) -> Html {
        html! {
            // highlight-next-line
            <div ref={self.node_ref.clone()}></div>
        }
    }

    fn rendered(&mut self, _ctx: &Context<Self>, _first_render: bool) {
        // highlight-start
        let has_attributes = self.node_ref
            .cast::<Element>()
            .unwrap()
            .has_attributes();
        // highlight-end
    }
}
```

## Ejemplos relevantes

- [Node Refs](https://github.com/yewstack/yew/tree/master/examples/node_refs)
