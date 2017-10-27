# Creando mejores componentes

![Abejas trabajando en un panal](./images/destacada.jpg)

<small>Foto por Meggyn Pomerleau en [Unsplash](https://unsplash.com/photos/hAYy2mFLjS8)</small>

Crear componentes visuales reutilizables parece tarea sencilla pero la realidad es que construirlos de manera que sean previsibles, extensibles y con una API intuitiva es más difícil de lo que parece.

---

## Consideraciones previas

Vamos a hablar de componentes altamente reutilizables. También podemos llamarlos primitivos, componentes que podrían ser parte de una librería, sistema de diseño o de un gran proyecto. Componentes que van a utilizarse en innumerables lugares y a los cuales queremos dotar de todo lo necesario para que los consumidores puedan usarlos sin necesidad de requerir cambios en los mismos continuamente.

Aunque sepamos el contexto en el que un componente se va a usar a día de hoy, no sabemos en qué contexto se usará dentro de un tiempo. Por ello buscaremos un nivel de flexibilidad que evite modificaciones innecesarias de nuestros componentes en el futuro. En el caso de una librería de componentes evitar estas modificaciones es crucial para evitar la publicación de nuevas versiones de nuestros paquetes muy a menudo y que nuestros consumidores se cansen.

Estos componentes primitivos son la base para construir otros componentes más complejos y son los que permiten a nuestros consumidores crear sus propios componentes y patrones sin necesidad de modificar la librería.

> Los ejemplos de código están escritos con React pero los conceptos son extrapolables a otros frameworks como Vue o Svelte.

Antes de crear componentes visuales reutilizables primero tenemos que definir nuestros objetivos. Buscamos:

- API intuitiva y estable
- Comportamiento previsible
- Extensibilidad
- Encapsulación
- Accesibilidad

## API intuitiva y estable

![Elefante](./images/api-intuitiva.jpg)

<small>Foto por Nam Anh en [Unsplash](https://unsplash.com/photos/QJbyG6O0ick)</small>

### Nombrar props como atributos del DOM

Algunos componentes tienen una correlación directa con elementos del DOM. Por ejemplo, un componente `Button` y un elemento `<button>`. Otros componentes quizás no tengan una correlación directa pero sí lo suficientemente cercana, como por ejemplo `Avatar`. `Avatar` renderiza una imagen y esto tiene una correlación con la etiqueta `<img>` de HTML. Podemos hacer uso de esa correlación y ese conocimiento ya existente de HTML para crear parte de la API de nuestros componentes de manera que incluso nuestros consumidores pueden estar familizados con ella de antemano. Cuanto más "aburrida" sea nuestra API, mejor.

Imagina un componente `Button` al que queremos poder establecer su estado _disabled_. El elemento `<button>` del DOM ya tienen un atributo `disabled` por lo tanto usaremos ese nombre para nuestra prop.

**✅ Do**

```javascript
function Button({ disabled, children, ...rest }) {
  return <button className={cn('btn', { disabled })} disabled={disabled} {...rest} />;
}
```

> `cn` en los ejemplos de código se refiere a [`classnames`](https://github.com/JedWatson/classnames), un diminuto y genial paquete de npm que ayuda a unir nombres de clases de manera condicional. Lo verás en los ejemplos a lo largo del artículo.
>
> `...rest` nos sirve para pasar a nuestro componente todas las props que no hemos usado explícitamente. En un siguiente apartado explicamos su importancia.

En el caso de un componente `Avatar` al que le tenemos que pasar la url del avatar, podemos usar una prop `src`, como el atributo de las etiquetas `<img>`, en vez de usar un nombre como `url`.

**✅ Do**

```javascript
function Avatar({ src, ...rest }) {
  return <img className="avatar" src={src} {...rest} />;
}
```

En el caso de que prefiramos no adherirnos a nombres de atributos existentes en el DOM, lo importante es mantener una coherencia. Si por ejemplo para nuestro componente `Button` preferimos usar una prop `isDisabled` en vez de `disabled`, debemos mantener esa nomenclatura en todos nuestros componentes: `<Button isDisabled>`, `<Checkbox isDisabled>`, `<Select isDisabled>`, etc.

### Usa _children_

Llamamos _children_ al contenido que se incluye entre la "etiqueta" de apertura y cierre de un componente (dependiendo del framework que usemos puede también puede llamarse _slot_).

```javascript
<Heading>Yo soy el children</Heading>
```

Así es como funciona HTML: lo que se incluye dentro de una etiqueta es lo que renderiza esa etiqueta salvo elementos con cierre automáticos como `<img />`, `<input />`, `<br />`, etc. Debemos respetar este comportamiento siempre que sea posible para pasar contenido a nuestros componentes en vez de usar props específicas en cada componente. Además tendremos una misma API para pasar el contenido a muchos de nuestros componentes.

**🛑 Don't**

```javascript
function Heading({ text, ...rest }) {
  return (
    <h1 className="heading" {...rest}>
      {text}
    </h1>
  );
}

// <Heading text="¿Wot?" />
```

**✅ Do**

```javascript
function Heading({ children, ...rest }) {
  return (
    <h1 className="heading" {...rest}>
      {children}
    </h1>
  );
}

// <Heading>Mucho mejor, claro</Heading>
```

Además, `children` nos permite componer de manera muy intuitiva tal y cómo lo haríamos con HTML.

```javascript
<Heading>
  <IconHome /> Bienvenido a Octuweb
</Heading>
```

## Previsible

![Perezoso colgado de un árbol](./images/previsible.jpg)

<small>Foto por Javier Mazzeo en [Unsplash](https://unsplash.com/photos/GTXvpZ2eTdA)</small>

### Singel: *Single Element Pattern*

[Diego Haz](https://twitter.com/diegohaz) escribió el que para mi es un artículo altamente influyente: [Introducing the Single Element Pattern](https://www.freecodecamp.org/news/introducing-the-single-element-pattern-dfbd2c295c5d/).

> Muchos de los principios descritos en el artículo de Diego Haz están reflejados en este mismo artículo. Diego los aplica en su librería open source [Reakit](https://reakit.io). Se trata de una librería de componentes que solamente provee funcionamiento (agnóstica en cuanto a estilos) y 100% accesible. Un proyecto referente y una auténtica maravilla de la composición. [Algunos](https://twitter.com/diegohaz/status/1305450112890662914) de sus [hilos](https://twitter.com/diegohaz/status/1313449837183078400) en Twitter son joyas. Muy recomendable seguir su trabajo.

La primera regla que nombra el artículo es: "Renderiza sólo un elemento". ¿Qué quiere decir esto? Veamos un ejemplo. Vamos a crear un componente `Tabs`. Primero vamos a mostrar una API que no cumple con el patrón _Singel_. El componente recibe un array de objetos con las tabs a renderizar. El componente internamente se encarga de renderizar correctamente los títulos de las pestañas y su respectivo contenido. Parece una API muy cómoda a la hora de usar este componente., ¡tan sólo una prop!

**🛑 Don't**

```javascript
<Tabs
  items={[
    {
      title: 'One',
      content: <p>one!</p>,
    },
    {
      title: 'Two',
      content: <p>two!</p>,
    },
  ]}
/>
```

¿Cuál es el problema? Que no es extensible:

- Si necesitamos añadir un icono al título de la primera tab, tendríamos que cambiar la API de nuestro componente.
- Si queremos añadir un `className` a una de las tabs no podemos.
- Si necesitamos añadir un atributo `aria` tampoco podemos sin cambiar la API.

Vamos a ver cómo luciría la API de nuestro componente siguiendo el patrón _Singel_.

**✅ Do**

```javascript
<Tabs>
  <TabList>
    <Tab>One</Tab>
    <Tab>Two</Tab>
  </TabList>

  <TabPanels>
    <TabPanel>
      <p>one!</p>
    </TabPanel>
    <TabPanel>
      <p>two!</p>
    </TabPanel>
  </TabPanels>
</Tabs>
```

Exactamente así es como luce la API de `Tabs` [en Chakra UI](https://web.archive.org/web/20191224183234if_/https://chakra-ui.com/tabs). Quizás puedas pensar que de esta forma escribimos más código y exigimos más a nuestros consumidores. Sí, es cierto, pero damos libertad para extender el componente lo cual era uno de nuestros objetivos. Esto nos permitirá no estar modificando la API de nuestro componente continuamente. Queremos una API estable.

Eliminamos muchos problemas y futuros cambios. Y ahora:

- Si necesitamos añadir un icono al título de la primera tab, simplemente lo pasamos en el _children_.

```javascript
<Tab>
  <IconStar /> One
</Tab>
```

- Si queremos añadir un `className` a una de las tabs, simplemente la pasamos como prop.

```javascript
<Tab className="awesome">One</Tab>
```

- Si queremos añadir un atributo `aria` lo añadimos.

```javascript
<TabPanel aria-labelledby="title">
  <p>one!</p>
</TabPanel>
```

El principio _Singel_, como cualquier otro, está para romperlo. Todo depende del contexto. Recordemos que en nuestro caso estamos en un contexto de componentes visuales de bajo nivel altamente reutilizables los cuales van a ser la fundación de nuestra librería de componentes y con los cuales vamos a componer para crear componentes más complejos. Queremos estabilidad.

Otra librería de componentes para React que sigue el principio _Singel_ es [Reach UI](https://reach.tech). Es agnóstica en cuanto a estilos y es toda una referencia en cuanto a accesibilidad. [Sus principios](https://gist.github.com/ryanflorence/e5c794e6093d16a69fa88d2112a292f7), escritos por Ryan Florence, son una joya.

### _Spread_ props

Habrás visto que en los ejemplos de código se hace [spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) del resto de las props que no hemos usado en nuestros componentes. ¿Por qué? Porque no queremos estar definiendo cada prop que el consumidor de nuestro componente quiera que acabe como un atributo del DOM, es algo que va a pasar continuamente. Además cuando hacemos spread de props en un elemento como `span` esas props acaban en el DOM, es algo que nuestros consumidores van a esperar que pase.

Pongamos otra vez `Button` como ejemplo. Imagina que alguien quiere pasarle una prop `id`:

```javascript
function Button({ children, disabled, id }) {
  return (
    <button className={cn('btn', { disabled })} id={id}>
      {children}
    </button>
  );
}
```

Más tarde se necesita pasar un `title`:

```javascript
function Button({ children, disabled, id, title }) {
  return (
    <button className={cn('btn', { disabled })} id={id} title={title}>
      {children}
    </button>
  );
}
```

Esto ocurrirá por cada atributo del DOM que queramos que acepte nuestro componente `Button`: `type`, `aria-label`, `value`, etc. No queremos estar definiendo cada prop que existe como atributo en el DOM, por ello hacemos _spread_ del resto de las props que no usamos al definir el marcado de nuestro componente.

**✅ Do**

```javascript
function Button({ disabled, ...rest }) {
  return <button className={cn('btn', { disabled })} {...rest} />;
}
```

> En React, [`children`](https://reactjs.org/docs/jsx-in-depth.html#children-in-jsx) es una prop más por lo que en este ejemplo está incluido en el objeto `rest`. No es necesario pasarla explícitamente.

En el caso de que nuestro componente devuelva más de un elemento, deberemos elegir el elemento principal sobre el que haremos el _spread_ de props. Pero si es posible, es recomendable adherirse al patrón _Singel_.

### Cuidado con las falsas _boolean_ props

De nuevo ponemos `Button` como ejemplo. En un momento dado tenemos que añadir una variante _primary_ y pensamos que lo mejor es añadir una prop `primary` que acepta `true`/`false`.

```javascript
function Button({ primary, ...rest }) {
  return <button className={cn('btn', { primary })} {...rest} />;
}
```

Dos semanas después necesitas añadir una nueva variante: _secondary_ y añades una prop `secondary`. Después te das cuenta de que hace falta una variante _outline_ y añades una prop `outline`. Acabamos con algo así:

**🛑 Don't**

```javascript
function Button({ primary, secondary, outline, ...rest }) {
  return <button className={cn('btn', { primary, secondary, outline })} {...rest} />;
}
```

Nuestras tres variantes son excluyentes y sin embargo el código permite usarlas al mismo tiempo: `<Button primary secondary outline>`. _No bueno_. ¿Cuál tiene prioridad? Deberíamos dejar claro en la API de nuestro componente que las tres variantes son excluyentes. Podemos usar una única prop `variant` para ello (o cualquier otro nombre, lo importante es ser consistente con el nombre en todos nuestros componentes que acepten una prop que cumpla la misma finalidad).

**✅ Do**

```javascript
function Button({ variant, ...rest }) {
  return <button className={cn('btn', variant)} {...rest} />;
}
```

De esta forma nos aseguramos de que sólo podemos asignar una variante a nuestro componente y los consumidores no estarán confundidos sobre qué variante usar o si estas se pueden combinar.

En el caso de que nuestros componentes empiecen a crecer en variantes y posibles combinaciones de props es bueno pararse a pensar si de verdad queremos que nuestro componente valga para todos esos casos o si vale la pena crear versiones específicas de nuestro componente que cumplan un propósito y lo cumplan bien: `Button`, `ButtonOutline`, `ButtonIcon`, etc.

## Extensible

![Jirafa alargando su lengua para comer hojas](./images/extensible.jpg)

<small>Foto por Slawek K en [Unsplash](https://unsplash.com/photos/XW0_pAwcPkQ)</small>

### Componentes polimórficos

En ocasiones queremos que un componente se renderice como un elemento diferente dependiendo del uso que le vayamos a dar o del contexto en el que se encuentre. Pongamos como ejemplo un componente `<Text variant="body" />`. Recibe una prop `variant` que establece el estilo visual entre uno de los que hemos definido en nuestro imaginario sistema de diseño: body, heading, overline, etc. Pero, ¿con qué etiqueta HTML acaba en el DOM? ¿Será siempre un `<p>`? ¿Creamos un componente por cada etiqueta?

Por suerte tenemos el concepto de componentes polimórficos. Mediante la `as` prop (fue [popularizada por styled-components](https://web.archive.org/web/20200803084325/https://styled-components.com/docs/api#as-polymorphic-prop)) podemos decirle a nuestro componente qué etiqueta queremos que renderice. Le damos por defecto el valor de `p`, para que se renderice como un párrafo en caso de que no se le pase la prop `as`.

> Puedes usar un nombre diferente a `as` para la prop. Otras librerías de componentes usan otros nombres como `component` o `tag`, pero `as` es la más extendida.

**✅ Do**

```javascript
function Text({ variant, as = 'p', ...rest }) {
  const Component = as;
  return <Component className={`text-${variant}`} {...rest} />;
}

// Por defecto es un <p>
// <Text variant="body" />

// Heading como <h1>
// <Text variant="heading" as="h1" />

// Heading como <span>
// <Text variant="heading" as="span" />

// Heading como cualquier otro componente
// <Text variant="heading" as={CustomComponent} />
```

Esta técnica es muy útil en el caso de un componente `Button` ya que nos permite renderizarlo como `<button>`, `<a>` o un `<Link>` de react-router, por ejemplo.

> Si usas React + TypeScript quizás quieras agradecerle a [Kristóf Poduszló](https://twitter.com/kripod97) su maravillosa librería [react-polymorphic-box](https://github.com/kripod/react-polymorphic-box) para crear componentes polimórficos (casi) perfectamente tipados. Es algo muy difícil de conseguir.

### Combina _className_ y estilos inline

Un error bastante común es no pasar la prop `className` a nuestro componente o que al pasarla sobreescribimos el `className` interno de nuestro componente.

**🛑 Don't**

```javascript
function ListItem(props) {
  return <li className="listItem" {...props} />;
}
```

En el ejemplo anterior si le pasamos `className` a `ListItem`, sobreescribirá nuestro `className` interno de manera que el componente perderá sus estilos.

Cuando pasamos un `className`, debemos asegurarnos de que lo combinamos con el `className` interno de nuestro componente.

**✅ Do**

```javascript
function ListItem({ className, ...rest }) {
  return <li className={cn('listItem', className)} {...rest} />;
}
```

En caso de que estemos definiendo estilos inline para nuestro componente también debemos de combinarlos con los que el componente reciba por props. Este caso es mucho menos común, pero en ocasiones es necesario agregar estilos inline para estilos computados en tiempo de ejecución que dependen de alguna prop o estado interno.

**✅ Do**

```javascript
function ListItem({ className, index, style, ...rest }) {
  return (
    <li
      className={cn('listItem', className)}
      style={{ '--delay': `${index * 0.1}s`, ...style }}
      {...rest}
    />
  );
}
```

Combinar los `className` y `style` internos con los que recibimos por props permite que los consumidores puedan extender los estilos de nuestro componente. Recordad que no sabemos cómo se van a utilizar nuestros componentes y no damos nada por sentado.

## Encapsulación

![Armadillo](./images/encapsulacion.jpg)

<small>Foto por Joe Lemm en [Unsplash](https://unsplash.com/photos/H9sTbgeuosY)</small>

### Construye tus componentes aislados del mundo

Probablemente la herramienta más usada para desarrollar componentes de manera aislada es [Storybook](https://storybook.js.org). Es compatible con React, Vue, Svelte, Web Componentes, etc. Es una auténtica gozada trabajar en Storybook ya que te permite centrarte única y exclusivamente en el componente que estás construyendo en ese momento y documentar los estados de tu componente.

Lo bueno de Storybook es que podemos subirlo a cualquier servicio como [Netlify](https://www.netlify.com), [Vercel](https://vercel.com/dashboard) o [Surge](https://surge.sh) y compartir la url de nuestro Storybook con el mundo. Esto ayuda mucho en la colaboración entre desarrollo y otras disciplinas como diseño o marketing. Es una buena forma de tener tus componentes documentados sin dedicar un gran esfuerzo a construir una documentación personalizada. Además podemos incluir _stories_ no sólo de nuestros componentes, sino de cualquier cosa que necesitemos. Por ejemplo, una página donde mostramos todos los tokens de nuestro sistema de diseño: colores, escala tipográfica, espaciados, etc.

Los [_addons_](https://storybook.js.org/addons/) disponibles nos pueden ayudar a nosotros y a personas no técnicas a explorar nuestra librería de componentes:

- Controls: modifica de manera visual las props que recibe tu componente para ver cómo se comporta
- Actions: visualiza eventos que lanzan tus componentes
- Backgrounds: probar nuestros componentes con diferentes colores de fondo
- Viewport: cambia el tamaño de viewport para simular el comportamiento del componente en diferentes dispositivos
- Accessibility: comprueba la accesibilidad de tus componentes con [axe](https://www.deque.com/axe/)

Y muchos otros oficiales y de la comunidad. También podemos añadir tests de regresión visuales usando [Chromatic](https://www.chromatic.com/), [Backstop](https://garris.github.io/BackstopJS/), [Loki](https://loki.js.org/) o una de las muchas [otras opciones](https://github.com/mojoaxel/awesome-regression-testing) que existen para cazar cualquier cambio no deseado que ocurra en nuestros componentes.

### Componentes indestructibles

Ya hemos visto que podemos usar Storybook para construir nuestros componentes en un entorno aislado. Eso tiene un pega: el entorno aislado puede no ser igual al entorno donde los componentes van a ser usados. Aunque en un mundo orientado a componentes lo ideal es no tener estilos globales, eso no siempre es posible sobre todo cuando el entorno en el que se van a usar está fuera de nuestro control. Por eso, debemos intentar cargar los mismos estilos globales que sabemos se cargarán en el entorno final.

Si nuestros componentes se ven alterados por estilos globales significa que no hemos conseguido una buena encapsulación. Tendremos que ver si nuestros estilos son poco específicos y si dependen demasiado de estilos globales. Por ejemplo, si el componente `Button` depende de un `font-family` global, estamos rompiendo el principio de encapsulación. Si ese estilo global cambia nuestro componente se romperá. Para solucionarlo simplemente declaramos `font-family` en los estilos de `Button`. Debemos ser lo más específicos posibles a la hora de escribir los estilos de nuestros componentes.

### Olvídate de `margin`

_Este título tiene un poco de clickbait pero deja que me explique._ Por lo general, un componente no debería afectar a nada que esté fuera del mismo. La encapsulación debe funcionar en ambos sentidos. ¿Qué propiedad CSS rompe este principio por encima de todas? `margin`. Los márgenes [son peligrosos](https://mxstbr.com/thoughts/margin) y no sólo por el [_margin collapse_](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing).

> Hablamos de márgenes que afectan al exterior de nuestro componente. Si dentro de nuestro componente hay varios elementos y usamos `margin` para espaciarlos sin afectar al exterior no hay problema.

El espacio en diseño se podría definir como la distancia entre dos elementos. Para saber la distancia entre esos elementos debemos saber de qué elementos estamos hablando. Esto convierte el espacio en circunstancial y dependiente del contexto. El espacio entre los componentes `<Label>` y `<Select>` no tiene porque ser el mismo que entre `<Label>` y `<Text>`. Por lo tanto no tiene sentido que definamos márgenes en nuestros componentes sin saber el contexto en el que serán usados.

¿Qué podemos hacer entonces? Definir el espacio en el contexto de uso de los componentes, es decir, normalmente a nivel de app y no en la librería de componentes. Tenemos muchísimas maneras de hacerlo y todo depende de cómo estemos escribiendo nuestros estilos. Mi manera preferida para gestionar el espaciado es usar _layout components_, esto es, componentes que se encargan de espaciar y colocar elementos siguiendo las consignas marcadas por nuestro diseño. El sistema de diseño [Braid](https://seek-oss.github.io/braid-design-system/) tiene varios ejemplos:

- [Columns](https://seek-oss.github.io/braid-design-system/components/Columns): distribuye los elementos en columnas. Permite definir diferentes espaciados por _breakpoint_ y decidir si queremos apilar los elementos a partir de cierto _breakpoint_.
  ```javascript
  <Columns space={['small', 'large']} collapseBelow="tablet">
    <Column>First</Column>
    <Column>Second</Column>
    <Column>Third</Column>
  </Columns>
  ```
- [Stack](https://seek-oss.github.io/braid-design-system/components/Stack): distribuye los elementos uno debajo del otro con un espaciado entre ellos. Al igual que `Columns` permite definir un espaciado diferente por _breakpoint_.
  ```javascript
  <Stack space={['small', 'large']}>
    <Text>First</Text>
    <Text>Second</Text>
    <Text>Third</Text>
  </Stack>
  ```
- [Inline](https://seek-oss.github.io/braid-design-system/components/Inline): distribuye los elementos _inline_, pasando ocupar varias líneas si es necesario. También permite pasarle espaciados diferentes por breakpoint.
  ```javascript
  <Inline space={['small', 'large']}>
    <Text>First</Text>
    <Text>Second</Text>
    <Text>Third</Text>
  </Inline>
  ```

Esto son sólo unos ejemplos de lo que podemos conseguir con los componentes de _layout_. El cielo es el límite. Podemos crear tantos componentes de _layout_ como necesitemos para intentar forzar las convenciones de nuestro sistema de diseño (si lo tenemos) o crear opciones más genéricas como [`Flex`](https://chakra-ui.com/flex) o [`Grid`](https://chakra-ui.com/grid). Dependerá de a quien esté orientada nuestra librería de componentes. Si conseguimos que los consumidores de nuestra librería de componentes usen los componentes de _layout_ habremos ganado mucho en consistencia en el terreno de los espaciados. Además se construyen UIs a una velocidad de vértigo una vez tenemos estos componentes ya que no hay que perder el tiempo escribiendo CSS _ad hoc_ en cada pantalla o componente que estemos construyendo en nuestra app.

Si los componentes de _layout_ no son lo tuyo, [dale 5 minutos](https://signalvnoise.com/posts/3124-give-it-five-minutes). Si todavía no son lo tuyo, siempre puedes usar clases de utilidad. Si estás usando [Tailwind](https://tailwindcss.com/), [Tachyons](http://tachyons.io/) o cualquier framework similar ya lo estás haciendo. Si estás usando [Styled System](https://styled-system.com), [Theme UI](https://theme-ui.com/), [Stitches](https://stitches.dev/) u otra librería CSS-in-JS que permita leer valores directamente de un _theme_, puedes usar sus propios mecanismos en vez de clases de utilidad. Lo que es poco productivo es definir clases específicas para cada contexto en el que quieras aplicar un espaciado.

**🛑 Don't**

```javascript
<div className="mySuperSpecificUseCase">
  <Button type="button">Cancel</Button>
  <Button variant="primary" type="submit">
    Save
  </Button>
</div>
```

```css
.mySuperSpecificUseCase > * + * {
  margin-left: var(--spacing-small);

  @media screen and (min-width: 768px) {
    margin-left: var(--spacing-large);
  }
}
```

**✅ Do**

```javascript
// Con layout components
<Inline space={['small', 'large']}>
  <Button type="button">Cancel</Button>
  <Button variant="primary" type="submit">Save</Button>
</Inline>

// Con clases de utilidad
<div>
  <Button type="button">Cancel</Button>
  <Button variant="primary" type="submit" className="ml-small ml-large@tablet">
    Save
  </Button>
</div>

// Con librería CSS-in-JS que accede a tokens
<div>
  <Button type="button">Cancel</Button>
  <Button variant="primary" type="submit" sx={{ marginLeft: ['small', 'large'] }}>
    Save
  </Button>
</div>
```

Escribir CSS para cada caso específico en nuestras apps dificulta muchísimo refactorizar y mover código de un componente a otro. Además, lleva más tiempo (_naming things is hard_) y requiere estar cambiando de contexto continuamente entre la vista y CSS. Con los componentes de layout o clases de utilidad podemos crear un lenguaje común entre diseño y desarrollo. Uno de los objetivos de una librería de componentes es evitar tener que escribir tanto CSS como sea posible a los consumidores. Los componentes de _layout_ cumplen esta premisa a la perfección.

> Si las clases de utilidad te dan grima, intenta leer el post [CSS Utility Classes and "Separation of Concerns"](https://adamwathan.me/css-utility-classes-and-separation-of-concerns) con una mente abierta. Está escrito por [Adam Wathan](https://twitter.com/adamwathan), creador de [Tailwind CSS](https://tailwindcss.com).

### Bonus point: Playroom ✨

Si usas React, [Playroom](https://github.com/seek-oss/playroom) es una herramienta maravillosa para ti y tu librería de componentes. Es un _playground_ en el que escribes código JSX mientras te autocompleta los nombres de componentes, props y sus valores. Previsualizas el código escrito al instante y te permite compartir la url del código que has escrito 🤯 Además, puedes ver varios viewports a la vez y definir snippets.

Con esto se prototipa a la velocidad de la luz si los componentes tienen una API intuitiva y tenemos a nuestra disposición componentes de _layout_. Podemos crear pantallas enteras sin necesidad de levantar el proyecto. [Un ejemplo para ver cómo funciona](https://seek-oss.github.io/braid-design-system/playroom/#?code=N4Igxg9gJgpiBcIA8AhCAPABABwIZSgEsA7AcwF4AdEAWxiIFcbqA%2BS4zTJAZQBdcwAa0wBnPGBhUQAG1wAnUjGqYaJAOqEovABZSAjAAYDAUlbtOnJAAkY%2BEqUzSYANxjT9ygO4xCpbbyk5GFIGWTkzDgtOGwBPTAB5MF4GbwAjAEJzCyQAehs7MjZiLMs%2BAWExAUlgAG1qERpcaWlqABpMajDFNo6QdC6lEABdAF8iqMsAQSc5XkxeCGJJahIAMwgIieyAFRh0XhY1N0g6eYhMAAVZGLkICBp03N398Ync6ZhZ16ikAGF5KDfN5lISicTLWj0QhMTZbHZ7A4AUSIcx0MEwkFgmFSbggnjOohg6LRKlwpEIYEw2lw2GwMGITwRQK2fwg0iYxBEYKqUgaTRaIAxbNk2BEMBQuM8Un4qScvFhcOyvzZHOZiq4KAYvAWHG8vn8Um8uEErAAcrgYrlNdrFmqWTlleyaMQ7W9HaqSurLNadSwAJoWx45H22z2K3Lu52uywOlXOkSu3IgwRqiMAoFJ-hCV6Z8pFK0YIogVogNF0EQIGogGkCYYlzyaHQV%2BA1ADMACYDK0AOwANgAHK1DO2ACyjIA) el Playroom de [Braid](https://seek-oss.github.io/braid-design-system/).

> Existe [un addon](https://github.com/rbardini/storybook-addon-playroom) que integra Playroom en Storybook.

## Accesibilidad

![Pingüino poniendo un ala sobre otro pingüino](./images/accesibilidad.jpg)

<small>Foto de autor desconocido</small>

Crear una librería de componentes accesibles desde cero es un esfuerzo enorme, además de ser muy difícil. Es tentador ya que es todo un reto. Esto no debe desanimar a nadie sino prevenirnos de pegarnos un tiro en el pie.

Para construir componentes accesibles siguiendo la especificación [WAI-ARIA](https://www.w3.org/TR/wai-aria-practices/) hace falta mucho testing en diferentes combinaciones de dispositivos, navegadores y sistemas operativos. En mi opinión, la accesibilidad (no básica) es una de las facetas [más difíciles](https://twitter.com/devongovett/status/1309346046737227776) del desarrollo web.

Cualquier componente con un grado mínimo de interacción es más complicado de construir de lo que parece. Un `Tooltip` por ejemplo. Podemos pensar: "simplemente mostramos una cajita con texto cuando hacemos _hover_". Pero echando un vistazo al [código fuente](https://github.com/reach/reach-ui/blob/68a514baf85f0443a02c2f41c08994e7a5d9bb87/packages/tooltip/src/index.tsx) del `Tooltip` de [Reach UI](https://reach.tech/tooltip) nos podemos hacer una idea de todo lo que hay que tener en cuenta. Nunca es tan sencillo.

Es bueno saber elegir en qué emplear nuestros esfuerzos y hacernos las preguntas adecuadas. ¿Me beneficia en algo construir este componente desde cero en vez de usar uno existente al que puedo añadirle la capa visual que necesito? ¿De verdad voy a tener el tiempo y los recursos para poder testear apropiadamente mis componentes en las combinaciones de dispositivos, navegadores y sistemas operativos necesarias? Me temo que muy pocos desarrolladores estamos en esa disposición.

En terreno React (_es con el que más familiarizado estoy, por si no se había notado_) a día de hoy existen librerías agnósticas en cuanto a estilos que nos permiten construir componentes interactivos perfectamente accesibles:

- [React Aria](https://react-spectrum.adobe.com/react-aria/index.html), de Adobe
- [Reakit](https://reakit.io), de Diego Haz
- [Reach UI](https://reach.tech), de React Training

Sí, son nuevas dependencias y es añadir un buen puñado de los kilobytes de JavaScript, pero por desgracia hace falta mucho JavaScript para hacer nuestros componentes accesibles.

Antes de embarcarte en la aventura de crear una librería de componentes piensa si puedes usar una base existente de calidad contrastada para ahorrar tiempo y esfuerzo que puedes dedicar a cosas que impacten más en tu equipo, producto o cliente.

## Conclusión

Lo expuesto en el artículo no es ni mucho menos un conjunto de reglas inquebrantables. Las reglas están para romperlas cuando es necesario. Todo depende del contexto y nadie mejor que tú para saber cuándo tendrá sentido aplicarlas o no. Pero en el terreno de componentes visuales reutilizables, ponerse siempre en el peor escenario es lo seguro.

Hay argumentos a favor y en contra para crear componentes extensibles. Salvo que sepas con certeza qué uso exacto se le va a dar a tus componentes, crearlos de manera extensible te va a permitir descansar mejor ya que sabrás que quien los use podrá adaptarlos a sus necesidades sin necesidad de modificarlos.

Nos hemos dejado muchas cosas en el tintero, por ejemplo hacer [forward de refs](https://reactjs.org/docs/forwarding-refs.html), pero lo dejamos para otro día. Hablar de componentes no tiene fin. Si quieres, [continuamos la conversación](https://twitter.com/aarongarciah) :)

### Recursos

**Lecturas**

- [Introducing the Single Element Pattern](https://www.freecodecamp.org/news/introducing-the-single-element-pattern-dfbd2c295c5d/), por [Diego Haz](https://twitter.com/diegohaz)
- [Margin considered harmful](https://mxstbr.com/thoughts/margin), por [Max Stoiber](https://twitter.com/mxstbr)
- [Reach UI Philosophy](https://gist.github.com/ryanflorence/e5c794e6093d16a69fa88d2112a292f7), por [Ryan Florence](https://twitter.com/ryanflorence)
- [Avoid soul-crushing components](https://epicreact.dev/soul-crushing-components/), por [Kent C. Dodds](https://twitter.com/kentcdodds)
- [CSS Utility Classes and "Separation of Concerns"](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/), por [Adam Wathan](https://twitter.com/adamwathan)
- [About HTML semantics and front-end architecture](http://nicolasgallagher.com/about-html-semantics-front-end-architecture/), por [Nicolas Gallagher](https://mobile.twitter.com/necolas)

**Charlas**

- [Designing with Components](https://youtu.be/nb0_BB9apYU), por [Mark Dalgleish](https://twitter.com/markdalgleish)
- [Rethinking Design Practices](https://youtu.be/xxbc3wAztl0), por [Mark Dalgleish](https://twitter.com/markdalgleish)
- [In Defense of Utility-First CSS](https://youtu.be/R50q4NES6Iw), por [Sarah Dayan](https://twitter.com/frontstuff_io)

**Librerías**

- [React Aria](https://react-spectrum.adobe.com/react-aria/index.html), de Adobe
- [Reakit](https://reakit.io), de Diego Haz
- [Reach UI](https://reach.tech), de React Training

### Agradecimientos

- [Diego Haz](https://twitter.com/diegohaz), [Mark Dalgleish](https://twitter.com/markdalgleish) y [Devon Govett](https://twitter.com/devongovett) por compartir su conocimiento constantemente.
- [Alex Jover](https://twitter.com/alexjoverm), [Arian Zargaran](https://twitter.com/arianfront), Héctor Gómez, Julián Salgado, Miguel Molina y [Paco Segovia](https://twitter.com/SeGo) por su valioso feedback.
