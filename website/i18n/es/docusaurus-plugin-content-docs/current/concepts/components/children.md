---
title: "Hijos"
---

## Uso general

_La mayor parte del tiempo,_ cuando permitimos que un componente tenga un hijo, no es de tú interés
qué tipo de hijo tiene el componente. En estos casos, el ejemplo de abajo
bastará.

```rust
use yew::{html, Children, Component, Context, Html, Properties};

#[derive(Properties, PartialEq)]
pub struct ListProps {
    #[prop_or_default]
    pub children: Children,
}

pub struct List;

impl Component for List {
    type Message = ();
    type Properties = ListProps;

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        html! {
            <div class="list">
                { for ctx.props().children.iter() }
            </div>
        }
    }
}
```

## Uso avanzado

### Hijos tipados

En casos donde quieres que un tipo de componente sea pasado como hijo a tu componente,
puedes usar `yew::html::ChildrenWithProps<T>`.

```rust
use yew::{html, ChildrenWithProps, Component, Context, Html, Properties};

pub struct Item;

impl Component for Item {
    type Message = ();
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, _ctx: &Context<Self>) -> Html {
        html! {
            { "item" }
        }
    }
}

#[derive(Properties, PartialEq)]
pub struct ListProps {
    #[prop_or_default]
    pub children: ChildrenWithProps<Item>,
}

pub struct List;

impl Component for List {
    type Message = ();
    type Properties = ListProps;

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        html! {
            <div class="list">
                { for ctx.props().children.iter() }
            </div>
        }
    }
}
```

### Hijos tipados Enum

Desde luego, algunas veces tienes la necesidad de restringir a los hijos a unos pocos
componentes. En estos casos, debes tener un poco más de práctica con Yew.

La caja [`derive_more`](https://github.com/JelteF/derive_more) se usa aquí
por mejor ergonomía. Si no quieres usarla, puedes implementar manualmente
`From` para cada variante.

```rust
use yew::{
    html, html::ChildrenRenderer, virtual_dom::VChild, Component, 
    Context, Html, Properties,
};

pub struct Primary;

impl Component for Primary {
    type Message = ();
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, _ctx: &Context<Self>) -> Html {
        html! {
            { "Primary" }
        }
    }
}

pub struct Secondary;

impl Component for Secondary {
    type Message = ();
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, _ctx: &Context<Self>) -> Html {
        html! {
            { "Secondary" }
        }
    }
}

#[derive(Clone, derive_more::From, PartialEq)]
pub enum Item {
    Primary(VChild<Primary>),
    Secondary(VChild<Secondary>),
}

// Now, we implement `Into<Html>` so that yew knows how to render `Item`.
#[allow(clippy::from_over_into)]
impl Into<Html> for Item {
    fn into(self) -> Html {
        match self {
            Self::Primary(child) => child.into(),
            Self::Secondary(child) => child.into(),
        }
    }
}

#[derive(Properties, PartialEq)]
pub struct ListProps {
    #[prop_or_default]
    pub children: ChildrenRenderer<Item>,
}

pub struct List;

impl Component for List {
    type Message = ();
    type Properties = ListProps;

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        html! {
            <div class="list">
                { for ctx.props().children.iter() }
            </div>
        }
    }
}
```

### Hijo tipado Optional

También puedes tener un solo componente hijo opcional de un tipo específico:

```rust
use yew::{
    html, html_nested, virtual_dom::VChild, Component, 
    Context, Html, Properties
};

pub struct PageSideBar;

impl Component for PageSideBar {
    type Message = ();
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, _ctx: &Context<Self>) -> Html {
        html! {
            { "sidebar" }
        }
    }
}

#[derive(Properties, PartialEq)]
pub struct PageProps {
    #[prop_or_default]
    pub sidebar: Option<VChild<PageSideBar>>,
}

struct Page;

impl Component for Page {
    type Message = ();
    type Properties = PageProps;

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        html! {
            <div class="page">
                { ctx.props().sidebar.clone().map(Html::from).unwrap_or_default() }
                // ... page content
            </div>
        }
    }
}

// The page component can be called either with the sidebar or without: 

pub fn render_page(with_sidebar: bool) -> Html {
    if with_sidebar {
        // Page with sidebar
        html! {
            <Page sidebar={{html_nested! {
                <PageSideBar />
            }}} />
        }
    } else {
        // Page without sidebar
        html! {
            <Page />
        }
    }
}
```
