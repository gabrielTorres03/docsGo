# Guia para el recolector de basura Go

## Introduccion

Esta guía tiene como objetivo ayudar a los usuarios avanzados de Go a comprender mejor los costos de sus aplicaciones al brindarles información sobre el recolector de basura de Go. También proporciona orientación sobre cómo los usuarios de Go pueden usar esta información para mejorar la utilización de los recursos de sus aplicaciones. No presupone ningún conocimiento sobre la recolección de basura, pero sí supone familiaridad con el lenguaje de programación Go.

El lenguaje Go se encarga de organizar el almacenamiento de los valores de Go; en la mayoría de los casos, un desarrollador de Go no necesita preocuparse por dónde se almacenan estos valores, o por qué, si es que lo hace. Sin embargo, en la práctica, estos valores a menudo deben almacenarse en la memoria física de la computadora y la memoria física es un recurso finito. Debido a que es finita, la memoria debe administrarse con cuidado y reciclarse para evitar quedarse sin ella mientras se ejecuta un programa Go. El trabajo de una implementación de Go es asignar y reciclar memoria según sea necesario.

Otro término para el reciclaje automático de memoria es recolección de basura . En un nivel alto, un recolector de basura (o GC, por sus siglas en inglés) es un sistema que recicla la memoria en nombre de la aplicación al identificar qué partes de la memoria ya no son necesarias. La cadena de herramientas estándar de Go proporciona una biblioteca de tiempo de ejecución que se entrega con cada aplicación, y esta biblioteca de tiempo de ejecución incluye un recolector de basura.

Tenga en cuenta que la existencia de un recolector de elementos no utilizados como se describe en esta guía no está garantizada por la especificación de Go , solo que el almacenamiento subyacente para los valores de Go lo administra el propio lenguaje. Esta omisión es intencional y permite el uso de técnicas de administración de memoria radicalmente diferentes.

Por lo tanto, esta guía trata sobre una implementación específica del lenguaje de programación Go y puede no ser aplicable a otras implementaciones . En concreto, la siguiente guía se aplica a la cadena de herramientas estándar (el gccompilador y las herramientas de Go). Tanto Gccgo como Gollvm utilizan una implementación de GC muy similar, por lo que se aplican muchos de los mismos conceptos, pero los detalles pueden variar.

Además, este es un documento dinámico y cambiará con el tiempo para reflejar mejor la última versión de Go. Este documento describe actualmente el recolector de elementos no utilizados a partir de Go 1.19.

## Donde viven los valores

Antes de profundizar en el GC, analicemos primero la memoria que no necesita ser administrada por el GC.

Por ejemplo, los valores Go que no sean punteros y que estén almacenados en variables locales probablemente no serán administrados por el GC de Go, y Go, en su lugar, se encargará de que se asigne memoria que esté vinculada al ámbito léxico en el que se creó. En general, esto es más eficiente que depender del GC, porque el compilador de Go puede predeterminar cuándo se puede liberar esa memoria y emitir instrucciones de máquina que la limpien. Normalmente, nos referimos a la asignación de memoria para valores Go de esta manera como "asignación de pila", porque el espacio se almacena en la pila de goroutines.

Los valores de Go cuya memoria no se puede asignar de esta manera, porque el compilador de Go no puede determinar su duración, se dice que escapan al montón . "El montón" se puede considerar como un cajón de sastre para la asignación de memoria, para cuando los valores de Go deben colocarse en algún lugar . El acto de asignar memoria en el montón se conoce normalmente como "asignación de memoria dinámica" porque tanto el compilador como el entorno de ejecución pueden hacer muy pocas suposiciones sobre cómo se utiliza esta memoria y cuándo se puede limpiar. Ahí es donde entra en juego un GC: es un sistema que identifica y limpia específicamente las asignaciones de memoria dinámica.

Existen muchas razones por las que un valor de Go podría necesitar escapar al montón. Una razón podría ser que su tamaño se determina dinámicamente. Considere, por ejemplo, la matriz de respaldo de una porción cuyo tamaño inicial está determinado por una variable, en lugar de una constante. Tenga en cuenta que el escape al montón también debe ser transitivo: si se escribe una referencia a un valor de Go en otro valor de Go que ya se ha determinado que escape, ese valor también debe escapar.

El hecho de que un valor de Go se escape o no depende del contexto en el que se utiliza y del algoritmo de análisis de escape del compilador de Go. Sería frágil y difícil intentar enumerar con precisión cuándo se escapan los valores: el algoritmo en sí es bastante sofisticado y cambia entre las distintas versiones de Go. Para obtener más detalles sobre cómo identificar qué valores se escapan y cuáles no, consulte la sección sobre la eliminación de asignaciones de montón .

