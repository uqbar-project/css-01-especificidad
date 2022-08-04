## Video explicativo

Tenés la explicación de este ejemplo [en este link](https://youtu.be/IV7n-lJeGGI).

## Cascada de estilos

Los archivos css permiten definir reglas [en cascada](https://developer.mozilla.org/es/docs/Learn/CSS/Building_blocks/Cascada_y_herencia) que se aplican y combinan sobre los elementos de un documento HTML. Ahora bien, para entender de qué manera una regla puede pisar la definición de otra, tenemos que conocer el concepto **especificidad**.

## Especificidad

La [especificidad](https://developer.mozilla.org/es/docs/Web/CSS/Especificidad) en CSS indica de qué manera cada elemento adopta el estilo que le corresponde según las siguientes prioridades, partiendo de la más baja hasta la más alta:

- selectores por elemento o _tag HTML_: `h1`, `div`, `p`, etc.
- selectores por clase/atributo, lo que en html se define con el atributo `class`
- selectores por identificador (atención que en una página HTML bien formada solo puede haber un identificador por elemento)
- estilos inline

![specificity calculation base](./images/specificity-calculationbase.png)

Pueden ver el artículo de [Andy Clarke](https://stuffandnonsense.co.uk/archives/css_specificity_wars.html), que propone una metáfora basada en Star Wars para explicar las prioridades de cada especificidad.

## Nuestro ejemplo

### Div solo

En el primer caso tenemos:

```html
<div>1</div>
```

Nuestro archivo de estilos es

```css
div {
  /* Fuente */
  font-family: Ubuntu;
  font-size: 20px;

  /* Tamaño */
  width: 100px;
  height: 100px;

  /* Colores */
  background-color: blue;
  color: white;
  
  /* Alineación */
  text-align: center;

  border-radius: 20px;
}
```

La especificidad de esta definición es `0,0,0,1`:

| inline | identificador | clase | tag |
| ------ | ------ | ----- | ------ |
|0|0|0|1|
| | | | div |

Por eso vemos un rectángulo de color azul.

### Div cajita

Veamos ahora el siguiente ejemplo:

```
<div class="cajita">2</div>
```

Aquí tenemos especificidades `0,0,1,1` (`0,0,1,0` de `.cajita` + `0,0,0,1` de `div`):

| inline | identificador | clase | tag |
| ------ | ------ | ----- | ------ |
|0|0|1|1|
| | | .cajita | div |

Le agregamos un color de fondo diferente para pisar la definición del estilo para el elemento `div`:

```css
.cajita {
  background-color: red;
}
```

A continuación te mostramos cómo abrir un navegador el modo Desarrollo con F12, y luego presionando el botón sobre la cajita podés ver qué propiedades tenés activadas y cuáles se inactivan porque tienen menos prioridad (incluso podés modificar esa prioridad para ver los cambios en el navegador):

![Especificidad en Firefox](./images/herramientasDesarrollo.gif)

### Tercera cajita: class

En esta definición

```html
<div class="cajita violet">
  <span>3</span>
</div>
```

aparecen dos clases, tenemos entonces especificidad general `0,0,2,1`, y aquí la clase `violet` pisa la definición de `div`: la prioridad la determina el orden en el que se define dentro del archivo de estilos css. Como `violet` está definido después que `cajita`, eso determina la prioridad.

| inline | identificador | clase | tag |
| ------ | ------ | ----- | ------ |
|0|0|1|1|
| | | .violet -> "violeta", .cajita -> "rojo" | div -> "azul" |

por eso vemos el rectángulo en violeta.

### Cuarta cajita: id

```html
<div id="yellow" class="cajita violet">
  <span>4</span>
</div>
```

La especificidad general es `0,1,2,1`, en particular para definir el color de fondo podemos ver que

| inline | identificador | clase | tag |
| ------ | ------ | ----- | ------ |
|0|1|1|1|
| | id="yellow" -> amarillo | .violet -> "violeta", .cajita -> "rojo" | div -> "azul" |

![especificidad con id yellow](./images/especificidadPorId.png)

> **Nota**: dado que un html válido solo debe contener un único `id`, no recomendamos generar estilos que dependan de un identificador a menos que explícitamente debamos definir un estilo particular en un lugar muy específico.

### Quinta cajita: inline style

```html
<div style="background-color: green; color: white;" id="yellow" class="cajita violet">
  <span>5</span>
</div>
```

Aquí tenemos una especificidad general `1,1,2,1`, para el color de fondo aplican estas definiciones

| inline | identificador | clase | tag |
| ------ | ------ | ----- | ------ |
|1|1|1|1|
| inline -> verde | id="yellow" -> amarillo | .violet -> "violeta", .cajita -> "rojo" | div -> "azul" |

Podemos ver cómo se van pisando los estilos en el navegador:

![especificidad inline](./images/especificidadInline.png)

De la misma manera que desaconsejamos el uso de `id` para generar estilos, el estilo inline [tiene muchas desventajas](https://stackoverflow.com/questions/2612483/whats-so-bad-about-in-line-css):

- los cambios se van propagando por medio de copy/paste, por lo que cada modificación requiere editar manualmente uno por uno cada estilo
- al no centralizar el look & feel, es muy difícil generar requerimientos nuevos como un "dark mode" (modo claro/oscuro)
- los navegadores descargan los archivos de estilos una vez y lo guardan en una cache, por lo que si la siguiente página html usa la misma hoja de estilos el archivo ya estará descargado (mejora en performance)

### VIP: Very !important anti-Pattern

Otra "herejía" consiste en utilizar la declaración `!important` que echa por tierra las prioridades establecidas por la especificidad. Hagamos ahora este cambio en nuestro css:

```css
.cajita {
  background-color: red !important;
}
```

¿Qué sucede con las cajitas?

![especificidad con important](./images/especificidadImportant.png)

Cuatro de las cajas se colorean con rojo, porque el important deja como prioritaria la definición de la clase "cajita". Esto puede iniciar una guerra de importants, donde alguien decide colocar el `!important` para el id blue:

```css
#yellow {
  background-color: yellow !important;
}
```

y ahora tenemos las últimas dos cajas pintadas de amarillo, porque cuando se cruzan dos `!important` prevalece el criterio de especificidad, y recordemos que `id` > `class`. En definitiva, cuando agregamos un `!important` rompemos el orden natural de las cosas y esto es muy probable que nos fuerce a agregarlo en más de un lugar, lo que nos recuerda a

![css important](./images/cssImportant.gif)

> Por este motivo desaconsejamos fuertemente el uso de la cláusula `!important` como técnica para forzar el cambio de prioridades en la definición de estilos.

### BONUS 1: Combinación de especificidades

Un caso adicional puede ser que el css combine un tag + una clase, por ejemplo:

```css
div.violet {
  background-color: peru;
}

/* definición previa */
.violet {
  background-color: darkviolet;
}
```

Ahora veremos que el div que tiene violet aparece con un color marrón por la definición `div.violet` (0, 0, 1, 1) que tiene **mayor especificidad** que `violet` (0, 0, 1, 0)

### BONUS 2: Múltiples clases

También podemos definir en nuestro css precedencia si varias clases se combinan en un elemento:

```css
.cajita.violet {
  background-color: darkturquoise;
}
```

volviendo nuestra tercera cajita a:

```html
<div class="cajita violet">3</div>
```

Ahora veremos en violeta la tercera cajita, ya que ambas clases `cajita` y `violet` tienen prioridad sobre la `violet`:

| inline | identificador | clase | tag |
| ------ | ------ | ----- | ------ |
|0|0|2|1|
| | | .cajita + .violet -> "turquesa" > .violet y .cajita por separado | div -> "azul" |

### Herencia de estilos

Para algunas propiedades es posible definir reglas que se heredan de un elemento padre a otro hijo: colores, font, entre otras. Si agregamos estas propiedades para el selector `div`:

```css
div {
  /* Fuente */
  font-family: Ubuntu;
  font-size: 20px;
  ...
}
```

Esto se hereda a los elementos hijos del div en el html, en nuestro caso el span:

```html
  <div>
    <span>1</span>
  </div>
```

Veremos que todas las cajas ahora se visualizan con un font Ubuntu color blanco de tamaño 20 píxeles.

![herencia de propiedades](./images/herenciaPropiedades.png)

## Material adicional

- [Jueguito para practicar especificidad](https://flukeout.github.io/)
- [CSS Specificity: Things you should know](https://www.smashingmagazine.com/2007/07/css-specificity-things-you-should-know/)
- [Specifics on CSS Specificity](https://css-tricks.com/specifics-on-css-specificity/)
- [A Specificity Battle](https://css-tricks.com/a-specificity-battle/)
- [CSS Specificity - When several rules collide](https://marksheet.io/css-priority.html)
- [Specificity Calculator](https://specificity.keegan.st/)
- [Grafo de especificidad](https://jonassebastianohlsson.com/specificity-graph/), podés copiar y pegar tu archivo css y te muestra gráficamente la definición de especificidades

