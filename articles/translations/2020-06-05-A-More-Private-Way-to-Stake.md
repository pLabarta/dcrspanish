# Una forma más privada de hacer Staking en DCR

Este artículo fué publicado en inglés por JAMIE HOLDSTOCK en el Blog de Decred y esta es su traducción al español.
La comunidad Decred considera que la privacidad es un derecho humano fundamental.

La protección de la privacidad de los usuarios de Decred ha sido un principio básico del proyecto desde su inicio. Para citar la constitución de Decred :
La privacidad y la seguridad son prioridades y se equilibrarán con la complejidad de sus implementaciones. Se implementará tecnología adicional de privacidad y seguridad de forma continua e incremental, tanto de forma proactiva como a pedido en respuesta a los ataques.
A finales de 2019, Jake Yocom-Piatt reveló una solución de privacidad muy esperada para Decred — CoinShuffle ++ (CSPP) . La introducción de CSPP permite a los usuarios anonimizar su Decred al mezclarse con el flujo constante de transacciones de tickets que fluyen a través de la red como parte de su mecanismo de consenso híbrido PoW y PoS .
La respuesta de la comunidad ha sido enfática. Más del 22% del suministro circulante de Decred actualmente reside en UTXO mixtos.
El anuncio de CSPP destacó que se requeriría trabajo adicional para que los usuarios de VSP participen en la mezcla. Dado que hasta el 50% de las entradas en cualquier momento son propiedad de VSP, es razonable aproximar que permitir que las entradas de VSP se mezclen podría duplicar la participación de CSPP.
Después de trabajar en el problema con David Hill durante algunas semanas, me complace anunciar que el trabajo en una nueva implementación de VSP está a punto de finalizar.
Proveedores de servicios de votación

Participar en el sistema de prueba de participación de Decred requiere el uso de una cartera siempre en línea. Aquellos que no pueden o no quieren mantener su propia computadora conectada a Internet las 24 horas del día, los 7 días de la semana, pueden optar por usar un proveedor de servicios de votación (VSP) . Un VSP mantiene un grupo de carteras de votación en línea y permitirá a los usuarios agregar sus tickets a las carteras a cambio de una pequeña tarifa. Los VSP no tienen custodia por completo (nunca tienen acceso a ninguno de los fondos de sus usuarios), el usuario solo le otorga al VSP los derechos para votar sus tickets.
Hasta ahora, los operadores de VSP han ofrecido este servicio a los usuarios ejecutando dcrstakepool . Este software funciona bien y tiene un historial probado de casi cuatro años. Actualmente, los VSP que ejecutan dcrstakepool tienen los derechos de voto de más de 14,500 tickets con un valor aproximado de 2 millones de DCR.
Cuando un usuario desea usar un VSP que ejecuta dcrstakepool, debe comprar su tiket de una manera particular, de modo que una parte de la recompensa del voto del ticket se pague automáticamente al VSP. Como resultado, los tickets VSP tienen una huella en la cadena diferente a los tickets comprados por los votantes en solitario, lo que los hace identificables por un observador externo que realiza análisis en la cadena. Esta huella es problemática porque prohíbe a los usuarios de VSP aprovechar al máximo la privacidad ofrecida por CSPP: los tickets de VSP no se pueden mezclar en el mismo conjunto de anonimato que los tickets individuales.

## Introduciendo vspd

