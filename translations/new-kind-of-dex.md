# Un Nuevo Tipo de DEX

![Dex-header](https://github.com/francov99/dcrspanish/blob/master/assets/dex-header.png)

*Este artículo fue redactado originalmente por JAKE YOCOM-PIATT en [Inglés](https://blog.decred.org/2018/06/05/A-New-Kind-of-DEX)*

El exchange descentralizado ("DEX") es un concepto que ha recibido una atención creciente en el campo de las criptomonedas como resultado de los exchanges hackeados, utilizados como estafas o sujetos a acciones regulativas. Existen varios proyectos con la intención de reemplazar los típicos exchanges centralizados con un token o una cadena de bloques. Proponemos una alternativa a los proyectos existentes con las siguientes propiedades:

- Facilitar el intercambio solo entre criptomonedas, no monedas fiduciarias.
- Diseñar como un simple cliente y servidor, sin un token correspondiente o una cadena de bloques.
- Los operadores del servidor nunca toman la custodia de los fondos del cliente.
- Utilizar transacciones en cadena para el cumplimiento de pedidos y la aplicación de normas.
- Los operadores del servidor no cobran alguna tarifa por los intercambios.
- Agregar soporte para monedas es solo una cuestión sencilla de agregar el soporte de intercambio atómico correspondiente.
- Las órdenes realizadas en el exchange se pueden regular internamente a través de reglas aplicadas por los clientes y el servidor.
- Los clientes maliciosos se gestionan utilizando un sistema de reputación basado en Politeia.
- Hay una tarifa inicial para crear una cuenta de cliente en un servidor, para desalentar el comportamiento malicioso.
- La coincidencia de órdenes ocurre pseudoaleatoriamente.
- Los tamaños de los pedidos tanto en el lado de compra como de venta de un par comercial tienen tamaños de lote estandarizados.
- Los clientes emiten órdenes limitadas y cancelaciones a través del servidor, pero las órdenes de mercado se enrutan de cliente a cliente.
- El intercambio casi instantáneo para pedidos más pequeños se puede lograr a través de una red relacionada fuera de la cadena basada en LN que utiliza intercambios atómicos.
- Los servidores pueden conectarse a través de una red de malla para permitir la coincidencia de pedidos entre servidores.
- Servicios externos, p. carteras, puede acceder a una API de cliente simple en el servidor que proporciona una fuente de datos, la capacidad de realizar órdenes y otros servicios.

Creo que esta infraestructura tiene la capacidad de mejorar la capacidad de recuperación en los ecosistemas cripto, y los mercados de Decred más específicamente. A continuación, explicaré las diversas consideraciones que nos han llevado a proponer la arquitectura resumida anteriormente.

## Motivación

Cualquiera que esté familiarizado con los procesos, usar un exchange centralizado o conocer a un proyecto de criptomoneda agregado a uno sabe que hay numerosos obstáculos que atravesar, ya que tomar la custodia de los fondos de los clientes requiere el cumplimiento de diversas regulaciones y agencias reguladoras específicas, p. registrarse con FinCEN, la Ley de secreto bancario, implementar políticas KYC / AML. Para los usuarios, estar sujeto a KYC / AML es una invasión sustancial de la privacidad y los fondos de los clientes en el exchange pueden congelarse en cualquier momento, por una variedad de razones. Los proyectos que buscan la adición de su criptomoneda a los exchanges a menudo se ven presionados a pagar grandes tarifas de cotización a pesar de que el trabajo para agregar soporte es sencillo, y muchos proyectos más grandes solo están interesados ​​en cotizar criptomonedas que creen que generarán la mayor ganancia a través de las tarifas comerciales.

Se han creado varios proyectos DEX para abordar algunos de estos problemas al reemplazar el exchange con una cadena de bloques o un token, y se han encontrado con diversos grados de éxito. Si bien eliminan al tercero de confianza ("TTP"), insertan sus propios productos como un medio para capturar las tarifas comerciales, lo que reemplaza la fricción de TTP con una nueva fricción de plataforma. El simple acto de cobrar tarifas comerciales sirve como un incentivo para centralizar una solución dada, que va en contra de un sistema de intercambio voluntario abierto. Al mismo tiempo, una cadena o token sirve para eliminar el TTP, también crea desafíos con la coincidencia de órdenes, que generalmente ocurre a través de la cadena o token intermedio.

Además de todos los desafíos presentes desde el punto de vista regulatorio, la experiencia del usuario y las perspectivas tecnológicas, en los exchanges y los proyectos DEX existentes, junto con sus ecosistemas correspondientes, son vulnerables a ser explotados por algoritmos comerciales de alta frecuencia ("HFT"). Las HFT se han convertido en una influencia cada vez más grande en los mercados de acciones, materias primas y divisas desde la década de 1990, y se han producido varios "[flash crashes](https://es.wikipedia.org/wiki/Flash_Crash_de_2010)" notables como resultado de que las HFT extrajeron su efímera liquidez simultáneamente. Los HFT generalmente mantienen al mínimo las posiciones que tienen abiertas en un momento dado, pero abren y cierran estas posiciones con una frecuencia muy alta. Aprovechan las políticas de coincidencia de orden de primero en entrar, primero en salir ("FIFO") en los intercambios y toman medidas extremas para tener una latencia más baja que su competencia, p. hacen que los HFT ejecuten operaciones desde máquinas muy cercanas a los servidores del exchange, pagan grandes sumas por los datos de latencia más bajos, usan FPGA o ASIC para puertas de enlace de pedidos, usan relés de microondas o láser para reducir la latencia entre intercambios. Si bien hay intentos de caracterizar las HFT como positivas, p. que agregan liquidez al mercado, que la liquidez puede desaparecer en un abrir y cerrar de ojos y actúa como una bomba para desviar fondos de los participantes del mercado menos sofisticados a aquellos que operan las HFT. Más allá de la cuestión de si es justo operar estas HFT, está claro que los operadores de HFT pueden distorsionar seriamente el precio de los activos que comercializan, saboteando el proceso de descubrimiento de precios. Muchas de estas HFT son operadas y / o financiadas por los brazos de banca de inversión de los principales bancos internacionales, lo que significa que a pesar de que las criptomonedas eliminan la necesidad de usar bancos para el almacenamiento y la transmisión de valor, el sector de la banca fiduciaria aún puede ejercer una influencia sustancial sobre las criptomonedas a través de sus HFTs.

![Consideraciones-practicas](https://github.com/francov99/dcrspanish/blob/master/assets/dex-practical-considerations.png)

## Consideraciones prácticas

Para que un DEX funcione, hay una serie de consideraciones prácticas básicas que deben tenerse en cuenta. Mediante el uso de intercambios atómicos, es posible realizar intercambios sin confianza de criptomonedas compatibles, tanto dentro como fuera de la cadena. Para generar y mantener un libro de pedidos, debe haber un lugar de reunión donde los usuarios puedan comunicarse sobre los precios. Para evitar que los usuarios envíen órdenes fraudulentas, se necesita un mecanismo para que los usuarios demuestren que controlan los fondos a los que corresponden sus órdenes. Los usuarios deben poder transmitir y recibir órdenes de límite y de mercado para que puedan ser igualadas. Debido a las restricciones de las transacciones en cadena, las órdenes limitadas deben tener un tamaño estandarizado en cadena. Es posible comenzar con una arquitectura simple de cliente-servidor y luego extenderla haciendo que los servidores transmitan órdenes entre sí, creando una malla.

## Intercambios Atómicos

Como demostramos con nuestras herramientas de [intercambio atómico(En inglés: Atomic Swap)](https://github.com/decred/atomicswap) en el otoño de 2017, utilizando un contrato inteligente de complejidad mínima particular, es posible intercambiar dos criptomonedas separadas sin permiso, y este proceso se conoce como un intercambio atómico. Debido a la naturaleza confiada del intercambio atómico, es una base natural sobre la cual construir un DEX. Varios otros proyectos hacen uso de intercambios atómicos precisamente por esta razón. Un intercambio atómico puede manejar la liquidación de pedidos coincidentes tanto dentro como fuera de la cadena, por lo que la tarea de construir un DEX se convierte en una cuestión de manejar el proceso de coincidencia de pedidos.

## Arquitectura cliente-servidor

Usando los intercambios atómicos como el bloque de construcción básico, podemos comenzar a discutir los contornos generales de la arquitectura de un DEX. Varios proyectos existentes han tomado la decisión de reemplazar el exchange con una cadena de bloques independiente o un token de Ethereum. En cada caso, estos proyectos utilizan los honorarios asociados con la ejecución del comercio como un medio para extraer valor del proceso de intercambio. Sin embargo, el lector observador notará que los intercambios atómicos facilitan el intercambio sin requerir ningún tercero de confianza ("TTP"). Tomamos la posición de que, en lugar de recrear un TTP como blockchain o token, que la ruta más lógica es crear un DEX como un simple servicio cliente-servidor que elimina por completo la necesidad de cualquier TTP. Las cadenas de bloques y los tokens son soluciones muy "pesadas" e imponen una variedad de restricciones de ingeniería, por lo que evitarlas nos libera de estas restricciones. En su forma más básica, un DEX es simplemente un lugar de encuentro donde los usuarios que desean intercambiar criptomonedas se encuentran y participan en algún proceso de descubrimiento de precios. El uso de un modelo cliente-servidor para un DEX nos permite recrear la forma más básica de intercambio humano en un contexto sin permiso: un pozo de protesta abierto bajo demanda.

## Verificación de la orden

Un DEX requiere soporte para 2 tipos de órdenes: limitadas y de mercado. Las órdenes limitadas son órdenes permanentes para comprar o vender a un precio en particular, y las órdenes de mercado son emitidas por compradores y vendedores al contado para obtener el mejor precio. El servidor puede actuar como un relé tonto, donde simplemente retransmite las órdenes enviadas por los clientes, o puede ensamblar y mantener el libro de pedidos, por lo que solo envía actualizaciones al libro de pedidos a medida que los clientes transmiten los pedidos. En cualquier caso, las órdenes de límite se transmitirán a otros clientes, mientras que las órdenes de mercado se enrutarán directamente a los clientes con las órdenes de límite coincidentes. Para evitar que el servidor falsifique el libro de pedidos, el servidor solo transmitirá pedidos firmados a los clientes, para que puedan verificar por sí mismos que los pedidos en el libro son válidos.

## Tamaños de lote estandarizado

Un detalle importante a considerar con un DEX es cómo manejar rellenos parciales, p. hay una orden de compra límite para DCR de  100 BTC y hay órdenes de venta en el mercado que se comparan con 0.1 BTC de DCR. Si permitimos órdenes de mercado y límites de tamaño arbitrario, crea problemas porque el cliente que se llena parcialmente debe crear una transacción en cadena para actualizar su orden firmada, que puede tomar varios minutos o más. Más explícitamente, suponga que tenemos una orden de compra límite para 100 BTC, se realiza un llenado parcial con una venta de mercado de 0.1 BTC ahora que el cliente con la orden de compra límite tiene que hacer un cambio desde su orden original para liquidar el llenado parcial utilizando un intercambio atómico, obligándolos a esperar hasta que el 99.9 BTC se pueda hacer en un nuevo pedido y volver a enviar al DEX. Para evitar este problema, el DEX se decidirá por tamaños estandarizados para lotes en el lado de compra y venta de cada par comercial, donde los clientes deben realizar pedidos utilizando estos tamaños de lote.

## Servidor en malla

El producto mínimo viable para el DEX tiene una arquitectura simple de cliente-servidor, pero ese modelo tiene algunas deficiencias que pueden abordarse creando una malla de servidores que transmiten órdenes entre sí. Agregar soporte para la comunicación de servidor a servidor obviamente no es trivial, pero permitiría la creación de pares comerciales mundiales, con la ventaja de que sería estable en caso de pérdida o desconexión de grandes segmentos de la red, a diferencia de muchas cadenas de bloques.

## Alineación de incentivos

Las criptomonedas han servido para demostrar el inmenso valor que se puede extraer de los incentivos alineados adecuadamente, y nuestro objetivo es reproducir esta alineación de incentivos en el contexto del DEX. Para las criptomonedas, la alineación de incentivos está relacionada con los subsidios en bloque y cómo distribuirlos, y con el DEX, se trata de cómo lidiar con las tarifas comerciales, la custodia de fondos y las tarifas de las cuentas. Como participantes tardíos en el espacio DEX, tenemos el lujo de ver cómo otros proyectos DEX han abordado la cuestión de los incentivos, en lugar de tener que innovar en el vacío.

![Zero-fees](https://github.com/francov99/dcrspanish/blob/master/assets/dex-zero-trading-fees.png)

## Cero comisiones por intercambio

Los proyectos DEX existentes proyectan todas las tarifas de extracción de cada operación a través de su correspondiente blockchain o token, pero tomamos la posición de que esta es la estructura de incentivos incorrecta. El cobro de tarifas en las operaciones incentiva directamente la centralización de las operaciones por parte del proveedor del servicio, ya sea una cadena de bloques, un token o un servicio. Para realinear los incentivos con un intercambio verdaderamente descentralizado, debemos eliminar las tarifas comerciales del DEX. Es probable que muchos usuarios potenciales estén felices de escuchar esto, pero muchos proyectos DEX o intercambios centralizados no están interesados ​​en ver que esto se sugiera. Es razonable preguntar "si no hay tarifas comerciales, ¿cuál es el incentivo para ejecutar un servidor DEX?", Y la respuesta es similar a la respuesta a preguntas como "si nadie le paga por operar un correo, chat o servidor blockchain, ¿cuál es el incentivo para ejecutarlo?”. Ejecutar un servidor DEX le brinda a usted y a otros una utilidad tangible, que es la fungibilidad de bajo costo en las cadenas de bloques. Afortunadamente, no todos necesitan operar un servidor DEX para que esta infraestructura brinde una utilidad sustancial.

## Servidores sin custodia

Si bien los intercambios atómicos eliminan la necesidad de que el servidor tome la custodia de los fondos, el servidor es otro ejemplo de alineación de incentivos. Cuando un intercambio toma la custodia de los fondos, sus incentivos cambian bastante. Se incentiva a no perder esos fondos desde un punto de vista reputacional y regulatorio, pero también se incentiva a tomar otras acciones (indeseables) como resultado de esas preocupaciones, p. Ej. recopilar una gran cantidad de información personal de los usuarios, rastrear el origen de los fondos de los usuarios, compartir información con los funcionarios encargados de hacer cumplir la ley, retrasar y / o limitar los retiros y depósitos. Más allá de estos incentivos de riesgo de pérdida, siempre hay incentivos presentes para que los intercambios sean malos actores, p. robar fondos de los usuarios, manipular el libro de pedidos, ejecutar bots contra sus propios libros, proporcionar a los usuarios seleccionados cuentas gratuitas, proporcionar ventajas de latencia a los usuarios seleccionados. Los servidores DEX que no toman la custodia de los fondos descartan la mayoría de estos diversos incentivos indeseables.

## Tarifa de creación a la cuenta del cliente

Para evitar que los clientes malintencionados causen problemas en los servidores del DEX, tendremos una pequeña tarifa configurable por el servidor para crear una nueva cuenta de cliente. Para los clientes benevolentes, esto equivale a una tarifa única que deben pagar para acceder a los servicios del servidor, pero los clientes malintencionados tendrán que pagar esta tarifa repetidamente si causan problemas y tienen su cuenta desactivada. Estas tarifas no se ampliarán con el volumen y están destinadas a actuar como un elemento disuasorio de spam, lo que puede compensar algunos de los costos de operación del servidor del DEX.

![transparencia-como-herramienta](https://github.com/francov99/dcrspanish/blob/master/assets/dex-transparency-as-a-tool.png)

## La transparencia como herramienta

Hasta la fecha, la mayoría de los participantes en el espacio blockchain han visto la transparencia de blockchains como una debilidad necesaria, pero estaremos usando esta transparencia como una herramienta constructiva en el contexto del DEX. Al realizar intercambios en cadena y utilizar la certificación criptográfica, tanto los clientes como los servidores pueden ser responsables de comportamientos maliciosos. La prueba criptográfica del comportamiento malicioso de un cliente o servidor se enviará a un repositorio de Politeia, creando un sistema de reputación verificable independientemente para el DEX. Más significativamente, es posible crear el equivalente de las reglas de consenso para uno o más servidores del DEX, lo que permite que el servidor y los clientes dicten y apliquen lo que constituye un orden aceptable, que es una especie de regulación descentralizada. Otro efecto secundario beneficioso del uso de la transparencia en la cadena es que hace que sea mucho más difícil reducir el volumen "falso", y aquellos que participan en prácticas comerciales cuestionables pueden ser identificados.

## Transacciones en cadena para la rendición de cuentas

Cuando se realizan intercambios atómicos en cadena, es posible que un cliente a cada lado del intercambio se comporte de manera maliciosa, por lo que es necesario tener responsabilidad en este proceso. Del mismo modo, es posible que los operadores del servidor DEX se comporten de manera maliciosa, y los clientes pueden demostrar su acción maliciosa por mérito de los recibos criptográficos de los pedidos enviados y coincidentes. Una combinación de transacciones en cadena y recibos criptográficos actuará como la base de un sistema de reputación para el DEX.

## Sistemas de reputación basados en Politeia

Este sistema de reputación se basará en Politeia, el sistema de archivos basado en git ordenado por tiempo de Decred, y dejará en claro que un cliente o servidor determinado se comportó mal en una fecha y hora en particular. Es posible tener múltiples sistemas de reputación y que sean públicos o privados, según el caso de uso. Los clientes y servidores utilizarán estos servicios de reputación como base para aceptar o rechazar pedidos.

## Regulación descentralizada

Si bien el DEX no será una cadena de bloques, es posible crear reglas que tanto los clientes como los servidores de la red apliquen voluntariamente. Según los comentarios anteriores sobre el formato de los pedidos, hay ciertas reglas que se deben cumplir para que el DEX funcione correctamente, p. que las órdenes deben corresponder a mensajes firmados que demuestren el control de una cantidad correspondiente de monedas no gastadas. Dado que el DEX utilizará transacciones en cadena, es posible establecer restricciones arbitrarias adicionales sobre lo que constituye un pedido válido por parte de clientes y / o servidores, p. que un pedido válido requiere que las monedas no se hayan movido en la cadena durante más de 24 horas. Es importante que los clientes y los servidores acuerden lo que constituye un pedido válido o de lo contrario podría causar problemas al hacer coincidir los pedidos, por lo que estas reglas adicionales son análogas a las reglas de consenso de blockchain, que deben ser las mismas en una cadena de bloques dada para la estabilidad / para evitar hard forks. Incluso reglas adicionales muy simples, p. Los pedidos solo se consideran válidos si las monedas no se han movido en la cadena durante 24 horas o más, pueden tener consecuencias significativas para el DEX, p. limitar a los clientes a batir sus monedas solo una vez al día. Estas reglas adicionales aplicadas voluntariamente constituyen una forma de regulación descentralizada, que permitirá que el DEX bloquee comportamientos considerados inaceptables por sus clientes y servidores, p. suplantación excesiva.

## Volumen Verificable

Un problema importante con los exchanges existentes es el volumen falso. Es sencillo configurar un exchange y luego falsificar por completo los datos de volumen o tener un bot. Los servicios de intercambio centralizado no tienen forma de demostrar que las transacciones que se realizan en ellos son "reales" y no están falsificadas o no son válidas, ya que las órdenes se realizan fuera de la cadena y no pueden verificarse con un libro mayor distribuido. Al hacer que el DEX funcione en cadena, los datos de volumen se pueden verificar externamente contra las cadenas de bloques correspondientes y los intercambios atómicos que ocurren allí. Además, los intentos de manipular el comercio pueden reconocerse y filtrarse por el mérito de su historial de transacciones en la cadena. Si los clientes y los servidores lo consideran adecuado, pueden implementar reglas para evitar la manipulación del comercio en sus diversas formas.

![sin-barreras](https://github.com/francov99/dcrspanish/blob/master/assets/dex-lowering-barriers.png)

## Reducción de la Barrera de Entrada

Existen muchos obstáculos tanto para operar un exchange como para ser un cliente, por lo que el DEX se construirá con el objetivo de simplificar ambos procesos. Según comentarios anteriores, el DEX tendrá una arquitectura simple de cliente-servidor, que no solo tiene sentido desde una perspectiva de implementación práctica, sino también desde la perspectiva de reducir las barreras de entrada. Obtener una criptomoneda en particular compatible con el DEX es una simple cuestión de agregar soporte para intercambios atómicos, que pueden hacer los desarrolladores asociados con el proyecto en lugar del servicio de intercambio. Una fuente de datos para cada servidor estará disponible para sus clientes, y opcionalmente puede ofrecer datos históricos sobre operaciones y almacenarse en un repositorio de Politeia.

## Configuración simple del cliente-servidor

Para mantener el DEX descentralizado, es ideal simplificar la configuración del servidor y agregarlo como cliente. La configuración para alternativas al DEX es engorrosa, p. crear y mantener una cadena de bloques o configurar un servicio con restricciones regulatorias KYC / AML. Si bien es poco probable, es posible bloquear el acceso a una cadena de bloques determinada dentro de las fronteras de los estados nacionales, y lo mismo ocurre con cualquier servicio de intercambio. Tener una arquitectura cliente-servidor con una configuración sencilla no solo facilita la configuración de nuevos servidores y clientes, sino que también mejora la resistencia a la censura.

## Agregar soporte a través de intercambios atómicos

Obtener una nueva criptomoneda compatible con los intercambios suele ser un ejercicio de paciencia, política, pago o alguna combinación de los mismos. El proceso actual para obtener cotizaciones en los exchanges es una verdadera pérdida de recursos de un proyecto y establece restricciones artificiales a la liquidez a la que tiene acceso un proyecto determinado. El único requisito para admitir una criptomoneda en particular en el DEX será el intercambio atómico correspondiente, eliminando los obstáculos políticos y de procedimiento existentes asociados con la inclusión en diversos servicios. La simplicidad para obtener acceso a la liquidez del intercambio ayudará a nivelar el campo de juego entre los proyectos de criptomonedas, donde algunos proyectos pueden obtener una ventaja sustancial sobre otros a través de sus listados de intercambio.

## API del Cliente

Las fuentes de datos estarán disponibles para los clientes desde sus respectivos servidores y se utilizarán para datos tanto en tiempo real como históricos. La API permitirá a los clientes obtener una instantánea de la cartera de pedidos, suscribirse para recibir actualizaciones de la cartera de pedidos y acceder a datos históricos. Dado que estos datos provienen del servidor, la cantidad de datos históricos y en tiempo real variará de un servidor a otro. Es probable que la calidad y la cantidad de datos disponibles aumenten sustancialmente una vez que los servidores puedan retransmitirse entre sí. Las carteras que admiten el DEX utilizarán la API del cliente para fines de interfaz de usuario.

## Juegos de latencia

A medida que los mercados de criptomonedas han crecido en tamaño y madurez, se han convertido en un objetivo maduro para los algoritmos de negociación automatizados, en particular los algoritmos de negociación de alta frecuencia ("HFT"). Los HFT aumentan sustancialmente los volúmenes de negociación citados por los exchanges, pero el valor de este volumen es cuestionable, en el mejor de los casos. El DEX utilizará la coincidencia de pedidos pseudoaleatoria dentro de las épocas para limitar los efectos de los HFT, al tiempo que permite que los clientes de menor frecuencia consigan que sus pedidos coincidan de manera justa. Vale la pena mencionar que los sistemas de pago fuera de la cadena, p. Lightning Network se puede utilizar para el intercambio, pero es importante comprender que existen problemas con la transparencia y el descubrimiento de precios, similares a los exchanges existentes de acciones o commodities.

![seudoaleatorio](https://github.com/francov99/dcrspanish/blob/master/assets/dex-pseudorandom-matching.png)

## Cruce a de orden pseudoaleatorio

La medida en que los mercados financieros modernos están dominados por los HFT no se discute a menudo. Hacer cambios para reducir la influencia de los HFT es aún menos discutido ya que los intercambios son incentivados para permitir su operación por el mérito de las tarifas comerciales que generan. Algunos intercambios tradicionales, p. [IEX y CHX](https://www.bloomberg.com/view/articles/2016-08-31/speed-bumps-are-the-hot-new-thing-for-exchanges) han agregado un “aumento de velocidad” de 350 microsegundos para evitar el arbitraje de latencia por debajo de este umbral, y esto es parte del camino para solucionar el problema. En lugar de abordar solo parcialmente el arbitraje de latencia, que todavía es posible frente a un mínimo de 350 microsegundos, el DEX hará coincidir las órdenes de manera seudoaleatoria dentro de los tiempos. El uso de un tiempo 10 segundos o más debería actuar para reducir sustancialmente las ventajas que pueden obtener los clientes con conexiones de baja latencia a su servidor. Al abandonar un algoritmo de cruce de orden: primero en entrar, primero en salir ("FIFO"), los clientes pueden hacer coincidir sus órdenes de una manera demostrablemente justa dentro del tiempo, mientras se evita el comportamiento de "corte en cola" requerido para la operación exitosa de la mayoría de los HFT.

## Subsistema fuera de la cadena

Considerando las latencias mucho más bajas involucradas con los sistemas de pago fuera de la cadena, p. la Red Lightning, es natural examinar su utilidad en el contexto del DEX. Una de las principales ventajas del funcionamiento en cadena del DEX es que los swaps son demostrables y pueden verificarse. Como Lightning Labs ha demostrado, [los intercambios atómicos fuera de la cadena son posibles](https://blog.lightning.engineering/announcement/2017/11/16/ln-swap.html), pero la verificación de cruces ocurrió y los cruces no son intercambios de lavado es un proceso no trivial y probablemente muy difícil. A la luz de estos problemas con la transparencia y el descubrimiento de precios que vienen con los intercambios de LN, el DEX podría incorporar intercambios fuera de la cadena como un medio para que los pedidos más pequeños se llenen con baja latencia. En lugar de tener un libro de pedidos fuera de la cadena, los pedidos se pueden completar a los precios de mercado establecidos por la cadena DEX, donde la transparencia y el descubrimiento de precios pueden ocurrir con menos complejidad.

![El-panorama](https://github.com/francov99/dcrspanish/blob/master/assets/dex-big-picture.png)

## El panorama

Gran parte de esta propuesta ha sido sobre las consideraciones prácticas de cómo debería funcionar un DEX ideal y cómo se puede lograr usando la infraestructura de criptomonedas existente. Dando un paso atrás, lo que se propone aquí es una infraestructura de multiplexación para el intercambio de criptomonedas. De la misma manera que el teléfono y los conmutadores de red multiplexan las comunicaciones electromagnéticas, las criptomonedas y su intercambio multiplexan el almacenamiento, la transmisión y el intercambio de valor. EL daemon de Lightning Network, [lnd](https://github.com/LightningNetwork/lnd), es un excelente ejemplo de multiplexor de contrato inteligente, neutral, sin custodia, sin permiso, similar a nuestro DEX propuesto. Mientras que el DEX operará dentro de la cadena y entre las criptomonedas, lnd opera fuera de la cadena y para una determinada criptomoneda.

Los proyectos DEX existentes tienen el objetivo de hacer que dicho protocolo sea "inflado", utilizándolo como un medio para monetizar sus servicios, mientras que lo que estamos proponiendo es hacer que el protocolo sea "ágil" intencionalmente. Al simplificar el protocolo y eliminar los incentivos para centralizar, nuestro objetivo es proporcionar valor a largo plazo aportando mayor estabilidad a los mercados de intercambio de criptomonedas. Poner todas las criptomonedas en una situación más equitativa en el contexto del intercambio facilitará el proceso de adquisición de criptomonedas, lo que beneficiará al ecosistema en general.

## Conclusión

Tener la capacidad de intercambiar criptomonedas con mínima fricción, riesgo y centralización es importante para el proceso de la adopción de las criptomonedas por parte del público en general. Implementar el DEX como se describe arriba:

- hacer que el proceso de intercambio no tenga permisos tanto para proyectos como para usuarios
- eliminar intermediarios, barreras de entrada y la mayoría de las tarifas
- ser más resistente a la censura que una cadena o token
- reducir la influencia de los HFT en el proceso de descubrimiento de precios
- proteger la liquidez del ecosistema cripto de la depredación
- prevenir o filtrar la manipulación de comercio, creando un volumen verificable
- facilitar la regulación descentralizada del comercio

La propuesta ya fue aceptada en [Politeia](https://proposals.decred.org/proposals/417607aaedff2942ff3701cdb4eff76637eca4ed7f7ba816e5c0bd2e971602e1) y se espera que entre el mes de [Mayo](https://twitter.com/chappjc/status/1245075450625511425) esté disponible. Puedes leer las especificaciones [aquí](https://github.com/decred/dcrdex/blob/master/spec/README.mediawiki).

Muy pronto conocerás el comercio descentralizado, seguro y sin permiso de verdad.

![dex](https://github.com/francov99/dcrspanish/blob/master/assets/first-look.jpg)

`¡El DEX!`
