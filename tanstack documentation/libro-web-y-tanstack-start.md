# De la Web a TanStack Start: fundamentos para pensar aplicaciones con criterio

## Introducción

Aprender desarrollo web moderno suele ser una experiencia extraña. La cantidad de material disponible es inmensa y, sin embargo, la comprensión profunda aparece con menos frecuencia de la que debería. El estudiante se cruza enseguida con una avalancha de nombres: HTML, CSS, JavaScript, DOM, React, router, SSR, hidratación, caché, loaders, server functions, middleware, streaming. El problema no es que esos nombres sean innecesarios. El problema es que suelen presentarse demasiado pronto, como si la tarea principal consistiera en memorizar piezas antes de entender el mecanismo del que esas piezas forman parte.

Cuando eso ocurre, la web se vuelve una especie de zoológico técnico. Uno reconoce animales, pero no entiende el ecosistema. Sabe que existe una URL, que hay requests, que React renderiza, que un framework resuelve ciertas cosas, pero no alcanza a ver por qué todo eso existe ni cuál es la presión real que obliga a que la arquitectura tome una forma determinada. En ese contexto, estudiar se parece demasiado a recordar recetas. Y las recetas funcionan mientras el ejercicio se parece al ejemplo; cuando cambian las condiciones, se deshacen.

Este libro intenta evitar ese destino desde el comienzo. La apuesta es sencilla y exigente a la vez: volver a los fundamentos hasta que las abstracciones modernas dejen de sentirse arbitrarias. No vamos a tratar TanStack Start como un catálogo de utilidades ni la web como una colección de tecnologías yuxtapuestas. Vamos a recorrer, paso a paso, el problema original que la web intenta resolver, las tensiones que aparecen cuando ese problema se vuelve serio y la razón por la cual un framework full-stack puede convertirse en una respuesta razonable.

La pregunta que irá guiando todo el recorrido es muy simple: qué problema real obliga a que exista cada concepto. Si la respuesta es clara, los nombres técnicos se ordenan solos. Si la respuesta es confusa, hasta la API mejor diseñada se vuelve un truco difícil de recordar. Dicho de otra manera, aquí no nos interesa acumular definiciones; nos interesa ganar una forma de pensar que permita reconstruir el sistema desde su base.

Por eso empezaremos mucho antes de TanStack Start. Primero hará falta comprender qué significa, en rigor, usar algo que vive en otra máquina. Después habrá que mirar con cuidado la diferencia entre navegador y servidor, entre recurso y representación, entre ver contenido y tener una aplicación realmente interactiva. Más adelante aparecerán las tensiones que convierten una app simpática en una app difícil: latencia, caché, seguridad, fallas parciales, observabilidad. Recién entonces TanStack Start entrará en escena, no como magia, sino como consecuencia arquitectónica de problemas ya bien entendidos.

Si el libro hace bien su trabajo, al final el lector no debería depender demasiado de la memorización. Debería poder explicar con sus propias palabras por qué la URL importa tanto, por qué un loader no es simplemente un fetch cómodo, por qué la seguridad no puede decidirse en el cliente, por qué la hidratación existe y por qué herramientas como TanStack Router, TanStack Query y TanStack Start se complementan en lugar de competir. Más que aprender una colección de técnicas, debería salir con una brújula.

## Parte I — La base: qué problema resuelve la web

### 1. La pregunta original: cómo usar algo que está en otra máquina

Si quitamos por un momento toda la terminología moderna, una aplicación web nace de una situación casi elemental. El usuario está frente a una máquina, pero la información o la capacidad que necesita no está allí. Puede tratarse de una noticia, de una imagen, de una base de datos, de una operación de pago, de un historial de compras o de una conversación con otra persona. El detalle cambia; la estructura del problema no. Lo que quiero usar vive en otra parte.

Esa observación, que parece inocente, tiene consecuencias inmensas. Obliga a abandonar la imagen de la aplicación como un programa entero y compacto, encerrado en un único lugar. Una aplicación web es, desde el principio, cooperación entre entornos separados. Hay una parte de la experiencia que sucede cerca del usuario, en el navegador, y otra parte que sucede en máquinas remotas, donde viven datos, reglas, secretos y recursos compartidos. Entre una cosa y la otra hay red, tiempo de viaje, posibilidad de falla y necesidad de coordinación.

Por eso conviene corregir una intuición bastante extendida. La web no es, en su esencia, una colección de páginas. Tampoco es, en primer término, un proyecto con una carpeta de frontend y otra de backend. Todo eso puede aparecer después, pero el fenómeno básico es otro: la web es una forma organizada de trabajar a distancia con recursos que no viven en la máquina del usuario. La pantalla visible es el resultado de ese acuerdo distribuido, no el sistema completo.

Pensemos en algo tan simple como abrir una foto alojada en un servidor remoto. El navegador no posee esa foto por adelantado. Tiene que averiguar adónde pedirla, hacer la petición, esperar la respuesta, recibir datos y convertirlos en algo visible. Ahora pensemos en una tienda online. El mecanismo profundo sigue siendo el mismo, aunque lo recubra mucha más complejidad: productos, stock, precios, carrito, permisos, recomendaciones, métodos de pago. En todos los casos el usuario intenta operar con algo que no está físicamente en su máquina.

En este punto aparece una idea que conviene fijar muy temprano, porque será el hilo conductor de todo lo demás. Una aplicación web seria es un sistema distribuido. No hace falta imaginar granjas de servidores para que esto sea cierto. Basta con que una parte del trabajo corra en el navegador y otra en el servidor para que aparezcan ya los problemas propios de la distribución: latencia, sincronización, confianza desigual, consistencia y fallas parciales. Si uno olvida esto, la arquitectura posterior parece caprichosa. Si lo recuerda, muchas decisiones empiezan a sentirse inevitables.

Esa es la base sobre la que habrá que construir todo. Antes de hablar de renderizado, de caché o de frameworks, hay que aceptar que la web tiene esta forma porque nace de esta necesidad: usar a distancia lo que no vive aquí.

### 2. Nombrar, localizar y conversar: la infraestructura mínima de una petición

Una vez que entendemos que queremos usar algo remoto, aparece una pregunta práctica: cómo se lo pide. Y esa pregunta, en realidad, contiene varias a la vez. Hay que saber con qué máquina quiero hablar, qué recurso de esa máquina me interesa y bajo qué lenguaje o contrato va a ocurrir la conversación. Gran parte de la infraestructura de la web existe para responder precisamente a eso.

Empecemos por lo más básico. Las máquinas en la red se identifican operativamente mediante direcciones. Pero los seres humanos no piensan bien con direcciones numéricas largas; piensan con nombres. DNS existe para tender ese puente. Cuando escribimos un dominio, no estamos nombrando una ubicación física definitiva, sino una identidad pública que luego puede resolverse a una dirección útil para la red. Esa separación es de una elegancia enorme, porque libera al sistema de acoplar el nombre que recordamos con la topología técnica que puede cambiar por detrás. Un mismo dominio puede apuntar hoy a cierta infraestructura y mañana a otra, y el usuario ni siquiera debería notarlo.

También conviene notar que esta resolución no es instantánea ni mística. DNS es un sistema distribuido con cachés, tiempos de vida y sucesivas consultas. Esto importa porque el viaje del usuario no empieza cuando nuestra aplicación ejecuta lógica propia; empieza bastante antes. Una experiencia lenta puede estar pagando costos incluso antes de alcanzar la primera línea de código que nosotros escribimos.

