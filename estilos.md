# Capítulo 3. Enlazando clases y estilos

Una de las cosas que más me gustaban cuando usaba jQuery era la posibilidad de incluir o quitar clases desde mi JavaScript en cualquier momento. Con dos simples métodos - `addClass` y `removeClass` - podía hacer de todo.

Me daba una versatilidad y tal toma de decisión sobre el cómo, cuándo y por qué incluir o eliminar una clase a un elemento HTML, que el mecanismo que suelen proponer los frameworks modernos, no me acababa de convencer o de darme esa comodidad que yo ya tenía.

En su día HandlebarsJS no consiguió convencerme del todo, AngularJS contiene tantas funcionalidades que el proceso se vuelve demasiado complejo y verboso. En VueJS nos pasa parecido, pero por lo menos la sintaxis es clara y sencilla.

El post de hoy vamos a dedicarlo a explicar cómo podemos incluir o quitar clases o estilos dependiendo del estado de nuestra aplicación, un breve post con el que cerraremos la introducción a la serie:

## Enlazando clases

Para enlazar una variable a una clase, hacemos uso de la directiva `v-bind:class` o su alternativa corta `:class`. Con esta declaración, podemos guardar el nombre de clases en variables y jugar con ello a nuestro antojo de manera reactiva.

Por ejemplo, yo podría hacer esto:

 ```html
 <div :class="player1.winner">
```

De esta manera, he enlazado la variable `winner` de mi modelo `player1` a la directiva `:class`. Lo que VueJS va a intentar es coger el contenido de winner y lo va a insertar como una clase.

Esto nos da poca versatilidad porque nos hace acoplarnos demasiado a una sola variable y no nos permite incluir más en un elemento. Sin embargo, VueJS acepta otras formas de enlazar modelos para conseguir funcionamientos más flexibles.

Dentro de la directiva `v-bind:class` podemos enlazar tanto un objeto como un array. Veamos qué nos aporta cada caso:

### Enlazando un objeto

Podemos enlazar un objeto en un elemento para indicar si una clase se tiene que renderizar en el HTML o no. Lo hacemos de la siguiente manera:

```html
 <div :class="{ winner: player1.winner }">
``` 
 
Cuando la variable `player1.winner` contenga un `true` la clase `winner` se renderizará. Cuando contenga `false` no se incluirá.  De esta manera puedo poner el toggle de las funciones que quiera. Este objeto, me lo puedo llevar a mi parte JavaScript y jugar con él como necesite.

### Enlazando un array

Puede darse el caso también, que no solo necesite hacer un toggle de clases. Puede que quiera indicar un listado de clases enlazadas. Yo podría hacer lo siguiente:

 ```html
 <div :class="['box', winner]">
```
 
En este caso lo que quiero es que el elemento `div` tenga la clase `box` y lo que contenga internamente la variable `winner`. Con esto se puede jugar bastante y crear híbridos como el siguiente:

 ```html
 <div :class="['box', { winner: player1.winner }]">
```
 
En este caso, hago que `winner` se incluya o no dependiendo del valor de `player1.winner`.

Al igual que pasaba con los objetos, esta estructura de datos puede ser almacenada en JS y ser enlazada directamente.

Un pequeño apunte a tener en cuenta al enlazar en un componente

Podemos enlazar variables a la definición de un componente que se encuentre en nuestra plantilla.

Teniendo la definición del siguiente componente:

 ```javascript
 Vue.component('pokemon', {
    template: `
        <div class="pokemon">
            <div class="pokemon-head"></div>
            <div class="pokemon-body"></div>
            <div class="pokemon-feet"></div>
        </div>
    `
});
```
 
Yo podría hacer esto en el template al usarlo:

 ```html
 <pokemon :class="player1.pokemon.name"></pokemon>
```
 