## Seguimiento de la recoleccion de basura.

La recolección de basura puede hacer referencia a muchos métodos diferentes de reciclaje automático de la memoria; por ejemplo, el conteo de referencias. En el contexto de este documento, la recolección de basura se refiere al seguimiento de la recolección de basura, que identifica objetos en uso, los llamados objetos activos , siguiendo punteros de manera transitiva.

### Definamos estos términos con más rigor.

1. Objeto: un objeto es una pieza de memoria asignada dinámicamente que contiene uno o más valores Go.

2. Puntero: dirección de memoria que hace referencia a cualquier valor dentro de un objeto. Esto incluye naturalmente valores Go de la forma *T, pero también incluye partes de valores Go integrados. Las cadenas, los sectores, los canales, los mapas y los valores de interfaz contienen direcciones de memoria que el GC debe rastrear.

Juntos, los objetos y los punteros a otros objetos forman el gráfico de objetos . Para identificar la memoria activa, el GC recorre el gráfico de objetos comenzando por las raíces del programa , punteros que identifican objetos que el programa definitivamente está utilizando. Dos ejemplos de raíces son las variables locales y las variables globales. El proceso de recorrer el gráfico de objetos se conoce como escaneo.

Este algoritmo básico es común a todos los GC de rastreo. En lo que se diferencian los GC de rastreo es en lo que hacen una vez que descubren que la memoria está activa. El GC de Go utiliza la técnica de barrido de marcas, lo que significa que para realizar un seguimiento de su progreso, el GC también marca los valores que encuentra como activos. Una vez que se completa el rastreo, el GC recorre toda la memoria del montón y hace que toda la memoria que no está marcada esté disponible para su asignación. Este proceso se denomina barrido .

Una técnica alternativa con la que quizás esté familiarizado es mover los objetos a una nueva parte de la memoria y dejar atrás un puntero de reenvío que se utiliza más adelante para actualizar todos los punteros de la aplicación. A un GC que mueve objetos de esta manera lo llamamos GC en movimiento ; Go tiene un GC inmóvil.

## El ciclo GC

Debido a que el GC Go es un GC de barrido de marcas, opera en dos fases: la fase de marca y la fase de barrido. Si bien esta afirmación puede parecer tautológica, contiene una idea importante: no es posible liberar memoria para que se asigne hasta que se haya rastreado toda la memoria, porque aún puede haber un puntero sin escanear que mantenga vivo un objeto. Como resultado, el acto de barrido debe estar completamente separado del acto de marcado. Además, el GC también puede no estar activo en absoluto, cuando no hay trabajo relacionado con el GC que hacer. El GC rota continuamente a través de estas tres fases de barrido, apagado y marcado en lo que se conoce como el ciclo del GC . Para los fines de este documento, considere el ciclo del GC comenzando con el barrido, el apagado y luego el marcado.

Las siguientes secciones se centrarán en generar intuición sobre los costos del GC para ayudar a los usuarios a ajustar los parámetros del GC para su propio beneficio.

### Entendiendo los costos

El GC es inherentemente un software complejo que se basa en sistemas aún más complejos. Es fácil perderse en los detalles cuando se intenta comprender el GC y modificar su comportamiento. Esta sección tiene como objetivo proporcionar un marco para razonar sobre el costo del GC Go y los parámetros de ajuste.

Para empezar, considere este modelo de costo de GC basado en tres axiomas simples.

1. La GC involucra solo dos recursos: tiempo de CPU y memoria física.

2. Los costos de memoria del GC consisten en memoria de montón activa, nueva memoria de montón asignada antes de la fase de marcado y espacio para metadatos que, incluso si son proporcionales a los costos anteriores, son pequeños en comparación.

Nota: la memoria de montón activa es la memoria que se determinó como activa en el ciclo de GC anterior, mientras que la memoria de montón nueva es cualquier memoria asignada en el ciclo actual, que puede o no estar activa al final.

3. Los costos de CPU del GC se modelan como un costo fijo por ciclo y un costo marginal que escala proporcionalmente con el tamaño del montón activo.

Nota: Asintóticamente hablando, el barrido es peor que el marcado y el escaneo, ya que debe realizar un trabajo proporcional al tamaño de todo el montón, incluida la memoria que se determina que no está activa (es decir, "muerta"). Sin embargo, en la implementación actual, el barrido es mucho más rápido que el marcado y el escaneo, por lo que sus costos asociados se pueden ignorar en esta discusión.