Ahora bien, encontrar la máquina correcta todavía no basta. En un mismo servidor puede haber miles de recursos posibles. Hace falta una forma de expresar no solo con quién quiero hablar, sino también qué quiero obtener de esa conversación. Esa es la tarea de la URL. Y aquí aparece una intuición importante: una URL no es un mero string largo ni un detalle decorativo que conviene “poner bonito”. Es una frase pública y estructurada. El dominio identifica el sistema. El path señala una zona o un recurso. Los query parameters afinan el contexto. El fragmento, cuando existe, ya no pide algo nuevo al servidor, sino que apunta a una parte del documento recibido.

Cuando una URL está bien diseñada, no solo localiza. También cuenta. Dice qué experiencia está atravesando el usuario y bajo qué coordenadas se la puede reproducir. Si alguien comparte `/products/42`, está compartiendo mucho más que una dirección técnica: está compartiendo una manera estable y pública de referirse al producto 42. Si comparte `/products?category=laptops&sort=price-asc&page=3`, está compartiendo además filtros, criterio y contexto de navegación. Esa es la razón profunda por la que el manejo de URL termina siendo una decisión arquitectónica, no cosmética.

Aquí aparece una distinción fina pero decisiva entre recurso y representación. El recurso es la entidad conceptual que queremos usar: un producto, un artículo, un usuario, una factura. La representación es la forma concreta en que el sistema nos lo entrega en ese momento: HTML, JSON, una imagen, un stream, una respuesta parcial condicionada por permisos o por idioma. La web no nos transfiere directamente los objetos internos del servidor; nos da representaciones negociadas de aquello que pedimos. Esta diferencia, que a veces parece sutil, se vuelve importantísima cuando hablamos de APIs, de caché y de renderizado.

Todavía falta una pieza más. Ya sé con qué sistema quiero hablar y qué recurso me interesa, pero necesito acordar cómo se formula la petición y cómo se interpreta la respuesta. Allí entra HTTP. Su valor no está en obligarnos a memorizar una lista de métodos y códigos, sino en ofrecer un contrato compartido para que muchas capas distintas puedan colaborar sin malentendidos. Un `GET` no expresa la misma intención que un `POST`; un código `200` no significa lo mismo que un `404` o un `302`; las cabeceras aportan contexto sobre tipo de contenido, caché, autenticación, compresión y muchas otras cosas.

Si alguien hace `GET /products/42`, está diciendo algo muy preciso: quiere una representación del producto 42. Si alguien hace `POST /orders`, no está pidiendo solo ver información; está intentando producir un efecto en el sistema, por ejemplo crear una orden. La distinción semántica importa porque no la usan únicamente nuestra aplicación y nuestro equipo. También la usan el navegador, los proxies, las CDN, las herramientas de monitoreo y las políticas de caché. Una web inteligible depende de que esa conversación sea semánticamente clara.

Visto así, ya se insinúa algo que más adelante será central. Si la URL forma parte del significado público de la aplicación y HTTP define la gramática de la conversación, entonces el router no puede ser un simple adorno visual. Está tocando el corazón mismo del sistema.

### 3. Navegador y servidor: dos entornos con poderes distintos

Hasta aquí hemos hablado de una aplicación repartida entre máquinas. Ahora conviene mirar de cerca las dos cajas de herramientas más importantes de ese reparto. El navegador y el servidor no son “dos lados” en un sentido vago. Son entornos de ejecución diferentes, con capacidades, restricciones y grados de confianza distintos. Casi toda decisión arquitectónica valiosa en la web nace de aceptar esa diferencia en lugar de maquillarla.

El navegador vive cerca del usuario. Esa cercanía le da una ventaja formidable. Tiene acceso inmediato a la pantalla, al DOM, a los eventos, al scroll, al foco, al tamaño del viewport, a la interacción casi instantánea. Puede sostener pequeños estados efímeros de interfaz y reaccionar con una inmediatez imposible para una máquina remota. Cuando un input se tiñe de rojo mientras escribimos, cuando un panel se despliega, cuando un modal se abre sin esperar viajes de red, estamos aprovechando justamente la fuerza del entorno cliente.

Pero esa fuerza tiene un límite estructural. El navegador corre en una máquina que no controlamos del todo. El usuario puede inspeccionar el código, alterarlo, repetir requests, interrumpir procesos, manipular almacenamiento local. Por eso el navegador no puede considerarse autoridad final sobre decisiones sensibles. Además, por definición, no posee acceso directo a secretos del sistema, a conexiones privilegiadas con la base de datos ni a variables de entorno privadas. Puede colaborar con la experiencia; no puede ocupar el lugar de aquello que debe permanecer bajo control del servidor.

El servidor, en cambio, vive lejos pero bajo nuestra administración. Allí están los secretos, las credenciales, el acceso privilegiado a la base de datos, las validaciones finales, las reglas de autorización, la posibilidad de generar HTML inicial y de componer respuestas con más autoridad. El servidor tiene algo que el cliente nunca tendrá: confianza operativa. Si una decisión requiere secretos o capacidad privilegiada, ese es su territorio natural.

La desventaja del servidor no es moral sino física. Está lejos. Cada vez que el navegador necesita algo de él, paga latencia, exposición a la red y posibilidad de falla. De modo que la arquitectura madura no glorifica ciegamente ni al cliente ni al servidor. No repite el mantra “todo al backend” ni el mantra opuesto “todo a la SPA”. Lo que hace es repartir responsabilidades con criterio, preguntándose en cada caso qué entorno tiene la herramienta correcta, el costo aceptable y el nivel de confianza apropiado.

Una forma útil de imaginarlo es pensar en dos talleres que construyen una misma obra. Uno trabaja pegado al espectador y puede mover rápidamente aquello que afecta a la experiencia inmediata. El otro trabaja con las llaves del depósito, con los materiales valiosos y con las reglas del oficio. Sería absurdo pedirle al primer taller que custodie el inventario y sería igualmente torpe obligar al segundo a intervenir en cada pequeño gesto visual. La web funciona bien cuando esas responsabilidades se reparten donde tienen sentido.

En un formulario de registro esto se ve enseguida. El navegador puede comprobar si el campo email tiene un formato razonable y puede dar feedback instantáneo mientras el usuario escribe. Pero validar de verdad si ese email ya existe en la base y si la cuenta puede crearse pertenece al servidor. En un checkout ocurre lo mismo a otra escala. El cliente puede mostrar el carrito y recalcular una vista preliminar; el servidor debe confirmar el stock real, el precio final y la operación de pago.

Cada vez que una aplicación se vuelve confusa, suele ser buena idea volver a esta pregunta: quién debería ser responsable de esto, dadas las capacidades reales de cada entorno. Esa pregunta, tan sencilla, es el cimiento del criterio full-stack.

### 4. El estado: la información que gobierna lo que la aplicación puede decir y hacer

En cuanto una interfaz deja de ser completamente estática, aparece una palabra que se usa sin descanso: estado. Pero como ocurre con muchos términos demasiado repetidos, a veces termina perdiendo filo. Conviene devolverle su significado más simple. Estado es la información de la que depende lo que la aplicación muestra o cómo se comporta en un momento dado. Si esa información cambia y la interfaz debería cambiar con ella, entonces estamos ante estado.

La definición parece modesta, pero enseguida revela algo importante: no todo estado es de la misma clase. Y si mezclamos clases distintas como si fueran equivalentes, empezamos a construir incoherencia. Hay información que pertenece conceptualmente al servidor, aunque el cliente la tenga en memoria durante un rato. Hay información que solo describe detalles pasajeros de la interfaz. Hay información que forma parte de la navegación pública y por eso debería expresarse en la URL. Y hay información que ni siquiera merece guardarse porque puede derivarse de otra.