El resultado del HTML generado, en este caso, sería el siguiente:

 ```html
 <div class="pokemon pikachu">
    <div class="pokemon-head"></div>
    <div class="pokemon-body"></div>
    <div class="pokemon-feet"></div>
</div>
```
 
VueJS primero coloca las clases que tenía definido en su plantillas interna y luego incluye las nuestras. De esta manera podremos pisar los estilos que deseemos. Es una buena forma de extender el componente a nivel de estilos.

## Enlazando estilos

También podemos enlazar estilos directamente en un elemento. En vez de usar la directiva `v-bind:class`, tenemos que usar la directiva `v-bind:style`. Haríamos esto:

```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
 
Hemos incluido un objeto que lleva todas las propiedades de CSS que queramos. Este objeto también podría estar en nuestro JS para jugar con él.

### Como un array

Puede que necesitemos extender estilos básicos de manera inline. VueJS ya ha pensado en esto:

```html
<div v-bind:style="[baseStyles, overridingStyles]">
```
 
Me detengo menos en esta directiva porque creo que es mala práctica incluir estilos en línea, simplemente es bueno que sepamos que existe la posibilidad por si en algún caso en concreto no quedase más remedio de hacer uso de esta técnica.

### A tener en cuenta

Cuando incluímos un elemento CSS que suele llevar prefijos debido a que no está estandarizado todavía (imaginemos en transform por ejemplo), no debemos preocuparnos, pues VueJS lo tiene en cuenta y el mismo añadirá todos los prefijos necesarios

## Todo junto

Los ejemplos que hemos ido viendo en el post son sacado del siguiente ejemplo:

Hemos creado un pequeño juego que te permite enfrentar en un combate a tu pokemon favorito contra otros. La idea está en elegir dos pokemons y ver quién de los dos ganaría en un combate. Una tontuna que nos sirve para ver mejor cómo enlazar clases en VueJS.

El ejemplo consta de estos tres ficheros:

 ```css
.box { display: inline-block; padding: 1rem; margin: 1rem; }
.box.winner { background: green; }

.pokemon { width: 3rem; display: inline-block; margin: 1rem; }
.pokemon-head, .pokemon-body { height: 3rem; }
.pokemon-feet { height: 1rem; }

