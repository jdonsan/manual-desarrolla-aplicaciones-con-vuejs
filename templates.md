# Capítulo 2. Trabajando con templates

Cuando trabajamos en Web, una de las primeras decisiones que solemos tomar es la forma en que se van a pintar los datos de los usuarios por pantalla:

Podemos optar por guardar una serie de HTMLs estáticos que ya cuentan con la información del usuario necesaria incrustada y almacenados dentro de base de datos. Este es el modelo que suelen desarrollar los CMS de muchos e-commerces o blogs personales.

Otra opción sería la de renderizar o pintar estos datos dentro del HTML de manera dinámica. La idea subyace en crear una plantilla del HTML que queremos mostrar al usuario, dejando marcados aquellos huecos donde queremos que se pinten variables, modelos o entidades determinadas. Este es el sistema que suelen usar la gran mayoría de WebApps actuales que se basan en SPA.

Cada una de las dos opciones es válida y dependerá del contexto en el que nos encontremos la forma en que actuemos. VueJS por ejemplo, opta por la segunda opción. VueJS nos permite desarrollar plantillas HTML que se pueden renderizar tanto en tiempo de back como de front de manera dinámica.

El post de hoy trata de explicar toda esta sintaxis mediante la explicación los conceptos de interpolación, directiva y filtro. Espero que os guste:

## ¿Cómo funciona?

Como decíamos en la introducción, VueJS cuenta con un motor de plantillas muy parecido al de librerías como HandlebarsJS. Lo que nos permite es crear HTML  + una sintaxis especial que nos permite incluir datos del usuario. Aunque tiene soporte para JSX, se ha optado por un marcado mucho más ligero y entendible por un abanico de desarrolladores mayor. Tanto maquetadores, como fronts, como ciertos perfiles back están muy acostumbrados a este tipo de marcado dinámico.

![Mecanismo interno de un motor de plantillas](/images/templates/templateenginediagram1.gif)

Lo bueno de VueJS es que el cambio de estas plantillas en DOM no se produce de manera directa. Lo que VueJS hace es mantener una copia del DOM cacheada en memoria. Es lo que se suele denominar Virtual DOM y es lo que permite que el rendimiento de este tipo de frameworks no se vea penalizado por la cantidad de cambios que se pueden producir de manera reactiva.

![Funcionamiento interno del Virtual DOM](/images/templates/virtual_dom_diff.png)

## La interpolación

Una interpolación es la posibilidad de cambiar partes de una cadena de texto por variables. Para indicar dónde queremos un comportamiento dinámico de una cadena de texto dentro de VueJS, lo podemos indicar, marcando la variable que queremos interpolar con las dobles llaves \(también conocidos como 'bigotes'\):

```html
<h1>Bienvenido {{ user.name }}</h1>
```

### Interpolando HTML

Este sistema nos sirve para interpolar variables que no contienen HTML. Si necesitáramos que la inserción sea interpretada como HTML tendríamos que indicarlo con la siguiente directiva `v-html`:

```html
<div v-html="rawHtml"></div>
```

Este caso puede ser una mala práctica si lo que intentamos es estructurar nuestras vistas por medio de este método. El concepto de componente es el idóneo para reutilizar elementos. Intentemos usar este método solo en casos en los que no es posible otro método y teniendo en cuenta que el HTML incrustado sea controlado 100% por nosotros y no por el usuario, de esta manera podremos evitar ataques por XSS.

### Interpolando atributos

VueJS no solo nos deja interpolar textos de nuestros elementos HTML, también nos va a permitir tener control sobre los valores de nuestros atributos. Podríamos querer indicar si un botón tiene que estar habilitado o deshabilitado dependiendo de la lógica de la aplicación:

```html
<button type="submit" v-bind:disabled="isFormEmpty">Entrar</button>
```

Podríamos tener este comportamiento con cualquier atributo de un elemento HTML.

### Interpolando por medio de expresiones

En VueJS se puede interpolar texto por medio de pequeñas expresiones. Es una posibilidad de incluir un poco de lógica en nuestra plantilla. Podríamos por ejemplo evaluar una expresión para renderizar o no un elemento de esta manera:

```html
<div class="errors-container" v-if="errors.length !== 0">
```