Imaginemos una pantalla de productos. La lista de productos y sus precios pertenece al mundo del dominio; la fuente de verdad está del lado del sistema y no en la memoria efímera del navegador. Si la aplicación decide recordarlos temporalmente, lo hace para no volver a pedirlos todo el tiempo, no porque el cliente se haya convertido en el dueño conceptual de esos datos. Muy distinto es el caso de un acordeón abierto, una pestaña seleccionada o el borrador de un campo que el usuario todavía no envió. Ahí la cercanía del navegador es una ventaja y no hace falta elevar esa información a una categoría más alta.

Hay, además, un tipo de estado subestimado que cambia por completo la calidad de una aplicación: el que debería vivir en la URL. Filtros, paginación, criterio de orden, pestaña activa en ciertas vistas, recurso seleccionado. Cuando esa información se guarda solo en memoria local, la experiencia se vuelve difícil de compartir, de recargar, de razonar y de navegar con el historial. Cuando se la expone en la URL, la aplicación gana reproducibilidad y la navegación deja de ser una especie de secreto privado entre el usuario y la pestaña actual.

Otra fuente clásica de confusión es el estado derivado. Si el total de productos visibles puede calcularse a partir de los productos ya cargados y de los filtros activos, almacenarlo por separado como si fuera fuente de verdad suele ser una invitación al bug. En programación, duplicar autoridad es casi siempre sembrar conflicto futuro. Lo mismo vale aquí. No porque algo pueda escribirse en una variable significa que deba tratarse como estado independiente.

Todo esto importa mucho más de lo que parece al comienzo, porque define cómo se ordena la aplicación. TanStack Router se vuelve especialmente valioso cuando entendemos que cierta información pertenece a la navegación y debe vivir en la URL. TanStack Query cobra sentido cuando entendemos que el estado del servidor tiene problemas propios de frescura, caché, revalidación e invalidación. Si se borra esta diferencia, las herramientas parecen solaparse. Si se la ve con nitidez, cada una cae en su sitio natural.

La idea de fondo es simple: antes de preguntar “dónde guardo esto” conviene preguntar “qué clase de verdad es esta”. Esa inversión ahorra una cantidad enorme de confusión.

## Parte II — Cómo un documento se convierte en una aplicación

### 5. De la estructura a la interacción: HTML, CSS, JavaScript, DOM y eventos

Después de pedir algo remoto y recibirlo, todavía no tenemos una experiencia utilizable. Tenemos, en el mejor de los casos, datos o un documento. Para que eso se convierta en una interfaz habitable intervienen varias capas de la plataforma web, y conviene mirarlas como piezas complementarias de un mismo mecanismo, no como disciplinas inconexas.

HTML es la capa que estructura y nombra. Cuando escribimos títulos, párrafos, formularios, botones, listas o tablas, no estamos solo “poniendo etiquetas”. Estamos declarando qué cosas hay en el documento y qué papel cumplen. Esa semántica importa más de lo que a veces se sospecha, porque el navegador no es el único lector de la página. También la leen tecnologías de accesibilidad, motores de indexación y herramientas que dependen de una estructura inteligible para operar bien. Un botón no es simplemente un rectángulo cliqueable; es una pieza del lenguaje del documento.

CSS aparece a veces como puro maquillaje, pero esa lectura se queda corta. La presentación también organiza comprensión. El modo en que distribuimos espacio, jerarquías visuales, contraste, ritmos y adaptaciones a distintos tamaños de pantalla no embellece un contenido terminado; participa de la forma en que ese contenido puede entenderse. Una interfaz bien diseñada es una explicación silenciosa.

JavaScript introduce el tiempo y la reacción. Gracias a él, la interfaz deja de ser solo una fotografía para convertirse en algo que responde a eventos, cambia de estado, pide nuevos datos y modifica porciones del documento sin recargarlo entero. No reemplaza ni la estructura ni la presentación; las pone en movimiento.

El DOM es la forma en que el navegador mantiene una representación viva del documento en memoria. No es el HTML como texto, sino el árbol operativo sobre el que se aplican estilos, se conectan eventos y se realizan cambios. Cuando el usuario hace clic, escribe, envía un formulario o desplaza la página, el navegador genera eventos. Esos eventos son, por así decirlo, la noticia de que algo ocurrió en el mundo del usuario y que el programa quizá deba responder.

Un contador sencillo deja ver esta cooperación. Hace falta un botón y un espacio donde mostrar el número; eso pertenece a la estructura. Hace falta decidir cómo se ve el botón y cómo se integra visualmente en la página; eso pertenece a la presentación. Hace falta, además, escuchar el clic y actualizar el valor visible; ahí entra el comportamiento. El usuario percibe un único gesto coherente, pero debajo de él están trabajando varias capas distintas.

Cuando la interfaz crece, manipular manualmente el DOM empieza a resultar trabajoso y propenso al error. Allí es donde React introduce una simplificación poderosa. En lugar de pensar todo el tiempo qué nodo tocar y en qué secuencia hacerlo, uno pasa a describir cómo debería verse la interfaz dado cierto estado. La propuesta es profundamente útil, pero solo si se la entiende bien. React no sustituye la plataforma web. Sigue habiendo HTML, CSS, navegador, eventos y DOM. Lo que hace es ofrecer un modelo declarativo para coordinar mejor esa complejidad.

Perder de vista la plataforma es un error caro. Más adelante, cuando aparezcan SSR, hidratación o componentes que solo pueden correr en el cliente, la comprensión de fondo seguirá dependiendo de esta base. React ayuda a hablar mejor el idioma de la web; no nos exime de aprenderlo.

### 6. Qué sucede realmente cuando alguien entra a una URL

Conviene ahora seguir una navegación completa, porque muchas ideas que suelen aprenderse aisladas se vuelven mucho más claras cuando se observan en movimiento. Una URL no “abre una página” de una vez y sin intermediarios. Desencadena una cadena de procesos en la que cada eslabón puede influir sobre la experiencia final.

Todo empieza con una intención humana. El usuario escribe una dirección, pulsa Enter, hace clic en un enlace o dispara una navegación interna. Si el dominio aún no está resuelto, se activa el proceso correspondiente para traducir ese nombre a una dirección útil. Luego viene la conexión, y a menudo también el establecimiento de la capa segura. Recién cuando esa preparación está en condiciones, el navegador puede emitir una request HTTP con suficiente precisión como para pedir lo que necesita.

Esa request no llega necesariamente de manera directa a la lógica que más nos interesa. Puede atravesar una CDN, donde acaso exista ya una versión cacheada del recurso. Puede pasar por proxies, balanceadores, middleware o infraestructura intermedia que añade cabeceras, controla seguridad, comprime respuestas o toma métricas. Desde la perspectiva del usuario, todo eso forma parte de “cargar la página”, aunque nosotros no lo veamos en el componente.

Cuando la petición alcanza la aplicación, hay varias decisiones por delante. Hay que reconocer qué ruta corresponde a esa URL, interpretar parámetros, validar búsqueda, quizás reconstruir la sesión del usuario, decidir si hace falta cargar datos, evaluar permisos y determinar qué clase de respuesta conviene producir. En algunos casos será HTML, en otros JSON, en otros una redirección, un error o un stream parcial. El trabajo de la app empieza mucho antes del primer píxel visible.

