# El caso de CSS-in-JS

Probablemente hayas oído hablar de CSS-in-JS. Puede que seas un firme detractor o un fiel defensor. O simplemente, te da igual. Vamos a hacer un análisis sobre esta forma de escribir CSS y si os puedo ayudar a decidir si se adapta a vuestro contexto, mejor que mejor.

## Aclaraciones previas

⚠️ El texto está basado en mi experiencia. Algunas de las cosas aquí escritas son opiniones totalmente subjetivas, como no podría ser de otra forma.

### ¿Qué es CSS-in-JS?

CSS-in-JS se refiere a formas de **escribir CSS usando JavaScript**, normalmente con un framework JavaScript orientado a componentes. Suena muy mal, lo sé. Se escribe CSS-in-JS mediante alguna de las numerosas librerías existentes, [la mayoría para ser usadas con React](http://michelebertoli.github.io/css-in-js/), cada una con su propia implementación aunque muchas comparten sintaxis. Las librerías más extendidas son [styled-components](https://www.styled-components.com/) y [Emotion](https://emotion.sh). Aunque internamente [las diferencias son muchas](https://twitter.com/kyehohenberger/status/1070901663622283265), podemos usar las características básicas de ambas de forma parecida.

### ¿CSS-in-JS son estilos inline?

CSS-in-JS hoy por hoy **NO son estilos inline**, aunque esta confusión tiene un motivo. En el año 2014, [Christopher Chedeau](http://blog.vjeux.com/) (conocido como [Vjeux](https://twitter.com/Vjeux)), ingeniero en Facebook y por aquel entonces miembro del Core Team de React, dio una charla titulada "[React: CSS in your JS](https://vimeo.com/116209150)" y [acuñó el término "CSS-in-JS"](https://speakerdeck.com/vjeux/react-css-in-js?slide=25). En la charla explicaba graves problemas a la hora de poder mantener el código CSS de proyectos enormes y cómo escribiendo CSS en JavaScript y haciendo uso de [estilos inline](https://speakerdeck.com/vjeux/react-css-in-js?slide=29) habían solucionado esos problemas.

Off-topic: Como curiosidad, años después Christopher Chedeau se encargó de mejorar, [mantener](https://github.com/prettier/prettier/commits?author=vjeux) y popularizar [Prettier](https://prettier.io/). Gracias Christopher.

### ¿Es CSS-in-JS específico a React?

No, aunque se popularizó en el ecosistema React y la mayoría de las librerías están creadas expresamente para React. Para otros frameworks como [Vue](https://vuejs.org/v2/guide/comparison.html#Component-Scoped-CSS) o [Svelte](https://svelte.dev/blog/svelte-css-in-js) también existen librerías de CSS-in-JS pero mucho menos extendidas. En el caso de Vue, la propia [documentación](https://vuejs.org/v2/guide/comparison.html#Component-Scoped-CSS) se desmarca en cierto modo diciendo que la manera por defecto de hacer estilos en Vue es con la etiqueta `<style>` en los *single file components*.

### ¿A qué se parece eso de CSS-in-JS?

Un ejemplo básico creando un componente `Button` de React con styled-components.

    import styled from 'styled-components';

    const Button = styled.button`
    	color: papayawhip;
    `;

    // Lo usamos así en otra parte de nuestra app
    <Button>Click me!</Button>

    // Y algo así acabará en el DOM, generando las clases CSS por nosotros
    <style>
    	.jUqbSf {
    		color: papayawhip;
    	}
    </style>

    <button class="jUqbSf">Click me!</button>

## ¿Por qué alguien escribiría CSS en JavaScript?

![](por-que@2x.jpg)

Foto de [Juan Rumimpunu](https://unsplash.com/@earbiscuits) en [Unsplash](https://unsplash.com/photos/nLXOatvTaLo)

Iniciarse en CSS es muy sencillo. Existen propiedades y sus respectivos valores, que juntos constituyen una declaración. Si juntamos un selector con una o varias declaraciones, tenemos una regla CSS. Fácil.

El problema viene cuando nos vamos adentrando más y más. A los conceptos básicos como la cascada, la herencia, block formatting context, margin collapse, posicionamiento, transiciones, transformaciones, etc. se le suman cada año más cosas: Flexbox, CSS Grid, Subgrid, blend modes, filters, feature queries, etc. Por eso, hasta los más veteranos siguen sin conocer CSS a la perfección. Nunca vamos a acabar de aprenderlo. Usar **CSS-in-JS no nos va a evitar tener que aprender CSS** de ninguna manera.

> CSS-in-JS no va a evitar que aprendamos CSS.

Ahora vamos a imaginar que nos manejamos muy bien con CSS. Vemos un diseño y en nuestra cabeza ya imaginamos el código para implementarlo. Nuestro compañero diseñador nos enseña una animación y ya sabemos cómo hacerla sin ni siquiera tocar el teclado. Es decir, puedes hacer cualquier cosa por conocimientos CSS. Pero una cosa es conocimiento sobre CSS puro y otra la arquitectura del CSS. Podemos conocer CSS a la perfección y no saber cómo hacer una buena arquitectura CSS.

Siguiendo con la hipótesis, imagina que estamos en un gran proyecto desarrollando el frontend de una [SPA](https://en.wikipedia.org/wiki/Single-page_application) con un framework orientado a componentes como Vue o React. No estamos desarrollando solos sino que un puñado de desarrolladores está involucrado. Pongamos que usamos Sass, [BEM](https://en.bem.info/) como nomenclatura de nuestras clases, [ITCSS](https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture/) para organizar nuestro CSS por capas y tener la especificidad controlada, algunos [namespaces](https://csswizardry.com/2015/03/more-transparent-ui-code-with-namespaces/) para diferenciar entidades y algunos prefijos como `is-` o `has-` de [SMACSS](http://smacss.com/) para indicar qué clases indican un estado (BEM + ITCSS + Namespaces = [BEMIT](https://csswizardry.com/2015/08/bemit-taking-the-bem-naming-convention-a-step-further/)).

Para alguien acostumbrado a ello, puede ser relativamente sencillo. Para alguien como yo que disfruto con esa manera de hacer CSS, puede ser una gozada. Pero aprender a trabajar así puede llevar mucho tiempo, no lo puedo negar. **Es muy difícil forzar a un equipo a mantener la consistencia y la calidad en el código CSS durante un largo periodo de tiempo cuando nos basamos en convenciones**, se necesitan reglas muy estrictas que son difíciles de cumplir y forzar. Existen maneras de ayudar con [linters](https://stylelint.io/), pero sigue siendo muy difícil. En proyectos que construyen y mantienen equipos con miembros de niveles muy diversos, donde unos se van y otros se quedan, donde gente se va de vacaciones y otros tienen que tocar código que no controlan ni conocen, en mi experiencia, es imposible.

Podemos argumentar que esto ocurre con casi cualquier lenguaje, lo cual es cierto, y por ello la comunidad de CSS-in-JS está buscando caminos de simplificar escenarios difíciles en la escritura de CSS en los frameworks JavaScript orientados a componentes.

> **CSS-in-JS es un intento por simplificar el desarrollo de UIs** con frameworks JavaScript orientados a componentes, principalmente en React.

Podríamos asegurar al 99% que si no estamos usando un framework JavaScript, no vamos a usar CSS-in-JS. El 1% lo dejo porque siempre tiene que haber de todo 😅. Viendo a las comunidades de los frameworks mayoritarios, es muy probable que nos lo planteemos sólo si usamos React, donde CSS-in-JS está muy extendido.

**CSS-in-JS no viene a salvar CSS** ni a sustituir a Sass, BEM ni nada por el estilo. Se puede seguir desarrollando como lo hemos hecho hasta ahora sólo que ahora tenemos una alternativa que en algunos escenarios puede simplificar mucho las cosas. Al igual que usamos frameworks como abstracción sobre JavaScript para desarrollar aplicaciones que se encarguen de actualizar el DOM por nosotros y hacer el [código más mantenible](https://twitter.com/dan_abramov/status/842329893044146176), CSS-in-JS es una abstracción sobre CSS. Y como sabemos, [toda abstracción conlleva un coste](http://250bpm.com/blog:86). Si hiciéramos nuestras aplicaciones web con código imperativo en vez de usar un Vue o React, con muchísimo esfuerzo quizás podríamos llegar a hacer que fueran más rápidas. El problema es que nos llevaría mucho más tiempo hacerlas y mantenerlas sería un imposible. Para decidirnos a usar CSS-in-JS, lo mejor es conocer sus ventajas e inconvenientes.

> **CSS-in-JS no viene a salvar CSS.** Viene a solucionar problemas concretos en contextos concretos.

CSS-in-JS es un intento por avanzar el ecosistema a nuevas cotas gracias a una gran cantidad de experimentación que se ha llevado a cabo en la comunidad Frontend en los últimos años. Pero repito, no es aplicable en todos los contextos ni mucho menos.

> ⚠️ Los términos "calidad", "entendible", "escalable" o "mantenible" para referirnos a código, especialmente CSS, son muy subjetivos y abstractos. Dependen siempre del contexto [como diría Juan Delgado](https://vimeo.com/178441743#t=360s) en su magnífica charla "Product vs Craft".

## Principales ventajas de CSS-in-JS

![](ventajas@2x.jpg)

Foto de [Bonnie Kittle](https://unsplash.com/@bonniekdesign) en [Unsplash](https://unsplash.com/photos/RMH3j6PPY-0)

> ⚠️ Los ejemplos de código usan React con la sintaxis de [styled-components](https://www.styled-components.com/) con [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals), probablemente la sintaxis más conocida. Otras librerías como [Emotion](https://emotion.sh/) o [Linaria](https://linaria.now.sh/) también la usan. Si no estás familiarizado con React, una buena lectura para iniciarse es el [tutorial introductorio](https://es.reactjs.org/tutorial/tutorial.html) de la documentación oficial de React.

Vamos a ver las principales ventajas que nos aporta CSS-in-JS. La lista no es ni mucho menos exhaustiva.

### Estilos dinámicos

Nuestros estilos tienen acceso a las props que le llegan a nuestro componente y por tanto podemos crear estilos 100% dinámicos.

    // Dependiendo de la prop `open` podemos dar diferentes
    // estilos a nuestro componente
    const Sidebar = styled.div`
    	display: ${props => props.open ? 'block' : 'none' };
    	color: black;
    	background-color: white;
    `;

Podemos definir varios estilos dependiendo de una sola prop, a modo de variantes:

    const Button = styled.button`
    	color: white;
    	background-color: #fc5d5f;
    	font-size: ${props => props.size === 'large' ? '2rem' : '1.6rem' };

    	${props => props.variant === 'primary' &&  css`
    		background: white;
    		color: #fc5d5f;
    	`};
    `;

### *All-in* en desarrollo orientado a componentes

Una de las grandes ventajas es no tener que estar creando clases para todos los posibles estados de nuestros componentes. En CSS/Sass con BEM podría ser algo así:

    .btn {
    	color: white;
    	background-color: #fc5d5f;
    	font-size: 1.6rem;
    }

    .btn--large {
    	font-size: 2rem;
    }

    .btn--primary {
    	background: white;
    	color: #fc5d5f;
    }

Después en nuestro componente deberíamos añadir condiciones para pintar unas clases u otras (podríamos abreviarlo utilizando alguna librería como [classnames](https://www.npmjs.com/package/classnames#usage-with-reactjs)):

    const Button = ({ size, variant, ...props }) => {
    	const className = 'btn';

      if (size === 'large') {
    		className += 'btn--large';
    	}

      if (variant === 'primary') {
    		className += 'btn--primary';
    	}

    	return (
    		<button className={className} {...props} />
    	);
    };

Usando CSS-in-JS evitamos ese paso intermedio de tener que pensar en clases y tener que componer los nombres de nuestras clases a aplicar en nuestros componentes. Los estilos de nuestros componentes reaccionan a las props que reciben. Cuando estamos desarrollando en un framework orientado a componentes, es una auténtica liberación. **Ya no pensamos en clases, lo hacemos en la unidad de construcción básica: componentes**. Eso no quiere decir que si dos componentes comparten estilos no los podamos compartir, se puede hacer en CSS-in-JS.

> CSS-in-JS nos permite pensar únicamente en nuestra unidad de construcción básica: los componentes.

### La potencia de JS en nuestras manos

Podemos usar cualquier código JavaScript para generar nuestros estilos de manera dinámica en tiempo de ejecución, así que debemos usarlo con cuidado.

Un ejemplo. Queremos animar un componente y que se mueva del punto *X* al punto *Y*. Cuando ese elemento es muy grande, la animación debería tardar menos ya que tiene menos distancia a recorrer. Si ese elemento es más pequeño, tardará más tiempo. Ya que estamos en terreno JavaScript, podemos calcular el ancho de nuestro elemento y calcular el tiempo que tardará la animación.

### Theme global

Nuestros componentes pueden tener acceso a un theme global de forma dinámica. Si este theme cambia alguno de sus valores, nuestros componentes se actualizan automáticamente.

    const Button = styled.button`
    	color: white;
    	background-color: #fc5d5f;

    	${props => props.theme.mode === 'dark' && css`
    		color: #fc5d5f;
    		background-color: white;
    	`}
    `;

### Critical CSS

Si hacemos [code-splitting](https://webpack.js.org/guides/code-splitting/), vamos a enviar sólo el CSS que necesita nuestra aplicación. Esto tiene algunas pegas que luego veremos. Cada vez es más fácil hacer code-splitting con los frameworks modernos como [Vue](https://vuejs.org/v2/guide/components-dynamic-async.html#Async-Components), [React](https://reactjs.org/docs/code-splitting.html#reactlazy) o [Svelte](https://github.com/lukeed/svelte-demo/blob/master/src/components/App.svelte#L36) por lo que deberíamos poder hacerlo con poco esfuerzo.

### Scope de estilos

Las librerías de CSS-in-JS van a generar nombres únicos de clases para que no existan conflictos. Esto no es ninguna innovación de las librerías de CSS-in-JS puesto que los [scoped styles](https://vue-loader.vuejs.org/guide/scoped-css.html) de Vue o [CSS Modules](https://github.com/css-modules/css-modules) ya lo hacen, pero es bueno recordarlo.

No es necesario que nos calentemos la cabeza. Podemos tener varios *container* en distintos componentes, sus estilos nunca van a colisionar y tampoco nos tenemos que preocupar de convenciones de nombres, namespaces ni nada por el estilo. Cuando desarrollamos nuestro componente, nos preocupamos única y exclusivamente de nuestro componente. Es liberador, especialmente si desarrollamos con herramientas como [Storybook](https://storybook.js.org/), el cual es recomendable usemos o no CSS-in-JS.

### HTML y CSS, juntos y revueltos

No es necesario estar separando el HTML de nuestro CSS. Cuando creamos componentes HTML y CSS forman una unidad.

### Adiós a la especificidad

Rara vez tendremos que preocuparnos por la [especificidad](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity) y eso para muchos desarrolladores es una bendición, es un auténtico quebradero de cabeza para muchísima gente.

### La prop `as`

Algunas librerías de CSS-in-JS ofrecen la posibilidad de usar la prop `as` que modifica el elemento que queremos renderizar. Por ejemplo, podemos crear un componente que renderiza un `<h1>` pero luego podemos decirle que se renderice como un `<h3>`. La `as` prop la podemos [implementar fácilmente](https://codesandbox.io/s/as-prop-m43ow), pero es muy cómodo no tener que hacerlo en cada componente en el que lo necesitemos.

    const Heading = styled.h1`...`;
    const Button = styled.button`...`;

    ---

    <Heading>I'm a nice looking h1</Heading> // <h1>
    <Heading as="h3">I look pretty good too to be an h3</Heading> // <h3>

    <Button>Click me, I'm a button!</Button> // <button>
    <Button as="a">I look like a button, but I'm a link!</Button> // <a>

### CSS variables (*kinda*) en JavaScript

Muchos proyectos deben dar soporte a navegadores que no soportan CSS variables. Usando CSS-in-JS puedes tener una funcionalidad parecida y con un mayor porcentaje de compatibilidad entre navegadores antiguos.

### Compartir estilos y *know-how* entre web y nativo

Algunos equipos que desarrollan en React y React Native, comparten estilos y conocimientos ya que tienen un mismo mecanismo para dar estilos a sus componentes.

## Problemas de CSS-in-JS

![](inconvenientes@2x.jpg)

Foto de [Manuel Oppel del Rio](https://unsplash.com/@manutastic) en [Unsplash](https://unsplash.com/photos/uvQv0LCj5ac)

CSS-in-JS viene con unas cuantas pegas, algunas no pequeñas, que debemos conocer antes de decidir usarlo. Normalmente, elegir un camino en el desarrollo de software significa renunciar a otros. Lo importante es decidir intentando prever las consecuencias de nuestra elección. Esta es una lista no exhaustiva pero contiene los que considero mayores inconvenientes.

### Coste de ejecución

La mayoría de librerías de CSS-in-JS usan un *runtime*. Es decir, una dependencia (por ejemplo styled-components) acaba en el bundle de tu aplicación y se encarga de generar las clases y las etiquetas `<style>` de manera dinámica en tiempo de ejecución. Suena muy lento, pero no lo es. Estas librerías están muy optimizadas y en la mayoría de casos el impacto en la velocidad de tu app no se va a notar. Al fin y al cabo, los frameworks orientados a componentes ya están creando y actualizando el DOM sin cesar por lo cual el impacto en el *runtime* no es notable en muchos casos.

Hay opciones interesantes como [Linaria](https://linaria.now.sh/) que no tienen un coste en el *runtime*, es decir, extraen todo el CSS a estático en tiempo de compilación. Linaria convierte las expresiones JavaScript que dependen de las props de tu componente en CSS Variables. Puedes verlo inspeccionando [este ejemplo](https://09jc7.sse.codesandbox.io/). Tiene sus limitaciones, pero es una opción muy interesante y que seguro va a traer cosas buenas en el futuro.

### Sin paralelización en la descarga de CSS y JavaScript

Al meter nuestro CSS en JavaScript, estamos evitando que el navegador pueda descargar paralelamente CSS y JavaScript. Con lo cual, debemos plantearnos seriamente si usar CSS-in-JS si no estamos haciendo code-splitting. Os dejo una buena lectura para entender mejor cómo los navegadores gestionan la descarga de CSS y JavaScript: [CSS and Network Performance](https://csswizardry.com/2018/11/css-and-network-performance/) por [Harry Roberts](https://twitter.com/csswizardry).

### Server Side Rendering

Cuando hacemos Server Side Rendering con las librerías de CSS-in-JS, la extracción de critical CSS suele ser automática. ¿Qué problema hay? La duplicación de CSS. Es decir, nuestro critical CSS se envía dentro de etiquetas `<style>` en el HTML renderizado en el servidor pero también se manda dentro de nuestro bundle JavaScript. Por ello, si no estamos haciendo code-spliting, no es recomendable usar CSS-in-JS.

### Impacto en los desarrolladores

El impacto que puede suponer en un desarrollador pasar de pensar en clases, BEM, OOCSS, ITCSS, etc. a un mundo sin selectores puede ser duro. En mi experiencia, los desarrolladores que no tienen un background como maquetador y por tanto no están tan familiarizados con las formas clásicas **de hacer CSS, se acostumbran muy rápido a la hora de hacer CSS-in-JS si ya saben lo mínimo sobre un framework orientado a componentes; el esquema mental ya lo tienen. Les resulta más fácil aprender las cuatro peculiaridades de CSS-in-JS que aprender y aplicar convenciones y nomenclaturas. Lo cual no digo que sea una razón suficiente para usarlo.

La división que existe en el mundo frontend [es](https://css-tricks.com/the-great-divide/) [real](http://bradfrost.com/blog/post/frontend-design-react-and-a-bridge-over-the-great-divide/), y CSS-in-JS puede agrandarla todavía más si no seleccionamos bien cuándo usarlo dependiendo del contexto.

> 💡 Linaria [aporta soluciones a muchos de los problemas descritos](https://github.com/callstack/linaria/blob/master/docs/BENEFITS.md#advantages-over-other-css-in-js-solutions). Si no necesitas complejos estilos dinámicos, pruébalo porque ofrece casi todas las ventajas de CSS-in-JS sin sus grandes inconvenientes.

## Rechazo a CSS-in-JS

Durante los últimos años se han visto muchos artículos criticando a CSS-in-JS. En algunos de esos artículos los autores dicen que ellos nunca usan frameworks JavaScript, otros incluso que no usan ninguna dependencia externa para crear sus webs estáticas y que pueden llevar a cabo sus proyectos sin problemas. Algunos no necesitan npm. ¡Evidentemente CSS-in-JS no es para esos casos! CSS-in-JS se ha creado para mejorar la creación de componentes y solventar problemas en cierto tipo de proyectos, no para ser usado en todos los contextos. Es muy importante recalcarlo.

> Hay que recalcar que CSS-in-JS no aplica a todos los contextos, de hecho, aplica a muy pocos.

Se ve una gran falta de entendimiento entre defensores y detractores. Los defensores a veces no enfatizamos lo suficiente que CSS-in-JS no se debe usar en todas las situaciones. Algunos detractores muchas veces nos enfrentamos a problemas diametralmente opuestos y es evidente que no necesitamos CSS-in-JS en casos de uso donde no encaja. Otros lo rechazamos tajantemente sin haberlo probado o sin mostrar el mínimo interés por el progreso que pueda aportar o pensamos que se está mancillando CSS.

Tampoco ayuda que nos movamos por modas o por el *hype* a la hora de seleccionar nuestras herramientas, aunque en muchas ocasiones no somos libres. Podemos estar restringidos en la toma de decisiones por nuestro equipo, tech leads, código legado, estándares de empresa, conocimientos, tiempo, recursos, etc. No todo es blanco o negro.

## Design Systems + CSS-in-JS = ❤️

El contexto en el que mejor se adapta CSS-in-JS es a la hora de crear y mantener Design Systems o Pattern Libraries con un framework como React. La comunidad de diseño y React convergen y evolucionan creando herramientas que permitan mejorar los flujos entre diseño y desarrollo. [react-sketchapp](http://airbnb.io/react-sketchapp/), [Framer X](https://www.framer.com/), [Relate](https://relate.app/) y [Modulz](https://www.modulz.app/) son sólo algunos ejemplos de herramientas que exploran este terreno. ¿Que las herramientas deberían ser *framework agnostic*? Sí, pero es que esas herramientas ya trabajan una abstracción muy similar a la que usa React: los componentes.

En un mundo perfecto, un Design System [debería ser agnóstico](http://bradfrost.com/blog/post/managing-technology-agnostic-design-systems/) a cualquier framework, pero la realidad es que muchos de los equipos construyendo un Design System ya se han casado con uno u otro framework y hacerlo agnóstico es un coste adicional que en muchos casos no compensa. Decisiones complejas por todos lados.

### Styled System

[Styled System](https://styled-system.com/) merece mención especial. Su autor, [Brent](https://jxnblk.comç) [Jackson](https://jxnblk.com), muy conocido en la comunidad React, [lleva años explorando](https://jxnblk.com/blog/interoperability/) herramientas y fórmulas para mejorar el diseño y creación de componentes y Design Systems. Muchas de sus herramientas y librerías están orientadas a ello: [Styled System](https://styled-system.com/), [System UI](https://system-ui.com), [Theme UI](https://theme-ui.com/), [Rebass](https://rebassjs.org), [CSS Stats](https://cssstats.com/), [Palx](https://palx.jxnblk.com/), [Basscss](https://basscss.com/), [Colorable](https://colorable.jxnblk.com/), [Spectral](https://jxnblk.github.io/Spectral/), [Paths](https://jxnblk.github.io/paths/), [microicon](https://icon.now.sh/), etc.

¿Qué tiene de especial Styled System? Nos permite definir APIs (props) de nuestros componentes para que tengan la posibilidad de consumir valores de un theme global. Veamos un ejemplo corto puesto que Styled System da para mucho.

Primero debemos definir nuestro theme siguiendo la especificación [System UI](https://system-ui.com/) y con el concepto de [escalas](https://styled-system.com/theme-specification#scale-objects) para nuestros breakpoints, tamaños de fuente, colores, etc. Podemos elegir si definir las unidades en `px`, `em`, `rem`, etc. Como queramos.

    const theme = {
    	breakpoints: ['40em', '52em', '64em'],
    	space: [0, 4, 8, 16, 32, 64],
    	fonts: {
    		regular: 'Papyrus',
    		mono: 'OperatorMono'
    	}
    	fontSizes: [12, 16, 20, 32],
    	lineHeights: [14, 20, 28, 40],
    	colors: {
    	  blue: '#07c',
    		// También podemos definir escalas de colores
    	  blues: [
    	    '#004170',
    	    '#006fbe',
    	    '#2d8fd5',
    	    '#5aa7de',
    	  ]
    	}
    };

Ya con nuestro theme definido podemos definir las props de nuestros componentes que consumirán nuestro theme global.

    import styled from 'styled-components';
    import { fontFamily, space, color } from 'styled-system';

    // Le decimos a nuestro componente que queremos que acepte props para
    // poder personalizar su font-family, espaciado (margin y padding) y color
    const Heading = styled.h1`
    	${fontFamily};
    	${space};
    	${color};
    `;

Ahora nuestro componente aceptará las props relacionadas con las utilidades que hemos añadido a los estilos de nuestro componente. En nuestro caso `fontFamily`, `space` (padding y margin) y `color`. Hay valores que aceptan escalas como `marginBottom`. Al pasarle un número como 2, le estamos diciendo que coja el item número 3 de su escala correspondiente (`space`) ya que los índices de las escalas empiezan en 0.

Pasando un array en una de las props, definimos estilos responsive. Es decir, cada valor se asignará a una media query `(min-width: [breakpoint])` en el orden en el que están definidas en el theme. El primer valor del array no va asignado a ningún breakpoint, se trata del valor inicial. De esta forma podemos desarrollar responsive a la velocidad de la luz 😮.

> Styled System es una de las grandes joyas que ha traído CSS-in-JS.

    <Heading fontFamily="mono" marginBottom={[2, 3]} color="blues[2]">

    // Las props pasadas serán transformadas en el siguiente código CSS:
    // {
    //	 font-family: 'OperatorMono';
    //	 margin-bottom: 8px;
    //	 color: #2d8fd5;
    //
    //	 @media (min-width: 40em) {
    //		 margin-bottom: 16px;
    //	 }
    // }

Con las facilidades que ofrece esta librería, no es extraño que varios Design Systems ya lo estén usando [junto a otras herramientas](https://jxnblk.com/blog/the-modern-front-end-design-system/) como [Storybook](https://storybook.js.org/), [Gatsby](https://www.gatsbyjs.org/), [MDX](https://mdxjs.com/), [react-live](https://github.com/FormidableLabs/react-live), etc. que hacen que la experiencia al consumir, consultar y desarrollar los componentes sea realmente impresionante. Echad un ojo a las siguientes documentaciones, se puede editar en vivo el código de los componentes para ver cómo se comportan.

- [Radix](https://radix.modulz.app) de [Modulz](https://www.modulz.app/) (impresionante el trabajo de [Pedro Duarte](https://twitter.com/peduarte) aquí)
- [Primer Components](https://primer.style/components/) de GitHub
- [Seeds](https://sproutsocial.com/seeds/components) de Sprout Social
- [Palette](https://palette.artsy.net/) de Artsy

No se me ocurre mejor manera de convencer a alguien para que use algo que el hecho de que pueda toquetearlo desde el primer instante sin necesidad de instalar nada. Bravo.

## ¿Debo usar CSS-in-JS?

![](debo-usarlo@2x.jpg)

Foto de [Andre Mouton](https://unsplash.com/@andremouton) en [Unsplash](https://unsplash.com/photos/GBEHjsPQbEQ)

No lo sé 😌. CSS-in-JS tiene algunas ventajas que no ofrecen enfoques más tradicionales pero a su vez viene con un puñado de inconvenientes. Depende totalmente del contexto en el que estemos desarrollando. Vamos a plantear algunas situaciones de ejemplo para hacernos una idea:

- "Voy a crear una landing estática".
No lo necesitas.
- "Voy a construir un blog con un static site generator".
Muy probablemente no lo necesitas. Recuerda que la mayoría de las librerías CSS-in-JS tienen un runtime que acaba en el bundle de tu web y la gracia de un static site generator es que genera un sitio estático 😅.
- "Nuestro equipo está cómodo usando un acercamiento más tradicional y nuestra velocidad de desarrollo creemos que es muy buena".
No lo uses. Aunque no estaría de más hacer una pequeña exploración para conocer de primera mano ventajas e inconvenientes.
- "Desarrollamos casi todas nuestras webs en lenguajes de servidor y las páginas tienen poca interactividad"
Asumiendo que no se está usando un framework JavaScript, al 99% no es necesario.
- "Vamos a hacer una SPA y necesitamos consumir componentes de un sistema de diseño que ya usa CSS-in-JS".
Es muy probable que lo necesites.
- "Estamos creando un Design System o Pattern Library en React".
Si tu equipo/empresa ha decidido apostar todo a React (no vamos a discutir aquí si eso es acertado o no), lo podéis considerar seriamente. Si el valor que os va a aportar es mayor que las desventajas, ¡adelante! Aunque si hay tiempo y recursos, lo mejor es probar primero.
- "Usamos React y en el equipo escasea la experiencia en arquitectura CSS".
Si las desventajas os parecen asumibles y tenéis tiempo de probar CSS-in-JS, ¡adelante!

Lo importante es seleccionar las herramientas que nos vayan a permitir desarrollar mejor y a un buen ritmo y siempre teniendo en cuenta todo el contexto.

## CSS-in-JS hoy y mañana

Debemos ser críticos con las herramientas que elegimos y no dejarnos llevar sólo por la comodidad que nos ofrecen. Si está en nuestra mano, debemos estudiar ventajas e inconvenientes y no usar una herramienta para un propósito para la que no ha sido creada.

CSS-in-JS es hoy por hoy una alternativa estable y parece que ha venido para quedarse. Aún así, es un terreno en el que se sigue avanzando y buscando alternativas constantes para mejorar la autoría de estilos en un mundo orientado a componentes. [Linaria](https://linaria.now.sh/) es un buen ejemplo de ello. Podemos estar de acuerdo o no, pero quizás podamos [entender el porqué](https://gist.github.com/threepointone/731b0c47e78d8350ae4e105c1a83867d) parte de la comunidad Frontend ha elegido explorar esa vía.

### Agradecimientos

Gracias [Paco Segovia](https://twitter.com/SeGo), [Pedro Fernández](https://twitter.com/fernandezamor) y [Laura Bonmati](https://twitter.com/LauraBonmati) por vuestro feedback 😄

Foto de portada por [Claudiu Pusuc](https://unsplash.com/@claudiupusuc) en [Unsplash](https://unsplash.com/photos/Z6P0mehUNRI).

---

**Lecturas recomendadas**

- [The Differing Perspectives on CSS-in-JS](https://css-tricks.com/the-differing-perspectives-on-css-in-js/) - Chris Coyier
- [What actually is CSS-in-JS?](https://medium.com/dailyjs/what-is-actually-css-in-js-f2f529a2757) - Oleg Isonen
- [The tradeoffs of CSS-in-JS](https://www.freecodecamp.org/news/the-tradeoffs-of-css-in-js-bee5cf926fdb/)- Oleg Isonen
- [Why I Write CSS in JavaScript](https://mxstbr.com/thoughts/css-in-js/) - Max Stoiber
- [Why you should definitely learn how to use CSS-in-JS](https://jxnblk.com/blog/why-you-should-learn-css-in-js/) - Brent Jackson
- [Is CSS-in-JS really bad for UX?](https://medium.com/@okonetchnikov/is-css-in-js-really-bad-for-ux-e9cce7b2da83) - Andrey Okonetchnikov
- [The zen of Just Writing CSS](https://svelte.dev/blog/the-zen-of-just-writing-css) - Rich Harris
- [CSS and Network Performance](https://csswizardry.com/2018/11/css-and-network-performance/) - Harry Roberts
- [What's wrong with CSS-in-JS?](https://gomakethings.com/whats-wrong-with-css-in-js/) - Chris Ferdinandi

**Autores**

- [Max Stoiber](https://twitter.com/mxstbr)
- [Kye Hohenberger](https://twitter.com/kyehohenberger)
- [Brent Jackson](https://twitter.com/jxnblk)
- [Mark Dalgleish](https://twitter.com/markdalgleish)
- [Glen Maddern](https://twitter.com/glenmaddern)
- [Sunil Pai](https://twitter.com/threepointone)
- [Satyajit Sahoo](https://twitter.com/satya164)
- [Oleg Isonen](https://twitter.com/oleg008)

**Otros Design Systems y Pattern Libraries construidos con CSS-in-JS**

- [Reakit](https://reakit.io/)
- [Garden](https://garden.zendesk.com/react-components/) (Zendesk)
- [Atlaskit](https://atlaskit.atlassian.com/) (Atlassian)
- [Cosmos](https://auth0-cosmos.now.sh) (Auth0)
- [Priceline One](https://pricelinelabs.github.io/design-system/) (Priceline)

### Autor

![](aaron-garcia.jpg)

Marido, padre² y Frontend Engineer, por ese orden. Intento hacer cosas bien en Cabify. Sueño con que algún día la distancia entre diseñadores y desarrolladores se reduzca. Disfruto coorganizando [Alicante Frontend](https://www.alicantefrontend.es/) y [Vue Day](https://vueday.org/). Lo bueno y breve.

[https://twitter.com/aarongarciah](https://twitter.com/aarongarciah)

[https://www.linkedin.com/in/aarongarciah/](https://www.linkedin.com/in/aarongarciah/)