Esto renderizaría el elemento dependiendo de si evalúe a true o a false. Aunque contemos con la posibilidad, es buena práctica que todas estas evaluaciones nos las llevemos a nuestra parte JavaScript. De esta manera separamos correctamente lo que tiene que ver con la vista de lo que tiene que ver con la lógica. Seguimos respetando los niveles de responsabilidad.

Si no tenéis mas remedio que usar una expresión de este estilo, hay que tener en cuenta que ni los flujos ni las iteraciones de JavaScript funcionan en ellos. La siguiente interpolación de una expresión daría un error:

```html
{{ if (ok) { return message } }}
```

## Las directivas

Las directivas son atributos personalizados por VueJS que permiten extender el comportamiento por defecto de un elemento HTML en concreto. Para diferenciar un atributo personalizado de los estándar, VueJS añade un prefijo `v-` a todas sus directivas.

Por ejemplo, y como ya hemos ido viendo, contamos con directivas como esta:

```html
<input id="username" type="text" v-model="user.name" />
```

`v-model` nos permite hacer un doble data-binding sobre una variable específica. En este caso user.name.

Una directiva puede tener o no argumentos dependiendo de su funcionalidad. Por ejemplo, una directiva con argumento sería la siguiente:

```html
<a v-bind:href="urlPasswordChange" target="_blank">
  ¿Has olvidado tu contraseña?
</a>
```

Lo que hace `v-bind` con el argumento `href` es enlazar el contenido de `urlPasswordChange` como url del elemento `a`.

Las directivas son un concepto bastante avanzado de este tipo de frameworks. Hay que tener en cuenta en todo momento que la ventaja de trabajar con estos sistemas es que el comportamiento es reactivo. Esto quiere decir, que si el modelo al que se encuentra enlazado un elemento HTML se ve modificado, el propio motor de plantillas se encargará de renderizar el nuevo elemento sin que nosotros tengamos que hacer nada.

### Directivas modificadoras

Una directiva, en particular, puede tener más de un uso específico. Podemos indicar el uso específico accediendo a la propiedad que necesitamos incluir en el elemento. Un caso muy común es cuando registramos un evento determinado en un elemento y queremos evitar el evento por defecto que tiene el elemento. Por ejemplo, en un evento `submit` querremos evitar que nos haga un post contra el servidor para así realizar la lógica que necesitemos en JavaScript. Para hacer esto, lo haríamos así:

```html
<form class="login" v-on:submit.prevent="onLogin">
```

Dentro de la API de VueJS contamos con todos estos modificadores.

### Atajo para la directiva v-bind o v-on

Una de las directivas más utilizadas es `v-bind`, lo que nos permite esta directiva es enlazar una variable a un elemento ya sea un texto o un atributo. Cuando lo usamos para enlazar con un atributo como argumento, podríamos usar el método reducido y el comportamiento sería el mismo. Para usar el atajo ahora debemos usar `:`. Para ver un ejemplo veremos cuando enlazamos una url a un elemento a. Podemos hacerlo de esta manera:

```html
<button type="submit" v-bind:disabled="isFormEmpty">Entrar</button>
```

O de esta otra que queda más reducido:

```html
<button type="submit" :disabled="isFormEmpty">Entrar</button>
```

Los dos generarían el mismo HTML:

```html
<button type="submit" disabled="disabled">Entrar</button>
```

En el caso de registrar un evento, tenemos algo parecido. No hace falta que indiquemos `v-on` sino que podemos indicarlo por medio de una arroba `@` de esta manera:

```html
<form class="login" @submit.prevent="onLogin">
```

El comportamiento sería el mismo que con v-on.