Después, la respuesta regresa al navegador. Si contiene HTML, el navegador lo parsea, construye el DOM, resuelve estilos, calcula layout y pinta. Si además la experiencia necesita JavaScript para comportarse como aplicación, ese código tendrá que descargarse, ejecutarse y conectarse con la estructura ya visible. Si hubo renderizado del lado servidor, ese proceso de conexión toma una forma particular: la hidratación.

Aquí conviene corregir una simplificación demasiado frecuente. “La página abrió” no describe un único instante homogéneo. Entre el inicio de la navegación y el momento en que la aplicación está realmente lista pueden haber ocurrido muchas cosas distintas: resolución de nombre, establecimiento de conexión, espera por datos, render del servidor, descarga de assets, parseo de HTML, ejecución de JavaScript, hidratación, consultas adicionales. Cada etapa tiene su propio costo y su propio tipo de falla posible.

Esa diferencia importa enormemente cuando una app se siente lenta. Sin este mapa mental, es fácil culpar a la primera capa visible. Pero la lentitud puede estar en DNS, en una consulta a la base, en el tamaño del bundle, en un waterfall de requests o en un mismatch que retrasa la hidratación. Diagnosticar bien exige ver el viaje completo, no solo el último tramo.

### 7. Renderizar no es mostrar: es distribuir trabajo en el espacio y en el tiempo

Cuando se habla de renderizado web aparecen enseguida siglas como CSR, SSR, SSG, ISR, streaming e hidratación. Muchas discusiones se empobrecen porque convierten esas siglas en identidades ideológicas. Pero el problema real es mucho más sobrio y bastante más interesante. Cada estrategia de render responde, en el fondo, a una misma pregunta: quién hace el trabajo de producir la primera experiencia útil y en qué momento conviene pagarlo.

En una estrategia centrada en el cliente, el navegador recibe una base relativamente delgada y construye buena parte de la interfaz después de descargar y ejecutar JavaScript. Esto puede ser perfectamente razonable en aplicaciones muy interactivas, donde una vez iniciada la sesión el usuario permanece mucho tiempo dentro del mismo sistema y la riqueza de interacción compensa el costo inicial. El precio a pagar es evidente: si la primera pantalla depende demasiado del bundle, puede haber un lapso apreciable entre la navegación y el contenido realmente útil.

El render del lado servidor desplaza parte de ese trabajo hacia una máquina remota que compone HTML antes de enviarlo. La ventaja es que el usuario puede ver contenido significativo mucho antes, a veces incluso antes de que el JavaScript haya terminado de llegar. Pero aquí conviene ser precisos. SSR no elimina el cliente. Si la pantalla necesita seguir siendo interactiva, el navegador igual tendrá que cargar código y enlazarlo con lo ya visible. El servidor adelanta una porción del trabajo; no borra el resto del proceso.

El prerenderizado o generación estática empuja aún más atrás la producción de esa primera representación. Si una página puede conocerse de antemano y cambia poco, resulta natural construirla antes de que exista la request concreta del usuario. La motivación es casi de sentido común: si ya sé cómo debe verse, por qué esperar a que alguien llame a la puerta para empezar a armarla. ISR introduce un matiz adicional. No todo contenido merece recalcularse en cada visita, pero tampoco todo puede quedarse congelado indefinidamente. Ahí aparece la revalidación incremental como forma de negociar entre frescura y costo.

El streaming responde a otra dificultad. A veces una página tiene partes rápidas y partes lentas. Si esperamos a que lo más costoso esté listo para enviar cualquier cosa, obligamos al usuario a contemplar el vacío. El streaming permite empezar a mandar antes las partes que ya están disponibles y continuar con las demás a medida que se resuelven. No es una sofisticación ornamental; es una decisión sobre cómo repartir la espera.

La hidratación, por su parte, aclara una confusión muy común entre ver y poder usar. Si el servidor envía un botón como HTML, el usuario puede verlo enseguida. Pero eso no garantiza todavía que el botón responda a un clic, que el formulario procese eventos o que los estados locales estén vivos. Hidratar es conectar el árbol visible con el código cliente-side que lo vuelve una aplicación en sentido pleno. Es la diferencia entre un decorado y un mecanismo funcional.

Una página de producto muestra muy bien esta combinación. El título, el precio y la descripción pueden llegar ya renderizados para que el usuario perciba contenido desde el principio. Luego el cliente hidrata el botón de compra y la interacción del carrito. Mientras tanto, recomendaciones o reseñas pueden seguir llegando de forma progresiva. Lo que el usuario vive como una sola pantalla es, en verdad, una negociación cuidadosa sobre dónde y cuándo se hizo cada parte del trabajo.

La pregunta madura, entonces, no es cuál sigla gana la discusión. La pregunta es qué reparto de trabajo conviene para esta ruta, para esta experiencia y para estos costos concretos.

## Parte III — Los problemas que vuelven seria a una aplicación

### 8. La latencia como fuerza arquitectónica

En programación solemos asociar rendimiento con algoritmos y cómputo local. En la web eso importa, pero hay una presión que muchas veces domina por encima de otras: la distancia entre máquinas. La latencia no es un detalle periférico que se corrige al final; reordena desde adentro la forma de diseñar una aplicación.

La razón es simple. Un cálculo local puede optimizarse dentro de un mismo proceso. Un viaje de red, en cambio, tiene un costo físico difícil de negociar. Si para llegar a una pantalla útil la aplicación obliga a realizar varias esperas en secuencia, el usuario paga la suma de esas esperas aunque cada operación individual sea correcta. La lentitud no viene necesariamente de una tarea monstruosa; a veces viene de descubrir demasiado tarde dependencias que ya eran previsibles.

Eso es exactamente lo que ocurre en los llamados waterfalls. La app arranca, luego descarga JavaScript, después un componente decide pedir datos, con esa respuesta aparece otro componente que recién entonces descubre que necesita otra consulta, y así sucesivamente. No estamos ante un problema de gran complejidad algorítmica, sino ante una coreografía mal distribuida en el tiempo. El sistema no avanza; espera demasiadas veces.

Supongamos una pantalla que necesita información del usuario, permisos y productos. Si se resuelve primero una cosa, luego la siguiente y recién después la tercera, el tiempo total no se parece al mayor de esos costos sino a su suma serial. En cambio, si la ruta ya sabía desde el inicio qué datos eran necesarios, pudo haberlos pedido antes, quizá en paralelo, quizá incluso durante una precarga de navegación. La diferencia entre ambas experiencias no es cosmética: es estructural.

Por eso también conviene matizar un juicio simplista que aparece seguido: “hacer fetch en el cliente está mal”. No. Lo problemático no es el cliente en sí mismo, sino descubrir tarde una necesidad que ya era conocida y obligar a la aplicación a pagar latencia en cascada. Habrá casos en los que una consulta desde el cliente sea totalmente razonable. La cuestión es si la dependencia fue ubicada en el momento correcto.

Aquí empieza a verse por qué routing, datos y render no deberían pensarse como mundos estancos. Si el router ya sabe hacia qué pantalla se dirige el usuario, esa información puede usarse para preparar trabajo antes de que la necesidad se vuelva visible demasiado tarde. El diseño bueno no elimina la latencia, pero intenta no multiplicarla por descuido.

### 9. Caché: memoria útil, verdad aplazada

