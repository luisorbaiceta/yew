---
title: "Portales"
description: "Renderizando en nodos fuera del árbol"
---

## ¿Cómo pensar en los portales?

Los portales proporcionan una forma de primera clase para representar nodos hijo en un nodo DOM que existe fuera de la jerarquía DOM de un componente padre.
`yew::create_portal(child, host)` regresa un valor `Html` que representa un `child` no jerarquizado bajo su componente padre,
pero como un hijo del elemento `host`.

## Uso

Los usos típicos de los portales pueden incluir diálogos modal y tarjetas flotantes, así como aplicaciones más técnicas como controlar los contenidos del [`shadowRoot`](https://developer.mozilla.org/en-US/docs/Web/API/Element/shadowRoot) de un elemento, agregando las hojas de estilos al `<head>` del documento circundante y recolectando los elementos referenciados dentro de un elemento `<defs>` central de un `<svg>`.

Nota que `yew::create_portal` es un bloque de construcción de bajo nivel, en el cual otros componentes deberían ser construidos que proporcione la interfaz para tu caso de uso específico. Como ejemplo, aquí hay un modal de diálogo simple que representa su `children` en un elemento fuera del control de `yew`, identificado por el `id="modal_host"`.

```rust
use yew::{html, create_portal, function_component, Children, Properties};

#[derive(Properties, PartialEq)]
pub struct ModalProps {
    #[prop_or_default]
    pub children: Children,
}

#[function_component(Modal)]
fn modal(props: &ModalProps) -> Html {
    let modal_host = gloo_utils::document()
        .get_element_by_id("modal_host")
        .expect("a #modal_host element");

    create_portal(
        html!{ {for props.children.iter()} },
        modal_host.into(),
    )
}
```

## Lectura adicional

- [Ejemplo de portales](https://github.com/yewstack/yew/tree/master/examples/portals)