.pokemon.bulvasaur .pokemon-head { background: #ff6a62; }
.pokemon.bulvasaur .pokemon-body { background: #62d5b4; }
.pokemon.bulvasaur .pokemon-feet { background: #317373; }

.pokemon.squirtle .pokemon-head,
.pokemon.squirtle .pokemon-feet { background: #8bc5cd; }
.pokemon.squirtle .pokemon-body { background: #ffe69c; }

.pokemon.charmander { background: #de5239; }

.pokemon.pikachu { background: #f6e652; }
```
Estas son las clases que dan forma a nuestros cuatro pokemons y las que vamos a ir enlazando de manera dinámica según lo que el usuario haya elegido.

 ```javascript
 Vue.component('pokemon', {
    template: `
        <div class="pokemon">
            <div class="pokemon-head"></div>
            <div class="pokemon-body"></div>
            <div class="pokemon-feet"></div>
        </div>
    `
});

const app = new Vue({
    el: '#app',
    data: {
        player1: { pokemon: {}, winner: false },
        player2: { pokemon: {}, winner: false },
        pokemons: [
            { id: 0, name: 'pikachu', type: 'electro' },
            { id: 1, name: 'bulvasaur', type: 'planta' },
            { id: 2, name: 'squirtle', type: 'agua' },
            { id: 3, name: 'charmander', type: 'fuego' }
        ],
        results: [
            [0, 2, 1, 0],
            [1, 0, 2, 2],
            [2, 1, 0, 1],
            [0, 1, 2, 0],
        ]
    },
    methods: {
        fight: function () {
            const result = this.results[this.player1.pokemon.id][this.player2.pokemon.id];
            const selectWinner = [
                () => { this.player1.winner = true; this.player2.winner = true; },    // empate
                () => { this.player1.winner = true; this.player2.winner = false; },   // gana jugador 1
                () => { this.player1.winner = false; this.player2.winner = true; }    // gana jugador 2
            ];

            selectWinner[result]();
        },
        resetWinner: function () {
            this.player1.winner = false;
            this.player2.winner = false;
        }
    }
});
```
El código anterior define un componente pokemon con diferentes `divs` para simular la anatomía 'estilo lego' de un pokemon.

La instancia de VueJS define una aplicación que simula la batalla. Lo que hacemos es definir dos jugadores (líneas 14 y 15), un listado de pokemons (líneas de la 16 a la 20) y una tabla de resultados posibles donde x​ e y indican quién ganaría entre los pokemons seleccionados por ambos jugadores (líneas 22 a la 26).

Hemos definido dos métodos para simular el combate. El método `fight` obtiene el `id` de ambos jugadores y busca la posición en la tabla de resultados. Dependiendo del resultado dado, se indica el jugador que ha ganado. El método `resetWinner` nos permite reiniciar la partida para empezar una nueva.

La plantilla que permite mostrar todo esta lógica por pantalla es el siguiente:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Example classes</title>

    <link rel="stylesheet" href="app.css">
</head>

<body>
    <div id="app">
        <div class="actions-container">
            <button @click="fight">Luchar</button>
        </div>

        <!-- Casilla del jugador 1 -->
        <div :class="['box', { winner: player1.winner }]">
            <select v-model="player1.pokemon" @change="resetWinner">
                <option v-for="pokemon in pokemons" v-bind:value="pokemon">{{ pokemon.name }}</option>
            </select>
            <pokemon :class="player1.pokemon.name"></pokemon>
        </div>

        <label>VS</label>

        <!-- Casilla del jugador 2 -->
        <div :class="['box', { winner: player2.winner }]">
            <pokemon :class="player2.pokemon.name"></pokemon>
            <select v-model="player2.pokemon" @change="resetWinner">
                <option v-for="pokemon in pokemons" v-bind:value="pokemon">{{ pokemon.name }}</option>
            </select>
        </div>
    </div>

    <script src="node_modules/vue/dist/vue.js"></script>
    <script src="app.js"></script>
</body>
</html>
```
 
Hemos definido dos contenedores para todo lo relativo a cada uno de los jugadores (esto en el futuro serán componentes, pero para que el ejemplo quede claro, hemos preferido dejarlo así).

En la línea 18 podemos ver cómo usamos un enlace de clases por medio de array y de objeto. Combinar ambos métodos nos da mucha versatilidad. Como yo necesito indicar dos clases, uso el array. La primera clase es una fija. No necesito dinamismo en tiempo de JS con lo que indico directamente como un string la clase `box`. La segunda clase está enlazada al modelo del jugador. La clase `winner` se activará cuando tengamos un ganador de la partida.

El otro elemento donde tenemos enlace de clases es en la línea 22. En este caso estoy enlazando una clase dinámica a un componente. Como vimos anteriormente, esto es posible y lo que nos va a permitir es pintar los colores del pokémon elegido. Ese modelo variará dependiendo de lo seleccionado en el elemento select.

[Si queréis ver el ejemplo funcionando podéis hacerlo en este jsfiddle](https://jsfiddle.net/pge2qrd9/1/).

## Conclusión

Con el post de hoy lo que hemos hecho es aumentar nuestros conocimientos en la sintaxis que podemos usar para las plantillas de VueJS. Nada nuevo ni revolucionario. Nada que otras alternativas no nos permitan.

Conocerlo es nuestra obligación para ser buenos desarrolladores de VueJS, así que aunque este post parezca un trámite, es necesario conocerlo y asimilarlo.

En el próximo post de la serie, elevaremos el nivel de dificultad y nos centraremos en la creación de componentes, la piedra angular de este framework progresivo.

Por el momento es todo.

Nos leemos :)