El caché nace de una idea que parece casi obvia: si hace muy poco ya obtuve una respuesta, quizá no tenga sentido volver a pagar todo el costo de conseguirla. Recordar puede ser más barato que preguntar otra vez. Sin embargo, esa conveniencia inmediata encierra una tensión profunda. Cada vez que usamos caché, aceptamos que durante cierto tiempo trabajaremos con una versión que podría no ser la más fresca. El caché es valioso precisamente porque transforma esa tensión en una política deliberada.

Por eso es mejor pensarlo no como un simple acelerador, sino como un pacto entre velocidad y frescura. A veces ese pacto es tranquilísimo. Un asset versionado por hash, como un archivo CSS cuyo nombre cambia cuando cambia su contenido, puede cachearse de manera muy agresiva sin casi riesgo conceptual. En otras situaciones el pacto se vuelve mucho más delicado. Un saldo bancario, un permiso recién revocado o un stock crítico no toleran con la misma facilidad la idea de estar viendo una versión anterior.

Hablar de “el caché” en singular también induce a error. En la práctica hay muchos cachés superpuestos: el del navegador, el de una CDN, el de proxies intermedios, el del servidor, el de una librería cliente que decide recordar datos del servidor por un tiempo. Cada capa tiene reglas y propósitos distintos. Cuando algo se siente desactualizado, la pregunta útil rara vez es “qué pasó con el caché”, como si hubiera una sola caja mágica recordando cosas. La pregunta buena es quién recordó qué, dónde y bajo qué criterio.

En el fondo, toda política de caché necesita responder tres cosas. Primero, qué estoy dispuesto a recordar. Segundo, en qué lugar del sistema quiero recordar eso. Tercero, durante cuánto tiempo considero aceptable esa versión. Parece una formulación austera, pero ayuda muchísimo. Obliga a pasar de la intuición vaga “hay que cachear” a una decisión concreta sobre consistencia.

Cuando una mutación cambia la realidad del sistema, aparece además el problema de la invalidación. No basta con haber recordado bien; ahora hay que saber cuándo dejar de confiar en ese recuerdo. Allí se ve con claridad que el caché no es solo cuestión de guardar resultados, sino de definir el momento en que la comodidad de reutilizarlos deja de ser intelectualmente honesta. TanStack Query se vuelve especialmente potente en este terreno porque trata el estado del servidor como algo que necesita políticas explícitas de frescura, no como un dato cualquiera olvidado en un store.

Usar caché con criterio es, en última instancia, aceptar que la velocidad nunca es gratis. Siempre se compra con alguna teoría sobre cuánto nos permitimos alejarnos, por un rato, de la verdad más reciente.

### 10. Seguridad: la frontera de confianza no coincide con la comodidad

La seguridad web suele enseñarse mediante rituales sueltos: login, cookie, token, roles, guards. Todo eso importa, pero debajo hay una realidad más desnuda y más fértil para el pensamiento. Parte de la aplicación corre en un entorno que el sistema no controla por completo: la máquina del usuario. De esa sola observación nace la necesidad de trazar una frontera de confianza.

Esa frontera obliga a distinguir con precisión entre autenticación y autorización. Autenticar es establecer quién es el usuario. Autorizar es decidir qué puede hacer. La interfaz puede reflejar ambas cosas, pero no constituye por sí misma la autoridad definitiva sobre ninguna. Si el cliente oculta un botón de borrado a quien no tiene permisos, mejora la experiencia y reduce confusión. Pero la protección real ocurre cuando el servidor recibe la petición de borrar y decide, con información confiable, si esa operación está permitida.

También se sigue de aquí una regla que parece trivial solo hasta que alguien la rompe: los secretos no pertenecen al bundle del cliente. Claves privadas, credenciales de base de datos, tokens privilegiados, lógica sensible que dependa de información no pública, todo eso debe vivir del lado servidor. El hecho de que un framework facilite compartir archivos entre entornos no cambia esta verdad. Compartir código no equivale a compartir legitimidad.

Hay otro matiz importante. La web trabaja sobre requests discretas. No existe una memoria invisible y eterna que preserve por sí sola el contexto de seguridad entre una operación y la siguiente. Cada vez que una acción importante llega al sistema, este necesita reconstruir o validar la identidad relevante, normalmente a partir de cookies, cabeceras, sesión u otros mecanismos equivalentes. La seguridad no se hereda por simpatía entre una pantalla y otra; se reestablece allí donde la autoridad debe ejercerse.

Esto cambia por completo la lectura de muchas decisiones de UI. Deshabilitar un botón no es validar. Ocultar un menú no es proteger. Reducir visualmente la superficie de acción puede ser útil, pero la frontera efectiva sigue estando en el lado que controla la operación y puede inspeccionar la request con criterio. Cuando uno entiende esto, la arquitectura deja de apoyarse en ilusiones visuales.

TanStack Start hace especialmente visible esta división porque permite mezclar ergonomía full-stack con límites explícitos de ejecución. Justamente por eso conviene llegar a sus APIs con esta intuición ya asentada. De otro modo, la comodidad del framework puede hacernos olvidar demasiado pronto dónde está la autoridad real.

### 11. Fallas, degradación y observabilidad: aprender a ver el sistema real

Las aplicaciones de ejemplo rara vez fallan de manera interesante. La aplicación real, en cambio, sí. Puede demorar una API externa, vencer una sesión, romperse una validación, caerse una base de datos o quedar una parte del sistema operativa mientras otra no responde. La madurez arquitectónica empieza cuando dejamos de pensar los errores como accidentes vergonzosos y empezamos a tratarlos como parte de la realidad que el sistema debe saber habitar.

Lo primero que conviene distinguir es la falla técnica de la experiencia de la falla. Un problema interno puede traducirse en una pantalla completamente rota, pero también puede degradarse de manera controlada. El usuario puede recibir una explicación inteligible, una opción de reintento, una vista parcial que conserve lo ya disponible o un fallback razonable. Una interfaz adulta no se limita a alternar entre éxito total y desastre total. Reconoce estados intermedios: carga, vacío, error recuperable, información parcial, sesión vencida, permiso insuficiente.

La observabilidad entra exactamente en este punto. Un sistema observable no es uno que imprime muchos logs; es uno que deja responder preguntas importantes sobre su comportamiento. Qué request falló. Cuánto tardó. En qué capa estuvo el cuello de botella. Qué mutación disparó un error. Qué rutas concentran mayor fragilidad. La diferencia entre observar y adivinar es la diferencia entre intervenir con criterio o moverse a ciegas.

Si una navegación tarda demasiado, no alcanza con anotar “anda lento”. Queremos saber si la demora estuvo en la red, en la base de datos, en una llamada a otro servicio, en el render del servidor, en el tamaño del bundle o en el tiempo de hidratación. Si una acción del usuario recorre middleware, server functions, loaders y componentes interactivos, hace falta una forma de seguir ese trayecto. De otro modo la aplicación se vuelve opaca y, con ella, la capacidad de pensar sobre ella.

Por eso el manejo de errores y la observabilidad no son lujos corporativos agregados al final. Son parte del mismo esfuerzo intelectual: hacer legible un sistema distribuido que por naturaleza puede fallar. TanStack Start ofrece puntos claros para boundaries, middleware y composición full-stack. Pero esas herramientas valen de verdad cuando uno entiende que el objetivo no es decorar el error ni coleccionar logs, sino permitir que el sistema siga siendo inteligible cuando las cosas dejan de funcionar de manera ideal.

## Parte IV — Por qué aparece un framework como TanStack Start

### 12. El momento en que “funciona” deja de ser suficiente

