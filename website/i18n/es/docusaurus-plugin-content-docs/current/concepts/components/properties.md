---
title: "Propiedades"
description: "Comunicación padre a hijo"
---

Las propiedades habilitan a los componentes padre e hijo el comunicarse entre sí.
Cada componente tiene un tipo de propiedades asociadas la cual describe qué es pasado desde el padre.
En teoría esto puede ser de cualquier tipo que implemente la característica `Properties`, pero en la práctica no hay
razón para que sea cualquier tipo sino una estructura donde cada campo representa una propiedad.

## Derivar macro

En lugar de implementar la característica `Properties` tú mismo, deberías usar `#[derive(Properties)]` para
generar automáticamente la implementación.
Los tipos para los cuales derivas `Properties` también deben implementar a `PartialEq`.

### Atributos de campo

Cuando derivas `Properties`, todos los campos son requeridos por defecto.
Los siguientes atributos te permiten dar a tus props valores iniciales los cuales serán usados a menos que le sean asignados otro valor.

:::tip
Los atributos no son visibles en la documentación generada por Rustdoc.
Los docstrings de tus propiedades deberían mencionar si una prop es opcional y si tiene un valor especial por defecto.
:::

#### `#[prop_or_default]`

Inicializa el valor del prop con el valor por defecto del tipo de campo usando la característica `Default`.

#### `#[prop_or(value)]`

Usa `value` para inicializar el valor del prop. El `value` puede ser cualquier expresión que regrese el tipo de campo.
Por ejemplo, para asignar por defecto un prop booleano a `true`, usa el atributo `#[prop_or(true)]`.

#### `#[prop_or_else(function)]`

Llama a `function` para inicializar el valor del prop. `function` debe tener la firma `FnMut() -> T` donde `T` es el tipo de campo.

## `PartialEq`

Los `Properties` requieren `PartialEq` sea implementado. Esto es para que puedan ser comparados por Yew al llamar
el método `changed` sólo cuando estos cambien.

## Sobrecarga de memoria/velocidad del uso de Properties

Internamente, las propiedades son contadas por referencia. Esto significa que sólo es pasado un apuntador por el árbol de componentes para los props.
Esto nos evita el costo de tener que clonar los props completos, lo cual puede ser costoso.

## Ejemplo

```rust
use std::rc::Rc;
use yew::Properties;

#[derive(Clone, PartialEq)]
pub enum LinkColor {
    Blue,
    Red,
    Green,
    Black,
    Purple,
}

fn create_default_link_color() -> LinkColor {
    LinkColor::Blue
}

#[derive(Properties, PartialEq)]
pub struct LinkProps {
    /// The link must have a target.
    href: String,
    text: String,
    /// Color of the link. Defaults to `Blue`.
    #[prop_or_else(create_default_link_color)]
    color: LinkColor,
    /// The view function will not specify a size if this is None.
    #[prop_or_default]
    size: Option<u32>,
    /// When the view function doesn't specify active, it defaults to true.
    #[prop_or(true)]
    active: bool,
}
```

## Macro de props

El macro `yew::props!` te permite construir propiedades de la misma forma que lo hace el macro `html!`.

El macro usa la misma sintaxis que una expresión de estructura excepto que no puedes usar atributos o una expresión base (`Foo { ..base }`).
La ruta de tipo puede tanto apuntar a las props directamente (`path::to::Props`) o a las propiedades asociadas de un componente (`MyComp::Properties`).

```rust
use std::rc::Rc;
use yew::{props, Properties};

#[derive(Clone, PartialEq)]
pub enum LinkColor {
    Blue,
    Red,
    Green,
    Black,
    Purple,
}

fn create_default_link_color() -> LinkColor {
    LinkColor::Blue
}

#[derive(Properties, PartialEq)]
pub struct LinkProps {
    /// The link must have a target.
    href: String,
    text: Rc<String>,
    /// Color of the link. Defaults to `Blue`.
    #[prop_or_else(create_default_link_color)]
    color: LinkColor,
    /// The view function will not specify a size if this is None.
    #[prop_or_default]
    size: Option<u32>,
    /// When the view function doesn't specify active, it defaults to true.
    #[prop_or(true)]
    active: bool,
}

impl LinkProps {
    pub fn new_link_with_size(href: String, text: String, size: u32) -> Self {
        // highlight-start
        props! {LinkProps {
            href,
            text: Rc::from(text),
            size,
        }}
        // highlight-end
    }
}
```