En este post hemos explicado algunas de [las directivas que hay, para entender el concepto, pero la API cuenta con todas estas](https://vuejs.org/v2/api/#Directives).

## Las filtros

Cuando interpolamos una variable dentro de nuestro HTML, puede que no se pinte todo lo bonito y claro que a nosotros nos gustaría. No es lo mismo almacenar un valor monetario que mostrarlo por pantalla. Cuando yo juego con 2000 euros. Dentro de mi variable, el valor será 2000 de tipo number, pero cuando lo muestro en pantalla, el valor debería ser 2.000,00 € de tipo string.

Hacer esta conversión en JavaScript, rompería con su responsabilidad específica dentro de una web, no solo estamos haciendo que trabaje con lógica, sino que la conversión implica hacer que trabaje en cómo presenta los datos por pantalla.

Para evitar esto, se han inventado los filtros. Un filtro modifica una variable en tiempo de renderizado a la hora de interpolar la cadena. Para cambiar el formato de una variable tenemos que incluir el carácter `|` y el filtro que queremos usar como transformación.

Este sería un caso donde convertimos el texto de la variable en mayúsculas:

```html
<h1>Bienvenido {{ user.name | uppercase }}</h1>
```

Los filtros se comportan como una tubería de transformación por lo que yo puedo concatenar todos los filtros que necesite de esta manera:

```html
<h1>Bienvenido {{ user.name | filter1 | filter2 | filterN }}</h1>
```

En VueJS v1 se contaba dentro del core con una serie de filtros por defecto que en VueJS v2 han quitado para agilizar su tamaño. [Para incluir estos filtros podemos insertar la siguiente librería que extiende el framework](https://github.com/freearhey/vue2-filters).

## Todo junto

Los ejemplos que hemos ido viendo forman parte del ejemplo que hemos desarrollado para este post. Lo que hemos hecho es desarrollar una plantilla en VueJS sobre una vista típica de login. El marcado es el siguiente:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Example templates</title>
</head>

<body>
    <div id="app" v-cloak>
        <h1>Bienvenido {{ user.name | uppercase }}</h1>

        <div class="login-errors-container" v-if="errors.length !== 0">
            <ol class="login-errors">
                <li v-for="error in errors"> {{ error }}</li>
            </ol>
        </div>

        <form class="login" v-on:submit.prevent="onLogin">
            <div class="login-field">
                <label for="username">Nombre de usuario</label>
                <input id="username" type="text" v-model="user.name" />
            </div>

            <div class="login-field">
                <label for="password">Contraseña</label>
                <input id="password" type="password" v-model="user.password" />
            </div>

            <div class="login-field">
                <button type="submit" v-bind:disabled="isFormEmpty">Entrar</button>
            </div>
        </form>

        <a v-bind:href="urlPasswordChange" target="_blank">
            ¿Has olvidado tu contraseña?
        </a>
    </div>

    <script src="node_modules/vue/dist/vue.js"></script>
    <script src="app.js"></script>
</body>

</html>
```

Creo que con lo visto en el post, todos los elemento se explican y no es necesario que profundicemos.

Os dejo también la instancia de VueJS donde se ve la lógica creada para la vista:

```javascript
const app = new Vue({
    el: '#app',
    data: {
        user: { name: null, password: null }, 
        urlPasswordChange: 'http://localhost:8080',
        errors: []
    },
    computed: {
         isFormEmpty: function () {
             return !(this.user.name && this.user.password);
         }
    },
    methods: {
        onLogin: function () {
            this.errors = [];

            if (this.user.name.length < 6) {
                this.errors.push('El nombre de usuario tiene que tener al menos 6 caracteres');
            }

            if (this.user.password.length < 6) {
                this.errors.push('La contraseña tiene que tener al menos 6 caracteres');
            }
        }
    },
    filters: {
        uppercase: function (data) {
            return data && data.toUpperCase();
        }
    }
});
```

## Conclusión

Parece una tontería, pero saber desarrollar plantillas en VueJS nos ayuda mucho en nuestro camino a comprender el framework. No olvidemos que estamos aprendiendo sobre un framework progresivo y que, quizá, existan desarrolladores que con esto tengan más que suficiente para desarrollar aplicaciones con VueJS y que no necesiten muchas más funcionalidades.

Los que hayan usado en su vida profesional motores de plantillas, habrán visto que el sistema es el mismo de siempre. Esto es otra de las ventajas de VueJS: el framework intenta ser todo lo práctico que puede y no intentar reinventar la rueda si es posible. Si algo como las plantillas HTML ha funcionado ya en otros sistemas, por qué no aprovecharnos de ellos.

Si a alguno de vosotros, al leer este post sobre plantillas, el sistema de directivas y filtros presentado se le queda un poco pequeño, es bueno recordar que VueJS permite su extensión por medio de plugins y que crear nuevas directivas y nuevos filtros estará disponible para nosotros. Hablaremos en El Abismo más adelante de cómo hacer esto.

Por el momento, nos quedamos aquí.

Nos leemos :\)