En una aplicación muy pequeña, muchas decisiones pobres pasan inadvertidas. La URL representa mal la navegación, pero nadie se queja. Los datos llegan tarde, pero el ejemplo tiene dos pantallas. La seguridad está puesta a medias, pero no hay operaciones realmente delicadas. El estado del servidor se mezcla con estado local y, aun así, la demo sobrevive. Durante un tiempo eso da la ilusión de que la arquitectura es una preocupación secundaria.

El problema empieza cuando la aplicación deja de ser un ejercicio y empieza a parecerse a un sistema real. Entonces hay que sostener rutas públicas y privadas, layouts anidados, estados que deberían sobrevivir a recargas, datos que necesitan políticas de frescura, acciones sensibles, errores contenidos por zonas, observabilidad, estrategias distintas de render según la página y una frontera más clara entre código que puede viajar al cliente y código que jamás debería hacerlo. En ese punto la dificultad principal ya no es “hacer que corra”. La dificultad es impedir que el sistema se vuelva mentalmente inmanejable.

Los frameworks modernos nacen como respuestas a ese problema de orden. No existen solo para ahorrar líneas ni para impresionar con features. Existen porque, a cierta escala, una aplicación necesita que algunas decisiones estructurales dejen de improvisarse una por una. Necesita un marco que diga, de manera más o menos explícita, dónde se define la navegación, cómo se cruzan cliente y servidor, qué puntos del ciclo de vida son adecuados para datos, errores o middleware, y qué mecanismos protegen la coherencia del conjunto.

Por eso la pregunta útil frente a un framework no es cuántas cosas promete hacer. La pregunta fructífera es qué problema organizativo intenta resolver y qué aspecto de la arquitectura coloca en el centro. Solo desde ahí tiene sentido estudiar TanStack Start.

### 13. TanStack Start y la decisión de poner el router en el centro

TanStack Start se presenta hoy como un framework full-stack para React, apoyado en TanStack Router y en Vite, con SSR de documento completo, streaming, server functions y bundling tanto para cliente como para servidor. La enumeración es correcta, pero no alcanza para captar lo más importante. Su decisión arquitectónica más interesante es otra: tratar al router como columna vertebral de la aplicación, no como un accesorio tardío encargado solo de cambiar pantallas.

Esta decisión encaja con todo lo que ya vimos. Si la URL forma parte del significado público de la app, si la navegación influye sobre qué datos hacen falta, si los layouts y los errores tienen estructura jerárquica, entonces el router no puede quedar relegado a una esquina. Necesita convertirse en el lugar donde se encuentran la semántica de la URL, la composición visual, la carga de datos, los parámetros, la búsqueda, los límites de error y buena parte de la estrategia de render.

El file-based routing ayuda a volver visible esa idea. El árbol de rutas expresado en archivos no es solamente una comodidad de organización. Es una forma de hacer tangible la estructura de navegación del sistema y de conectar esa estructura con type safety, code-splitting y jerarquía de layouts. Cuando uno mira una ruta anidada, no está viendo únicamente dónde vive un componente; está viendo cómo se organiza una zona del espacio semántico de la aplicación.

La ruta raíz, ubicada típicamente en `__root.tsx`, muestra muy bien esta filosofía. Allí no solo se monta un componente más. Allí se compone el documento base, con su `html`, su `head`, su `body`, los scripts y el lugar donde se insertan las rutas hijas. El router participa, entonces, de la construcción del documento mismo. No es un mecanismo encerrado dentro de una página ya dada; ayuda a definir la forma del contenedor que hará posible todas las páginas.

Pensemos en una ruta como `/posts/$postId`. No es solo “la pantalla del post”. Es una unidad donde importan el parámetro que identifica el recurso, los datos mínimos que hacen que esa navegación tenga sentido, el layout que la envuelve, las posibles condiciones de no encontrado y la estrategia de render apropiada para ese segmento del árbol. Cuando se comprende esto, resulta natural que TanStack Start trate a la ruta como una pieza con mucha más responsabilidad que un simple switch visual.

En muchas aplicaciones se subestima el router porque se lo aprende primero como sinónimo de “navegar sin recargar”. Eso es cierto, pero demasiado pobre. En un sistema serio, el router organiza la geografía conceptual de la aplicación. TanStack Start toma esa idea al pie de la letra.

### 14. El modelo de ejecución: isomórfico por defecto no significa idéntico

Uno de los puntos más importantes de la documentación oficial de TanStack Start es su modelo de ejecución. La idea central puede resumirse en una frase: el código es isomórfico por defecto. En otras palabras, salvo que uno imponga una restricción explícita, ese código puede formar parte tanto del bundle del servidor como del bundle del cliente. La ventaja inmediata es clara: compartir lógica útil entre entornos reduce duplicación y facilita un modelo mental unificado.

Pero esa comodidad trae un riesgo si se la interpreta mal. Isomórfico no quiere decir que cliente y servidor se hayan vuelto el mismo lugar. Quiere decir únicamente que cierta pieza de código puede existir en ambos. Las capacidades reales siguen siendo distintas. El navegador sigue teniendo DOM y eventos, pero no secretos del sistema. El servidor sigue pudiendo tocar la base de datos y las variables privadas, pero no tiene viewport ni interacción directa con el usuario. Compartir presencia no equivale a compartir poderes.

La documentación de Start insiste, con razón, en un ejemplo que conviene grabarse: los loaders de ruta son isomórficos. En la request inicial pueden correr en el servidor, y en navegaciones posteriores pueden correr en el cliente. Esta sola observación desarma un malentendido frecuente. Un loader no es, por definición, un espacio server-only. Si su código depende de algo que solo existe en el servidor, entonces necesita apoyarse en un mecanismo más explícito.

```tsx
export const Route = createFileRoute('/users')({
  loader: async () => {
    const response = await fetch('/api/users')
    return response.json()
  },
})
```

Este loader es razonable porque conversa con una superficie pública de la aplicación. Sería muy distinto si intentara abrir una conexión directa a la base o leer una variable secreta del proceso. El hecho de que un archivo esté cerca de una ruta no lo vuelve automáticamente privilegiado.

Para hacer visibles esas fronteras, Start ofrece herramientas más específicas. `createServerFn()` define lógica que corre en el servidor aunque pueda invocarse desde el cliente como una llamada remota con buena ergonomía. `createServerOnlyFn()` marca utilidades que nunca deberían existir del lado cliente. En el sentido opuesto, `createClientOnlyFn()` o `ClientOnly` sirven cuando una lógica o un componente dependen esencialmente del navegador. Y `useHydrated` permite saber cuándo la aplicación ya pasó efectivamente a estar hidratada en el cliente.

La enseñanza de fondo no es memorizar nombres, sino preservar una disciplina mental. Cada vez que compartimos código entre entornos conviene preguntarse qué está compartiéndose realmente: una lógica pura y portable, o una responsabilidad que depende de capacidades que no están presentes en todas partes. TanStack Start hace muy cómodo compartir código; no vuelve legítimo olvidar dónde se está ejecutando.

### 15. Server functions, server routes y middleware: tres maneras de cruzar la frontera

Cuando una aplicación necesita hacer algo del lado servidor, no siempre necesita el mismo mecanismo. Esta distinción es importante porque gran parte de la elegancia de TanStack Start consiste precisamente en no reducir todo lo server-side a una sola caja. Las server functions, las server routes y el middleware resuelven problemas emparentados, pero no idénticos.

