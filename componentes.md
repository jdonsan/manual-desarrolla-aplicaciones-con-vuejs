#Capítulo 4. Creando componentes

Estamos en pleno boom de los frameworks y librerías front orientados a componentes. Parecen ser la mejor forma de modularizar nuestro código y de conseguir piezas reutilizables y mantenibles con un alto nivel de cohesión interno y un bajo acoplamiento entre piezas.

VueJS no iba a ser menos y basa su funcionamiento en el aislamiento de estados y comportamientos en pequeños componentes que se encarguen de llevar a cabo el ciclo de vida entero de una mínima parte de la UI que estamos creando.

Como en casi todos los casos - quizá a excepción de Polymer - sufre de todo aquello bueno y malo de estas soluciones. Para explicar cómo trabajar con componentes en VueJS lo mejor será desarrollar un ejemplo. En este ejemplo vamos a explicar qué son las propiedades, qué son los eventos personalizados y qué son 'slots'. ¿Empezamos?


## El ejemplo

El desarrollo que vamos a hacer es un pequeño 'marketplace' para la venta de cursos online. El usuario va a poder indicar el tiempo en meses que desea tener disponible la plataforma en cada uno de los cursos.

## Creando la instancia

Para ello lo que vamos a crear es un primer componente llamado course. Para hacer esto, tenemos que registrar un nuevo componente dentro del framework de la siguiente manera:

 

 
Con esto ya podremos hacer uso de él en cualquier plantilla en el que necesitemos un ítem curso dentro de nuestra app. Hemos incluido unos datos de inicialización del componente. En este caso datos de la cabecera. Cómo puedes apreciar, en un componente, data se define con una función y no con un objeto.

## Incluyendo propiedades

Un componente no deja de ser una caja negra del cual no sabemos qué HTML se va a renderizar, ni tampoco sabemos como se comporta su estado interno. Este sistema de cajas negras es bastante parecido al comportamiento de una función pura en JavaScript.

Una función, para poder invocarse, debe incluir una serie de parámetros. Estos parámetros son propiedades que nosotros definimos al crear una función. Por ejemplo, puedo tener una función con esta cabecera:

```javascript
function createCourse(title, subtitle, description) {
    ...
}
```

Si yo ahora quiero hacer uso de esta función, simplemente lo haría así:

```javascript
createCourse(
    'Curso JavaScript', 
    'Curso Avanzado', 
    'Esto es un nuevo curso para aprender'
);
```

Dados estos parámetros, yo espero que la función me devuelve lo que me promete: un curso.

Pues en VueJS y su sistema de componentes pasa lo mismo. Dentro de un componentes podemos definir propiedades de entrada. Lo hacemos de la siguiente manera:

 

 
Estamos indicando, dentro del atributo `props` del objeto `options`, las propiedades de entrada que queremos que tenga nuestro componente, en nuestro caso son 3: `title`, `subitle` y `description`, al igual que en la función.

Estas propiedades, ahora, pueden ser usadas en su template. Es buena práctica dentro de cualquier componente que indiquemos estas propiedades y que les pongamos validadores.

En nuestro caso, lo único que estamos diciendo es que las tres propiedades sean de tipo String y que sean obligatorias para renderizar correctamente el componente. Si alguien no usase nuestro componente de esta forma, la consola nos mostrará un warning en tiempo de debug.

Ahora ya podemos usar, en nuestro HTML, nuestro componente e indicar sus propiedades de entrada de esta forma:

 

 
Como vemos, es igual que en el caso de la función.

Hay que tener en cuenta que las propiedades son datos que se propagan en una sola dirección, es decir de padres a hijos. Si yo modifico una propiedad dentro de un componente hijo, ese cambio no se propagará hacia el padre y por tanto no provocará ningún tipo de reacción por parte del sistema.

Aunque hablaremos más de props en el futuro, aquí tienes más documentación.

## Personalizando eventos

Hemos visto cómo el componente padre consigue comunicar datos a un componente hijo por medio de las propiedades, pero ¿Qué ocurre cuando un hijo necesita informar de ciertos datos o acciones que están ocurriendo en su interior? ¿Cómo sería el return de un componente en VueJS si fuese una función JavaScript?

Bueno, la opción por la que ha optado VueJS, para comunicar datos entre componentes hijos con componentes padres, ha sido por medio de eventos personalizados. Un componente hijo es capaz de emitir eventos cuando ocurre algo en su interior.

Tenemos que pensar en esta emisión de eventos como en una emisora de radio. Las emisoras emiten información en una frecuencia sin saber qué receptores serán los que reciban la señal. Es una buena forma de escalar y emitir sin preocuparte del número de oyentes.

En los componentes de VueJS pasa lo mismo. Un componente emite eventos y otros componentes padre tienen la posibilidad de escucharlo o no. Es una buena forma de desacoplar componentes. El sistema de comunicación es este:

props-events

En nuestro caso, el componente va contar con un pequeño `input` y un botón para añadir cursos. Lo que ocurrirá es que el componente `course` emitirá un evento de tipo `add` con un objeto que contiene los datos del curso y los meses que se quiere cursar:

 

 
Lo que hacemos es usar el método del componente llamado `$emit` e indicar un 'tag' para el evento personalizado, en este caso `add`.

Ahora, si queremos registrar una función cuando hacemos uso del componente, lo haríamos de la siguiente manera:

 

 
Hemos registrado un evento de tipo `@add` que ejecutará la función `addToCart` cada vez que el componente emita un evento `add`.

Aunque hablaremos más de eventos en el futuro, aquí tienes más documentación.

### Extendiendo el componente

Una vez que tenemos esto, hemos conseguido definir tanto propiedades de entrada, como los eventos que emite mi componente. Podríamos decir que tenemos un componente curso base.