Este modelo es simple pero efectivo: categoriza con precisión los costos dominantes del GC. Sin embargo, este modelo no dice nada sobre la magnitud de estos costos ni sobre cómo interactúan. Para modelarlo, considere la siguiente situación, a la que de aquí en adelante se denominará estado estacionario.

* La velocidad a la que la aplicación asigna nueva memoria (en bytes por segundo) es constante.

Nota: es importante entender que esta tasa de asignación es completamente independiente de si esta nueva memoria está activa o no. Ninguna de ellas podría estar activa, toda podría estar activa o parte de ella podría estar activa. (Además de esto, alguna memoria del montón antigua también podría morir, por lo que no es necesariamente el caso de que si esa memoria está activa, el tamaño del montón activo crezca).

Para decirlo de forma más concreta, considere un servicio web que asigna 2 MiB de memoria de montón total para cada solicitud que maneja. Durante la solicitud, como máximo 512 KiB de esos 2 MiB permanecen activos mientras la solicitud está en tránsito, y cuando el servicio termina de manejar la solicitud, toda esa memoria muere. Ahora, para simplificar, supongamos que cada solicitud tarda aproximadamente 1 segundo en procesarse de extremo a extremo. Luego, un flujo constante de solicitudes, digamos 100 solicitudes por segundo, da como resultado una tasa de asignación de 200 MiB/s y un montón activo máximo de 50 MiB.

* El gráfico de objetos de la aplicación se ve aproximadamente igual cada vez (los objetos tienen un tamaño similar, hay una cantidad aproximadamente constante de punteros, la profundidad máxima del gráfico es aproximadamente constante).

Otra forma de pensar en esto es que los costos marginales de GC son constantes.

Nota: el estado estable puede parecer artificial, pero es representativo del comportamiento de una aplicación bajo una carga de trabajo constante. Naturalmente, las cargas de trabajo pueden cambiar incluso mientras se ejecuta una aplicación, pero, por lo general, el comportamiento de la aplicación parece un conjunto de estos estados estables unidos con algún comportamiento transitorio entre ellos.

Nota: el estado estable no hace suposiciones sobre el montón en vivo. Puede crecer con cada ciclo de recolección de basura subsiguiente, puede reducirse o puede permanecer igual. Sin embargo, tratar de abarcar todas estas situaciones en las explicaciones que siguen es tedioso y no muy ilustrativo, por lo que la guía se centrará en ejemplos en los que el montón en vivo permanece constante. La sección GOGC explora el escenario del montón en vivo no constante con más detalle.

En el estado estable, mientras el tamaño del montón en vivo es constante, cada ciclo de recolección de basura se verá idéntico en el modelo de costos siempre que la recolección de basura se ejecute después de que haya transcurrido la misma cantidad de tiempo. Esto se debe a que en esa cantidad fija de tiempo, con una tasa fija de asignación por parte de la aplicación, se asignará una cantidad fija de nueva memoria de montón. Por lo tanto, con el tamaño del montón en vivo constante y esa nueva memoria de montón constante, el uso de memoria siempre será el mismo. Y debido a que el montón en vivo tiene el mismo tamaño, los costos marginales de CPU de la recolección de basura serán los mismos y los costos fijos se incurrirán en algún intervalo regular.

Ahora, pensemos en lo que ocurriría si el GC cambiara el punto en el que se ejecuta más adelante . En ese caso, se asignaría más memoria, pero cada ciclo del GC seguiría teniendo el mismo coste de CPU. Sin embargo, en otro intervalo de tiempo fijo, finalizarían menos ciclos del GC, lo que daría como resultado un menor coste total de CPU. Lo opuesto sería cierto si el GC decidiera comenzar antes : se asignaría menos memoria y se incurriría en costes de CPU con mayor frecuencia.

Esta situación representa el equilibrio fundamental entre el tiempo de CPU y la memoria que puede alcanzar un GC, controlado por la frecuencia con la que se ejecuta realmente el GC. En otras palabras, el equilibrio está completamente definido por la frecuencia del GC .

Queda por definir un detalle más, que es cuándo debe decidir comenzar el GC. Tenga en cuenta que esto establece directamente la frecuencia del GC en cualquier estado estable particular, lo que define el equilibrio. En Go, decidir cuándo debe comenzar el GC es el parámetro principal sobre el que tiene control el usuario.