Las server functions son la respuesta de Start al deseo de tener RPC interno tipado y cómodo. Queremos ejecutar lógica en el servidor y llamarla desde un componente, un loader o incluso otra server function sin tener que escribir manualmente un endpoint por cada caso, serializar y mantener tipos duplicados en varios lugares. La ergonomía es excelente, pero solo si no borra la realidad de fondo: una server function no es una función local ordinaria. Sigue implicando cruce de red, serialización, latencia, cancelación posible y manejo de errores a través de esa frontera.

```tsx
export const getCurrentUser = createServerFn().handler(async () => {
  return db.users.me()
})
```

La llamada se siente agradable, pero debajo de esa comodidad sigue existiendo una operación remota. Entenderlo así evita uno de los errores más comunes en sistemas full-stack ergonómicos: actuar como si la sintaxis hubiera eliminado la distancia.

Las server routes sirven para otra clase de necesidad. A veces no queremos una llamada RPC interna tipada, sino una superficie HTTP clara y explícita: un webhook, un endpoint público, un upload, un health check, una integración con terceros que espera hablar el lenguaje clásico de request y response. Allí tiene sentido trabajar con rutas de servidor porque el problema no es tanto invocar lógica interna con comodidad como ofrecer una interfaz HTTP bien definida al mundo externo.

El middleware, por su parte, aparece cuando la preocupación ya no pertenece a una función o a una ruta aislada, sino que atraviesa el sistema. Autenticación, autorización, trazas, logging, inyección de contexto, locale, time zone, manejo uniforme de errores, políticas de seguridad. Todo eso se vuelve torpe si cada pieza debe reescribirlo a mano. Start distingue entre middleware de request y middleware de server functions, y la separación es útil porque no todas las preocupaciones transversales actúan exactamente sobre el mismo ciclo de vida. En cualquier caso, la idea esencial es la misma: ofrecer lugares explícitos para políticas que son estructuralmente compartidas.

Mirado desde arriba, cada una de estas piezas ocupa un lugar muy claro. Las server functions vuelven cómodo el cruce interno cliente-servidor sin negar la red. Las server routes preservan el lenguaje HTTP cuando es ese el lenguaje que importa. El middleware organiza aquello que debe envolver o atravesar muchos puntos del sistema. TanStack Start resulta potente precisamente porque no obliga a resolver problemas distintos con una única herramienta.

### 16. Selective SSR, SPA mode e hidratación honesta

Si una aplicación tiene rutas con necesidades distintas, sería extraño exigir que todas se rendericen bajo la misma política. TanStack Start toma en serio esa diversidad y por eso permite decidir, por ruta, qué papel jugará el servidor en la request inicial. Esa flexibilidad aparece de manera especialmente visible en la opción `ssr`.

Cuando una ruta usa `ssr: true`, el comportamiento es el esperado en un SSR tradicional dentro del marco de Start: durante la request inicial se ejecutan `beforeLoad` y `loader` en el servidor, el componente de la ruta se renderiza allí y el resultado se envía al cliente para que este lo hidrate. Con `ssr: false`, el servidor deja de ejecutar esa parte de la experiencia para la ruta en cuestión y la responsabilidad se desplaza al cliente. Entre ambos extremos existe un caso particularmente interesante: `ssr: 'data-only'`. Allí el servidor resuelve datos y contexto necesarios, pero no renderiza el componente de la ruta. Es una respuesta muy precisa a una necesidad real: quiero llegar al cliente con información ya preparada, pero la UI en sí depende tanto del navegador o aporta tan poco valor al render server-side que no conviene generar su HTML.

La documentación oficial aclara, además, que el árbol hereda restricciones de SSR y solo permite moverse hacia configuraciones más restrictivas. La idea es sensata. Si un padre ya decidió que cierta zona no debe renderizarse en servidor, un hijo no puede deshacer mágicamente esa realidad. También aclara algo importante sobre la raíz: aunque se desactive SSR para la ruta raíz, la shell documental sigue necesitando renderizarse en el servidor. El sistema siempre necesita un armazón de `html`, `head` y `body` sobre el cual arrancar.

Este punto enlaza naturalmente con el SPA mode. En Start, elegir modo SPA no significa renunciar a servidor ni a capacidades full-stack. La documentación es explícita: una app en SPA mode puede seguir usando server functions y server routes. Lo que cambia es la manera en que se produce el documento inicial. En lugar de enviar HTML completamente renderizado para cada request, se sirve una shell estática que luego el cliente completa. El ahorro está en simplificar la política de render; el costo, en general, aparece en el tiempo hasta tener contenido pleno y en ciertas consideraciones de SEO o indexación.

Todo esto lleva inevitablemente a la hidratación y a sus posibles desajustes. Un hydration mismatch no es un capricho de React ni una rareza de consola. Es la señal de que servidor y cliente construyeron árboles distintos a partir de contextos distintos. Fechas, time zones, viewport, `localStorage`, IDs aleatorios, flags dependientes del entorno: todo eso puede romper la coincidencia si no se trata con cuidado. La salida madura rara vez consiste en silenciar el aviso. Lo razonable es volver consistente el contexto cuando eso tiene sentido, persistir lo necesario en cookies o en request context, encapsular ciertas partes en `ClientOnly`, usar `useHydrated` donde la diferencia es inevitable o incluso decidir que cierta ruta no merece SSR completo.

Una pantalla de analítica con gráficos dependientes del tamaño real del viewport muestra bien la cuestión. Los datos pueden llegar perfectamente desde el servidor, pero pretender que la visualización final sea idéntica antes de conocer el entorno cliente puede generar más fricción que valor. Ahí Start permite una gama de decisiones mucho más fina que un simple “SSR sí” o “SSR no”.

### 17. Navegación, datos y frescura: el lugar de TanStack Query

Una de las mejores maneras de entender el ecosistema TanStack es dejar de preguntar qué herramienta “hace fetch” y empezar a preguntar qué problema conceptual resuelve cada una. En ese marco, la relación entre TanStack Start, TanStack Router y TanStack Query se vuelve mucho más clara.

El loader de una ruta responde a una pregunta muy ligada a la navegación: qué necesito saber para que entrar en esta parte de la aplicación tenga sentido. Su momento natural es el ingreso a la ruta o una transición que la involucra. Su mirada es estructuralmente navegacional. No intenta, por sí sola, resolver todo el ciclo de vida del dato; intenta garantizar que la experiencia de esa entrada tenga la base necesaria para existir.

TanStack Query trabaja en otra dimensión. Una vez que cierto dato del servidor ya importa dentro de la experiencia, aparece la necesidad de administrarlo en el tiempo: cuánto dura fresco, cuándo conviene reutilizarlo, cuándo invalidarlo, cómo revalidarlo después de una mutación, cómo compartir su estado entre distintas zonas de la interfaz. Allí Query despliega su fuerza. No organiza la geografía de la aplicación; organiza la vida temporal del estado del servidor.

Pensemos en una ruta `/users/42`. La navegación hacia esa pantalla puede exigir un loader que garantice la existencia del usuario y aporte la información mínima para que la ruta tenga sentido. Una vez dentro, quizá haya paneles secundarios con actividad reciente, permisos, proyectos o métricas que cambian con otra cadencia. Ahí Query puede hacerse cargo de políticas de frescura e invalidación de una manera mucho más rica que un simple “pedí esto al entrar”.

Esta distinción también sugiere una manera sana de ordenar el código. Separar wrappers de server functions en archivos `*.functions.ts`, lógica puramente server-side en `*.server.ts` y tipos o esquemas compartidos en archivos neutrales no es una manía de nombres. Es un intento de hacer visibles las fronteras conceptuales del sistema. Cuando los límites de ejecución y de responsabilidad se expresan también en el código, la arquitectura se vuelve más legible.