Ahora bien, me gustaría poder definir ciertos estados y comportamientos dependiendo del tipo de curso que quiero mostrar. Me gustaría que los cursos de JavaScript tuviesen un estilo y los de CSS otro.

Para hacer esto, podemos extender el componente base `course` y crear dos nuevos componentes a partir de este que se llamen `course-js` y `course-css`. Para hacer esto en VueJS, tenemos que hacer siguiente:

 

 
Lo que hemos hecho es sacar todo el constructor a un objeto llamado course. Este objeto contiene todo lo que nosotros queremos que el componente tenga como base. Lo siguiente es definir dos componentes nuevos llamados `course-js` y `course-css` donde indicamos en el parámetro `mixins` que queremos que hereden.

Por último,  indicamos aquellos datos que queremos sobreescribir. Nada más. VueJS se encargará de componer el constructor a nuestro gusto y de generar los componentes que necesitamos. De esta forma podemos reutilizar código y componentes. Ahora podemos declarar nuestros componentes dentro del HTML de la siguiente forma:

 

 
Ambos componentes tienen la misma firma, pero internamente se comportan de diferente manera.

> En el futuro hablaremos más de mixins. Si necesitas saber más sobre ello, aquí puedes.

## Refactorizando el componente

Después de crear dos componentes más específicos, se me viene a la cabeza que ese template que estamos usando en course, presenta una estructura bastante compleja. Sería buena idea que refactorizásemos esa plantilla en trozos más pequeños y especializados que nos permiten centrarnos mejor en el propósito y la responsabilidad de cada uno de ellos.

Sin embargo, si vemos los componentes en los que podríamos dividir ese template, nos damos cuenta que por ahora, no nos gustaría crear componentes globales sobre estos elementos. Nos gustaría poder dividir el código pero sin que se encontrase en un contexto global. Esto en VueJS es posible.

En VueJS contamos con la posibilidad de tener componentes locales. Es decir, componentes que simplemente son accesibles desde el contexto de un componente padre y no de otros elementos.

Esto puede ser una buena forma de modularizar componentes grandes en partes más pequeñas, pero que no tienen sentido que se encuentren en un contexto global ya sea porque su nombre pueda chocar con el de otros, ya sea porque no queremos que otros desarrolladores hagan un uso inadecuado de ellos.

Lo que vamos a hacer es coger el siguiente template:

 

 
Y lo vamos a convertir en los siguiente:

 

 
Hemos sacado el header, el content y el footer en diferentes componentes a los que vamos pasando sus diferentes parámetros.

Los constructores de estos componentes los definimos de esta manera:

 

 
Estos constructores podrían ser usados de forma global, y no estaría mal usado. Sin embargo, para el ejemplo, vamos a registrarlos de forma local en el componentes course de esta manera:

 

 
Todos los componentes cuentan con este atributo components para que registremos constructores y puedan ser usado.

Personalmente, creo que pocas veces vamos a hacer uso de un registro local, pero que contemos con ello, creo que es una buena decisión de diseño y nos permite encapsular mucho mejor a la par que modularizar componentes.

## Creando un componente contenedor

Una vez que hemos refactorizado nuestro componente course, vamos a crear un nuevo componente que nos permita pintar internamente estos cursos. Dentro de VueJS podemos crear componentes que tengan internamente contenido del cual no tenemos control.

Estos componentes pueden ser los típicos componentes layout, donde creamos contenedores, views, grids o tablas donde no se sabe el contenido interno. En VueJS esto se puede hacer gracias al elemento slot. Nuestro componente lo único que va a hacer es incluir un div con una clase que soporte el estilo flex para que los elementos se pinten alineados.

Es este:

 

 
Lo que hacemos es definir un 'template' bastante simple donde se va a encapsular HTML dentro de slot.  Dentro de un componente podemos indicar todos los slot que necesitemos. Simplemente les tendremos que indicar un nombre para que VueJS sepa diferenciarlos.

Ahora podemos declararlo de esta manera:

 

 
Dentro de marketplace definimos nuestro listado de cursos.

Fijaros también en el detalle de que no estamos indicando ni courseni `course-js` ni `course-css`. Hemos indicado la etiqueta component que no se encuentra definida en ninguno de nuestros ficheros.

Esto es porque component es una etiqueta de VueJS que en combinación con la directiva `:is` podemos cargar componentes de manera dinámica. Como yo no se que tipo de curso va haber en mi listado, necesito pintar el componente dependiendo de lo que me dice la variable del modelo `course.type`.

Para saber más sobre slots, tenemos esta parte de la documentación.

## Todo junto

Para ver todo el ejemplo junto, contamos con este código:

 

 
 

 
 

 
## Conclusión

Hemos explicado todo lo que tiene que ver con el corazón de la librería. Controlando y sabiendo cómo funcionan los componentes en VueJS, tendremos mucho recorrido ganado en poder aplicaciones del mundo real.

Las propiedad, los eventos y los slots son una buena forma para diseñar componentes de una forma versátil y dinámica. Diseñar bien nuestros componentes será un primer paso a tener en cuenta si queremos que nuestra arquitectura triunfe, pero sí es importante tener en cuenta que posibilidades nos da VueJS para que hacer este diseño más robusto y constante.

No te preocupes si el ejemplo te parece bastante enrevesado o sin sentido. Por culpa de tener que explicar todos los casos posibles que se pueden dar en un componente, hemos tenido que complicarlo todo. En el futuro veremos que muchas de estas decisiones que hemos tomado, como la herencia o el registro local, se podría haber solucionado con el paso de un nuevo parámetros y el registro global.

En próximos posts, seguiremos hablando sobre componentes y seguiremos entendiendo los mejor.

Nos leemos :)