vspd es una implementación desde cero de un proveedor de servicios de votación (VSP) para la red Decred. Se ha creado con la privacidad del usuario como un objetivo de diseño fundamental.
vspd no requiere que se compren ticket con condiciones especiales, como el pago de la tarifa incorporada requerido por dcrstakepool. Los pagos de tarifas ocurren como una transacción en cadena completamente independiente. Esto permite que tanto la compra del ticket como el pago de la tarifa se mezclen en el mismo conjunto de anonimato que los tickets individuales.
Los titulares de ticket no están obligados a registrar una cuenta en vspd. Esto significa que no es necesario entregar una dirección de correo electrónico, no hay contraseña para recordar y no es necesario resolver CAPTCHA .
vspd también ofrece las siguientes mejoras a los usuarios:
No secanjean los scripts para realizar copias de seguridad.
Una sola cartera podría usar un VSP diferente para cada una de sus entradas.
Las preferencias de votación se pueden establecer por ticket.
No se reutiliza la dirección .
La posibilidad de utilizar múltiples VSP para un solo ticket.
Los operadores de VSP también se han considerado en el diseño de vspd. dcrstakepool tenía la desafortunada propiedad de recaudar pagos de tarifas con una sola dirección reutilizada por usuario. vspd requiere que los clientes soliciten una nueva dirección de tarifa y un monto de tarifa para cada ticket, lo que no solo elimina la reutilización de la dirección, sino que también permite a los administradores actualizar fácilmente el monto de su tarifa como lo deseen.
El trabajo del administrador del sistema y los gastos generales también se han reducido con los siguientes cambios:
Una instancia de etcd-io / bbolt en el servidor front-end se usa como la única fuente de confianza:
bbolt no tiene la sobrecarga de sysadmin asociada con el mantenimiento de un servidor de base de datos completo como MySQL. vspd crea y mantiene automáticamente su propia base de datos.
Solo se accede a la base de datos mediante el proceso vspd. No es necesario exponer el sistema a riesgos adicionales abriendo puertos para que otros procesos accedan a la base de datos.
Los servidores de carteras de votación solo requieren que se ejecuten dcrwallet y dcrd. Ya no hay un proceso VSP adicional, es decir, un grupo de interés, que se ejecuta en servidores de votación.
Vspd no posee direcciones de correo electrónico ni información personal; no es necesario preocuparse por GDPR u otras regulaciones de protección de datos.
Los servidores de votación ya no necesitan que dcrd se ejecute con un índice de transacción.
No es necesario usar la misma semilla de carteras en cada votación.
Próximos pasos
Como pieza central de la infraestructura Decred, fundamental para el buen funcionamiento de la red, vspd es de suma importancia y su desarrollo no puede ser apresurado.
Hasta ahora, vspd solo se ha probado con una herramienta cliente personalizada, no apta para el uso diario. No recomendaremos ejecutar vspd en mainnet hasta que el nuevo software del servidor haya sido probado exhaustivamente y el código del cliente esté correctamente integrado en dcrwallet . Una vez que los tickets vspd se pueden comprar con dcrwallet, la integración en el decrediton de la cartera GUI debe seguir poco después.
Se espera un período de transición de alrededor de 12 meses, donde los VSP pueden ejecutar vspd, dcrstakepool o ambos. Esto debería evitar interrumpir la experiencia del usuario demasiado bruscamente y proteger el flujo constante de compra de tickets y votación que es esencial para la red Decred. Los operadores de VSP existentes pueden optar por implementar vspd junto con su implementación de dcrstakepool existente. Los operadores más nuevos pueden optar por ejecutar solo vspd. Se prevé que, con el tiempo, la mayoría de las entradas de VSP se comprarán en vspd, las implementaciones de dcrstakepool se pueden retirar y el soporte para dcrstakepool se puede eliminar de las carteras.
En conclusión
El lanzamiento de vspd representa otro paso progresivo en la misión del proyecto de Decred para producir tecnología de código abierto para beneficio público. Permitir que se compren tickets VSP con fondos combinados a través de CSPP aumenta el tamaño del conjunto de anonimato general, lo que a su vez proporciona un mayor nivel de privacidad para todos los usuarios Decred.
Para obtener más información sobre cómo se construye vspd y cómo funciona, el código fuente y alguna documentación están disponibles en GitHub .
El proyecto Decred siempre busca desarrolladores talentosos, por lo que, si vspd o cualquier otro aspecto del desarrollo Decred le interesa, ingrese a nuestro chat y preséntese.