Nada de esto obliga a una rigidez dogmática. Habrá casos en los que un loader y Query colaboren estrechamente. Lo importante es no perder la diferencia básica: una cosa ayuda a que la navegación tenga sentido; la otra gobierna la frescura y la memoria del dato a lo largo del tiempo.

## Parte V — Cómo pensar una aplicación real con criterio

### 18. Diseñar desde primeros principios

Después de recorrer la plataforma y el framework, conviene volver a una escena concreta. Supongamos que tenemos que diseñar una aplicación real. La tentación más común es empezar eligiendo stack, librerías y patrones de carpetas. Pero si el objetivo es pensar con criterio, el comienzo tiene que ser otro. Primero hay que describir la experiencia y el problema que la experiencia intenta resolver.

Qué quiere hacer el usuario. Qué partes del sistema son públicas y cuáles exigen identidad. Qué datos deben estar disponibles muy rápido. Qué estados conviene que sean compartibles por URL. Qué operaciones son sensibles. Qué partes dependen del navegador. Qué zonas toleran cierta desactualización y cuáles no. Esa conversación, que a veces parece “menos técnica”, es en realidad la parte más técnica de todas, porque determina la forma que la solución debería tomar.

Después llega el momento de identificar fuentes de verdad. Qué entidades existen y dónde vive su autoridad conceptual. Cuándo estamos ante estado del servidor y cuándo ante estado local de interfaz. Qué información pertenece a la navegación y debería expresarse en la URL. Qué valores pueden derivarse sin convertirse en nuevas fuentes de verdad. Sin este mapa, la aplicación queda condenada a improvisar sincronizaciones y a duplicar autoridad.

Recién entonces tiene sentido distribuir responsabilidades entre cliente y servidor. Algunas cosas pedirán claramente el servidor: secretos, base de datos, validaciones finales, autorización, generación de cierto HTML inicial. Otras pertenecerán al cliente: interacción inmediata, DOM, almacenamiento local, reacciones de alta frecuencia, componentes que dependen del entorno del navegador. Otras estarán en una zona intermedia y exigirán diseño fino: qué datos se resuelven al entrar a una ruta, cuáles se mantienen con políticas de revalidación, qué páginas merecen SSR completo, cuáles solo una shell, cuáles pueden vivir sin él.

En ese punto las herramientas dejan de ser apuestas estéticas y se vuelven respuestas argumentables. TanStack Start resulta convincente cuando ya entendimos que el router debe ocupar un lugar estructural, que el cruce cliente-servidor necesita ergonomía sin negar la red y que las estrategias de render pueden variar por ruta. TanStack Query se vuelve natural cuando ya vimos que el estado del servidor tiene problemas distintos del estado local. Selective SSR deja de ser una feature simpática y se convierte en una decisión pensable.

La mejor señal de que uno está razonando bien no es cuánto recuerda de memoria, sino qué preguntas es capaz de formular. Qué significa realmente esta URL. Dónde está la fuente de verdad de este dato. Qué parte de esta pantalla tiene que llegar temprano. Qué parte depende del navegador. Qué mutación obliga a invalidar qué recuerdos. Dónde está la frontera de confianza en esta operación. Un buen sistema suele ser, antes que nada, el resultado de haberse hecho esas preguntas a tiempo.

### 19. Los errores de razonamiento que más confunden

Hay ciertas confusiones que conviene detectar temprano porque, cuando se consolidan, deforman toda la arquitectura. Una de ellas consiste en tomar una abstracción ergonómica como si hubiera abolido la realidad subyacente. Que una server function se llame como una función y pueda importarse cómodamente no la vuelve local. Que un loader tenga una API agradable no lo convierte automáticamente en código server-only. La buena ergonomía simplifica el trabajo; no altera la física del sistema.

Otra confusión frecuente aparece cuando se mezclan fuentes de verdad diferentes. Guardar en memoria local lo que debería vivir en la URL, tratar como estado de interfaz lo que en realidad es estado del servidor o persistir como si fuera autoridad algo que puede derivarse son errores silenciosos al principio y muy costosos más adelante. La aplicación sigue funcionando un tiempo, pero cada corrección futura se vuelve más frágil porque ya no está claro de dónde debería salir la verdad.

También es común confundir control visual con seguridad real. Si uno piensa que ocultar un botón equivale a proteger la operación, la interfaz empieza a cargar una responsabilidad que no puede cumplir. El cliente puede colaborar con la experiencia de seguridad, pero la decisión efectiva pertenece al entorno que puede validar la request con autoridad.

En torno al render suele aparecer otro tipo de simplificación, casi tribal. SSR, SPA, prerender, `data-only`: a veces se discuten como si fueran bandos con los que conviene identificarse. Pero no son identidades; son respuestas a problemas distintos. La pregunta buena nunca es “qué estrategia es superior en abstracto”, sino qué estrategia paga mejor los costos de esta ruta, de este contenido y de esta interacción.

Algo parecido sucede con el rendimiento. Es fácil dedicar mucha energía a optimizar cálculos locales y olvidar que en la web la red manda sobre gran parte de la experiencia. Un diseño brillante en teoría puede sentirse torpe si obliga a esperar varias veces en cascada. Del mismo modo, usar caché sin una teoría clara sobre frescura e invalidación produce una ilusión de velocidad que luego cobra su deuda en incoherencia.

Por último, hay una ceguera especialmente cara: tratar la observabilidad como si fuera un lujo posterior. Cuando el sistema crece, no observar equivale a renunciar a comprender. Y un sistema que no puede comprenderse termina corrigiéndose por superstición más que por conocimiento.

Reconocer estas confusiones no vuelve automática la solución, pero sí devuelve algo muy valioso: la posibilidad de volver a los principios y reconstruir.

## Conclusión

Aprender web de verdad no consiste en coleccionar tecnologías, sino en entender un pequeño conjunto de ideas estructurales con la suficiente claridad como para que el resto deje de parecer arbitrario. Todo empieza con una situación muy simple: queremos usar, desde nuestra máquina, algo que vive en otra. De esa distancia nacen la URL, HTTP, el navegador, el servidor, la latencia, el caché, la seguridad y el renderizado.

En cuanto esa base se vuelve visible, también se vuelve visible por qué cliente y servidor no son meras carpetas, sino entornos con poderes distintos; por qué renderizar significa repartir trabajo entre lugares y momentos; por qué no todo estado debe tratarse igual; por qué la navegación necesita ocupar un lugar central en la arquitectura; y por qué un framework full-stack no vale tanto por la cantidad de utilidades que ofrece como por la forma en que ordena estas tensiones.

TanStack Start se entiende de verdad desde ese marco. Entonces deja de parecer un conjunto de features con buen marketing y empieza a verse como una propuesta coherente: apoyarse en el router como estructura principal, permitir un modelo de ejecución explícito, ofrecer server functions y middleware para cruzar la frontera full-stack con más claridad, y dar herramientas para decidir por ruta cuánto trabajo conviene pedirle al servidor y cuánto al cliente.

Si este libro cumplió su tarea, el lector no debería sentir solamente que aprendió ciertos conceptos de TanStack Start. Debería sentir algo más sólido: que ahora puede mirar una aplicación web moderna, volver a las preguntas fundamentales y reconstruir desde ahí una explicación razonable. Esa capacidad vale más que cualquier API concreta, porque las APIs cambian. Los buenos principios, en cambio, permiten orientarse incluso cuando cambia el mapa.
