# Informe: Escenario Acumulación y Despacho

Sep 30, 2026 · @German

Propuesta para ARSIM Studio: unificar los escenarios de cinta "Tope y Singulado", "Lote de 3" y "Buffer de 3 Zonas" en un solo escenario de 5 niveles, "Acumulación y Despacho", que elimina los niveles redundantes y suma dos niveles nuevos: uno de cambio de velocidad y uno integrador.

## Situación actual

Hoy la categoría Básico tiene 8 escenas. Tres de ellas son escenarios de cinta con 2 niveles cada uno, y en los tres el nivel 2 repite el nivel 1: solo cambia la cantidad de cajas y se agrega una parada al final.

| Escenario | Tags | Nivel 1 (se conserva) | Nivel 2 (redundante) |
| --- | --- | --- | --- |
| Tope y Singulado | 9: Sensor\_1, Tope\_1, Marcha\_1, Parada\_1, Emergencia\_1, Cinta\_1\_Velocidad, Cinta\_1\_Motor\_FWD/REV, Baliza\_1 | "Una caja por Marcha": cada Marcha con caja en Sensor\_1 baja el tope 0,6 s (TON + SET/RESET). 5 cajas liberadas de a una. | Más cajas + parada final |
| Lote de 3 | 10: agrega Sensor\_2 | "Lotes de tres": CTU cuenta en Sensor\_1; con 3 cajas y la primera en Sensor\_2, el tope baja 2,4 s. 3 lotes; un lote de 2 o de 4 es falla. | Más cajas + parada final |
| Buffer de 3 Zonas | 13: Sensor\_1..3, Tope\_1..3 | "Llenar el buffer": cada zona retiene su caja mientras la siguiente esté ocupada; Tope\_3 retiene siempre y Marcha lo baja 0,6 s. Buffer lleno 2 veces, 5 cajas despachadas. | Más cajas + parada final |

## Propuesta: "Acumulación y Despacho"

Los tres escenarios pasan a ser uno solo con 5 niveles. Los niveles 1 a 3 son los niveles 1 actuales con un ítem final de Parada. Los niveles 4 y 5 son nuevos. La categoría Básico pasa de 8 a 6 escenas (a 3 con las tres mejoras de este informe).

Descripción de la card: *"Una cinta, cinco desafíos: singulá cajas con un tope, contá lotes con un CTU, armá un buffer por zonas, aprendé a cambiar la velocidad de la cinta y combiná todo en una línea de despacho por lotes."*

| # | Nivel | Origen | Qué enseña | Ítems de la misión |
| --- | --- | --- | --- | --- |
| 1 | Una caja por Marcha | N1 de Tope y Singulado | TON, SET/RESET, pulso de tope | Marcha → 5 cajas de a una → Parada |
| 2 | Lotes de tres | N1 de Lote de 3 | CTU, comparación, reset | Marcha → 3 lotes de 3 → Parada |
| 3 | Llenar el buffer | N1 de Buffer de 3 Zonas | Lógica de zonas (enclavamiento entre topes) | Marcha → buffer lleno ×2 → 5 despachadas → Parada |
| 4 | Buffer a doble velocidad (nuevo) | Variante del N3 | Escribir la velocidad de la cinta desde el programa | Marcha → velocidad 100 → buffer lleno ×2 → 5 despachadas con pulso de 0,3 s → Parada |
| 5 | Despacho por lotes (nuevo) | Integra N1 a N4 | Tope + lote + buffer + velocidad | Marcha → buffer lleno → 3 lotes de 3 → Parada → Emergencia |

Nombres alternativos: escenario "Control de Flujo en Cinta"; nivel 4 "Pulso corto, cinta rápida"; nivel 5 "Línea completa".

## Niveles nuevos

### Nivel 4: Buffer a doble velocidad

Es el nivel 3 con el pulso de despacho de Tope\_3 reducido a 0,3 s. A velocidad 50, 0,3 s no alcanza para que salga una caja, así que el alumno tiene que escribir `Cinta_1_Velocidad` = 100 desde el programa para que salga exactamente una. El objetivo pedagógico es aprender a modificar la velocidad de la cinta.

### Nivel 5: Despacho por lotes

Integra todo: el buffer de 3 zonas acumula 3 cajas y se despachan siempre en lotes de exactamente 3 por Tope\_3.

1. La lógica de zonas del nivel 3 retiene las cajas.
2. Con las 3 zonas ocupadas (buffer lleno), Marcha\_1 dispara el despacho. Marcha sin buffer lleno no hace nada.
3. Tope\_3 baja, un CTU cuenta las cajas que salen y al llegar a 3 el tope sube y el contador se resetea.
4. El alumno tiene que ajustar la velocidad de la cinta para que se acumulen 3 cajas y salgan juntas. La regla exacta la define el programador (ver Decisiones).
5. Al final se detiene la línea con Parada y se prueba la Emergencia: con la cinta en marcha, Emergencia\_1 apaga motores y deja los topes reteniendo, y al soltarla la línea no rearranca sin Marcha.

Fallas: lote de 2 o de 4, caja que pasa un tope de zona, despacho sin buffer lleno.

## Decisiones

Dos puntos están definidos; la regla de velocidad del nivel 5 la define el programador después de probarla en el simulador.

| Tema | Decisión | Estado |
| --- | --- | --- |
| Disparo del despacho en el nivel 5 | Con Marcha\_1, solo con el buffer lleno | Definido |
| Progreso de alumnos que ya jugaron | Trasladar: el récord del N1 de cada escenario viejo pasa al nivel 1, 2 o 3 nuevo; los niveles 2 viejos se descartan; los niveles 4 y 5 arrancan en cero | Definido (alternativa: reiniciar todo si solo hubo pruebas internas) |
| Regla de velocidad del nivel 5 | Sugerencia: acumular a velocidad baja (a alta los topes de zona no retienen) y despachar a velocidad alta dentro de una ventana máxima de Tope\_3 (1,2 s) | Lo define el programador tras probar en el simulador |

## Impactos técnicos

- **Escena y tags por nivel.** El N1 usa 1 sensor y 1 tope, el N2 usa 2 sensores y 1 tope, y los N3 a N5 usan 3 y 3. Hoy cada escenario tiene una escena fija, así que el layout pasa a definirse por nivel.
- **Velocidad escribible.** `Cinta_1_Velocidad` hoy es una entrada forzable (valor 50). En los niveles 4 y 5 el programa tiene que poder escribirla y la escena 3D tiene que aplicarla en tiempo real.
- **Instrucción MOVE.** El simulador Ladder necesita una instrucción para escribir un entero. Si no existe, hay que agregarla a la librería.
- **Ítem de Parada reutilizable.** Los 5 niveles lo usan, así que conviene implementarlo una sola vez.
- **Card y conteos.** El escenario nuevo tiene 5 niveles, 13 tags y 15 estrellas; la categoría Básico pasa de 8 a 6 escenas (a 3 con las tres mejoras de este informe).
- **Migración de récords.** Tiene que correr una sola vez y sin duplicar datos.

## Prompts de cambio

Un prompt por cambio, para pasarle a la IA del programador en este orden. Pegar el contexto común antes de cada prompt.

### Contexto común

```
Trabajás sobre ARSIM Studio, un simulador de PLC con escena 3D de planta, editor Ladder, panel de tags (entradas/salidas/variables) y misiones por nivel con ítems de progreso y puntaje. En la categoría "Básico" existen tres escenarios de cinta: "Tope y Singulado", "Lote de 3" y "Buffer de 3 Zonas", cada uno con 2 niveles. Primero localizá en el código dónde se definen escenarios, niveles, ítems de misión, validaciones, tags y layout de escena (buscá esos nombres y los títulos de nivel "Una caja por Marcha", "Lotes de tres", "Llenar el buffer"). Respetá las convenciones existentes y no rompas los demás escenarios. Antes de cambiar código, resumime qué archivos vas a tocar.
```

### Prompt 1: escenario unificado y retiro de los tres viejos

```
Creá un escenario nuevo en la categoría Básico llamado "Acumulación y Despacho", con 5 niveles, que reemplace a "Tope y Singulado", "Lote de 3" y "Buffer de 3 Zonas". Retirá esos tres escenarios de la categoría (Básico pasa de 8 a 6 escenas, o a 3 si también se unifican los escenarios básicos y los de clasificación) y eliminá sus niveles 2, que eran redundantes. Descripción de la card: "Una cinta, cinco desafíos: singulá cajas con un tope, contá lotes con un CTU, armá un buffer por zonas, aprendé a cambiar la velocidad de la cinta y combiná todo en una línea de despacho por lotes." Dificultad INICIAL.

El escenario tiene que admitir layout de escena y set de tags por nivel: N1 usa 1 sensor y 1 tope, N2 usa 2 sensores y 1 tope, N3 a N5 usan 3 sensores y 3 topes. Si hoy el layout es por escenario, llevalo a nivel.

Migración de progreso: para los usuarios que ya jugaron los escenarios viejos, trasladá el récord (puntaje y estrellas) así: nivel 1 de "Tope y Singulado" -> nivel 1 nuevo; nivel 1 de "Lote de 3" -> nivel 2 nuevo; nivel 1 de "Buffer de 3 Zonas" -> nivel 3 nuevo. Los récords de los niveles 2 viejos se descartan. Los niveles 4 y 5 arrancan sin progreso. La migración tiene que poder correr una sola vez sin duplicar datos. Si no hay forma de migrar, avisame antes de borrar nada.

Actualizá la card del escenario: 5 niveles, 13 tags, 15 estrellas en total (3 por nivel).
```

### Prompt 2: Nivel 1 "Una caja por Marcha"

```
En "Acumulación y Despacho", el nivel 1 es una copia del nivel 1 de "Tope y Singulado" (mismo enunciado, escena con Sensor_1 y Tope_1, 9 tags, pulso de tope de 0,6 s, 5 cajas liberadas de a una, falla si pasan dos o si el tope baja sin caja). Agregá un ítem final nuevo: "Detené la línea con Parada". Se cumple cuando, completados los ítems anteriores, se presiona Parada_1 y Cinta_1_Motor_FWD queda en FALSE. El nivel termina recién al cumplir ese ítem. Implementá este ítem como reutilizable, porque lo usan los 5 niveles. Ajustá el enunciado para mencionar la parada final.
```

### Prompt 3: Nivel 2 "Lotes de tres"

```
El nivel 2 de "Acumulación y Despacho" es una copia del nivel 1 de "Lote de 3" (Sensor_1 cuenta con CTU, Sensor_2 detecta la primera caja del lote, el tope baja 2,4 s, 3 lotes, falla si sale un lote de 2 o de 4). Escena con Sensor_1, Sensor_2 y Tope_1 (10 tags). Agregá al final el ítem reutilizable "Detené la línea con Parada" y ajustá el enunciado.
```

### Prompt 4: Nivel 3 "Llenar el buffer"

```
El nivel 3 de "Acumulación y Despacho" es una copia del nivel 1 de "Buffer de 3 Zonas" (3 zonas con Sensor_1..3 y Tope_1..3; Tope_1 retiene si Sensor_2 ve caja; Tope_2 retiene si Sensor_3 ve caja; Tope_3 retiene siempre y Marcha lo baja 0,6 s; ítems "Buffer lleno" 0/2 y "Cajas despachadas" 0/5). 13 tags. Agregá al final el ítem reutilizable "Detené la línea con Parada" y ajustá el enunciado.
```

### Prompt 5: velocidad de cinta escribible (base para los niveles 4 y 5)

```
Hoy Cinta_1_Velocidad aparece como ENTRADA forzable (valor 50). El escenario "Cinta Transportadora con Clasificador" ya escribe Cinta_1_Velocidad desde el programa (a partir de Cinta_1_Velocidad_SP): revisá cómo lo hace y reutilizá ese mecanismo. Hacé que, en los niveles que lo habiliten (flag por nivel), sea un tag escribible por el programa Ladder y que la escena 3D aplique ese valor en tiempo real a la velocidad de la cinta. La velocidad de las cajas tiene que ser proporcional: a 100 una caja recorre la misma distancia en la mitad de tiempo que a 50.

Verificá que el simulador Ladder tenga una instrucción para escribir un entero (MOVE o asignación). Si no existe, agregala a la librería de instrucciones con su símbolo, su ayuda y su validación de tipo (INT).

En los niveles sin el flag, el comportamiento actual no cambia.
```

### Prompt 6: Nivel 4 "Buffer a doble velocidad" (nuevo)

```
Creá el nivel 4 de "Acumulación y Despacho", titulado "Buffer a doble velocidad". Escena, tags y lógica de zonas iguales al nivel 3, con el flag de velocidad escribible activado.

Diferencia: el pulso de despacho de Tope_3 es de 0,3 s. A la velocidad inicial (50), 0,3 s no alcanza para que salga una caja. El alumno tiene que escribir Cinta_1_Velocidad = 100 (o más) desde el programa para que con 0,3 s salga exactamente una caja.

Ítems:
1) Poné en marcha la cinta.
2) Subí la velocidad de la cinta a 100 desde el programa (se cumple solo si el valor lo escribe la lógica, no si se fuerza a mano).
3) Buffer lleno 0/2.
4) Cajas despachadas de a una 0/5 (falla si salen dos en un pulso).
5) Detené la línea con Parada.

Enunciado sugerido: "La planta necesita más ritmo: subí la velocidad de la cinta desde el programa y ajustá el pulso del tope final a 0,3 s. Llená el buffer dos veces y despachá cinco cajas de a una."

Agregá a la ficha del nivel una explicación de cómo se escribe la velocidad (MOVE -> Cinta_1_Velocidad).
```

### Prompt 7: Nivel 5 "Despacho por lotes" (nuevo, integrador)

```
Creá el nivel 5 de "Acumulación y Despacho", titulado "Despacho por lotes". Escena de 3 zonas (Sensor_1..3, Tope_1..3), velocidad escribible activada. Objetivo: acumular 3 cajas en el buffer y despacharlas siempre en lotes de exactamente 3 por Tope_3.

Reglas: la lógica de zonas del nivel 3 retiene las cajas. Cuando las 3 zonas están ocupadas (buffer lleno), el despacho se habilita y se dispara con Marcha_1. Si se presiona Marcha sin el buffer lleno, no pasa nada. Durante el despacho Tope_3 baja, un CTU cuenta las cajas que salen y, al llegar a 3, el tope sube y el contador se resetea.

Velocidad (la definís vos después de probar): la idea pedagógica es que el alumno tenga que cambiar Cinta_1_Velocidad desde el programa para que el lote funcione. Primero probá en el simulador:
(a) a velocidad alta (100 o más), ¿los topes de zona llegan a retener las cajas o se pasan?
(b) a velocidad 50, ¿cuánto tardan 3 cajas en salir por Tope_3?
Con esos datos, definí vos la regla (por ejemplo: acumular a velocidad baja y despachar a velocidad alta dentro de una ventana máxima con Tope_3 abajo, o una sola velocidad obligatoria distinta de 50). Implementala y explicala en la ficha del nivel.

Fallas: lote de 2 o de 4, caja que pasa un tope de zona, despacho sin buffer lleno.

Ítems:
1) Poné en marcha la cinta.
2) Buffer lleno (3 zonas).
3) Lotes de 3 despachados 0/3.
4) Detené la línea con Parada.
5) Probá la parada de emergencia: volver a arrancar la línea y presionar Emergencia_1 con la cinta en marcha. Se cumple si en ese mismo scan quedan Cinta_1_Motor_FWD y Cinta_1_Motor_REV en FALSE y todos los topes en posición de retener, y si al soltar la emergencia la línea NO rearranca sola (requiere Marcha de nuevo).

Enunciado sugerido: "Integrá todo lo aprendido: acumulá tres cajas en el buffer, ajustá la velocidad de la cinta y despachá con Marcha lotes de exactamente tres. Terminá deteniendo la línea y probando la parada de emergencia."
```

## Mejora: escenarios básicos de cinta

Los 3 primeros escenarios de Básico ("Arranque con retención y parada", "Parada por fotocélula" y "Arranque con aviso") pasan a ser un solo escenario de 4 niveles: **"Funcionamientos Básicos de Cinta Transportadora"**. Cada escenario actual queda como un nivel con su nombre, se le suma la Parada que hoy está en su nivel 2 y un objetivo de 5 cajas transportadas. El nivel 4 es nuevo e integra los tres.

| # | Nivel | Origen | Escena | Ítems de la misión |
| --- | --- | --- | --- | --- |
| 1 | Arranque con retención y parada | N1 de "Arranque con retención y parada" | Solo la cinta | Consignas actuales → 5 cajas → Parada |
| 2 | Parada por Sensor Fotocélula | N1 de "Parada por fotocélula" | Cinta + sensor | Consignas actuales → 5 cajas → Parada |
| 3 | Arranque con Aviso | N1 de "Arranque con aviso" | Cinta + baliza (sin sensor) | Consignas actuales → 5 cajas → Parada |
| 4 | Ciclo Completo (nuevo) | Combina N1 a N3 | Cinta + sensor + baliza | Marcha con aviso → parada por fotocélula → Emergencia a mitad del nivel → 5 cajas → Parada |

### Nivel 4: Ciclo Completo

El alumno arma un solo programa que combina los tres niveles, como funciona una cinta real. Ciclo sugerido (el programador puede ajustarlo):

1. Marcha\_1 enciende la baliza como aviso y, pasado el tiempo de aviso, la cinta arranca con retención.
2. Cuando la caja llega a la fotocélula, la cinta se detiene.
3. Una nueva Marcha repite el ciclo con la siguiente caja.
4. A mitad del nivel (por ejemplo, después de la caja 3) aparece la consigna "Detené con el botón de emergencia": con la cinta en marcha, Emergencia\_1 detiene el motor y al soltarla la cinta no rearranca sola.
5. Se completan las 5 cajas y se detiene la línea con Parada.

### Decisiones

El nombre del escenario y de los niveles quedan como se definieron. El resto de los puntos los define el programador al implementar, y los prompts ya se lo piden.

| Punto | Estado | Sugerencia de este informe |
| --- | --- | --- |
| Nombre del escenario y de los niveles | Definido | — |
| Qué libera la caja detenida por la fotocélula (niveles 2 y 4) | Lo define el programador | Una nueva Marcha |
| Momento de la emergencia en el nivel 4 | Lo define el programador | Después de la caja 3 |
| Conteo de cajas en los niveles 1 y 3 (sin sensor) | Lo define el programador | Que las cuente el simulador al llegar al final de la cinta |

### Impacto sumado con la otra mejora

Con las tres mejoras de este informe, Básico pasa de 8 a 3 escenas: "Funcionamientos Básicos de Cinta Transportadora" (primera), "Acumulación y Despacho" y "Clasificación por Empujadores". Las tres mejoras necesitan escena y tags por nivel: la que se implemente primero construye esa base y las demás la reutilizan. El ítem de Parada también es el mismo en ambas. Los récords se migran con el mismo criterio: el N1 de cada escenario viejo pasa a su nivel nuevo y los niveles 2 viejos se descartan.

## Prompts de cambio: escenarios básicos

Un prompt por cambio, para pasar en este orden. Pegar el contexto antes de cada prompt. Se numeran B-1 a B-3 para no confundirlos con los prompts de Acumulación y Despacho.

### Contexto (escenarios básicos)

```
Trabajás sobre ARSIM Studio, un simulador de PLC con escena 3D de planta, editor Ladder, panel de tags (entradas/salidas/variables) y misiones por nivel con ítems de progreso y puntaje. En la categoría "Básico", los tres primeros escenarios son "Arranque con retención y parada", "Parada por fotocélula" y "Arranque con aviso", cada uno con 2 niveles: el nivel 2 repite el nivel 1 y agrega el botón de Parada. Primero localizá en el código dónde se definen escenarios, niveles, ítems de misión, validaciones, tags y layout de escena (buscá esos nombres). Respetá las convenciones existentes y no rompas los demás escenarios. Si ya existe soporte de escena y tags por nivel, o un ítem reutilizable "Detené la línea con Parada" (de la unificación de "Acumulación y Despacho"), reutilizalos. Antes de cambiar código, resumime qué archivos vas a tocar.
```

### Prompt B-1: escenario unificado y retiro de los tres viejos

```
Creá un escenario nuevo en la categoría Básico llamado "Funcionamientos Básicos de Cinta Transportadora", con 4 niveles, que reemplace a "Arranque con retención y parada", "Parada por fotocélula" y "Arranque con aviso". Va primero en la categoría. Retirá esos tres escenarios y sus niveles 2.

Niveles: 1) Arranque con retención y parada; 2) Parada por Sensor Fotocélula; 3) Arranque con Aviso; 4) Ciclo Completo.

La escena y el set de tags cambian por nivel:
- Nivel 1: solo la cinta (sin sensor ni baliza).
- Nivel 2: cinta + sensor fotocélula.
- Nivel 3: cinta + baliza (sin sensor).
- Nivel 4: cinta + sensor + baliza.
Si hoy el layout es por escenario, llevalo a nivel. En cada nivel el panel de tags muestra solo los tags de los elementos presentes.

Migración de progreso: para quienes ya jugaron, trasladá el récord (puntaje y estrellas) del nivel 1 de cada escenario viejo a su nivel nuevo (1, 2 y 3). Los récords de los niveles 2 viejos se descartan. El nivel 4 arranca sin progreso. La migración corre una sola vez sin duplicar datos. Si no hay forma de migrar, avisame antes de borrar nada.

Actualizá la card: 4 niveles, 12 estrellas (3 por nivel), cantidad de tags del escenario, y una descripción como: "Lo esencial de una cinta: arranque con retención, parada por fotocélula, arranque con aviso y, al final, un ciclo completo como en planta."
```

### Prompt B-2: niveles 1 a 3 (los niveles actuales + Parada + 5 cajas)

```
En "Funcionamientos Básicos de Cinta Transportadora", los niveles 1, 2 y 3 son copias del nivel 1 de "Arranque con retención y parada", "Parada por fotocélula" y "Arranque con aviso" respectivamente: mismo enunciado, lógica, tags y validaciones. A cada uno agregale, después de sus consignas actuales:

1) Ítem "Transportá 5 cajas" (0/5). En los niveles 1 y 3 no hay sensor: el conteo lo hace el simulador cuando cada caja llega al final de la cinta, no un tag del PLC. En el nivel 2 respetá la mecánica actual de la fotocélula; si hoy no está definido qué libera la caja después de que la fotocélula detiene la cinta, definilo vos (sugerencia: una nueva Marcha) y explicalo en el enunciado.
2) Ítem final "Detené la línea con Parada" (el mismo que hoy está en el nivel 2 de cada escenario): se cumple cuando, completadas las 5 cajas, se presiona Parada_1 y el motor de la cinta queda en FALSE. El nivel termina recién al cumplirlo. Si ya existe un ítem reutilizable de Parada, usalo.

Ajustá cada enunciado para mencionar las 5 cajas y la parada final.
```

### Prompt B-3: Nivel 4 "Ciclo Completo" (nuevo)

```
Creá el nivel 4 de "Funcionamientos Básicos de Cinta Transportadora", titulado "Ciclo Completo". Escena: cinta + sensor fotocélula + baliza, con todos sus tags. El alumno arma un solo programa que combina los niveles 1 a 3, como funciona una cinta real.

Ciclo esperado:
1) Marcha_1 enciende la baliza como aviso y, pasado el tiempo de aviso (el mismo del nivel 3), la cinta arranca con retención.
2) Cuando la caja llega a la fotocélula, la cinta se detiene (como en el nivel 2).
3) Una nueva Marcha repite el ciclo con la siguiente caja.

Ítems:
1) Arrancá la cinta con aviso (la cinta solo cuenta como arrancada si la baliza avisó antes el tiempo requerido).
2) Cajas transportadas 0/5, con parada por fotocélula en cada una.
3) A mitad del nivel, después de la caja 3, aparece la consigna "Detené con el botón de emergencia": con la cinta en marcha se presiona Emergencia_1. Se cumple si el motor de la cinta queda en FALSE en ese mismo scan y si al soltar la emergencia la cinta NO rearranca sola: tiene que volver a arrancarse con Marcha y aviso. Sirve para verificar que la emergencia está bien programada.
4) Después de la caja 5: "Detené la línea con Parada" (ítem reutilizable).

Fallas: arrancar sin aviso, que la cinta no se detenga en la fotocélula, que rearranque sola después de la emergencia.

Enunciado sugerido: "Ahora como en planta: arrancá con aviso, detené cada caja en la fotocélula y verificá que la parada de emergencia funciona. Después de cinco cajas, detené la línea con Parada."

El ciclo depende de cómo se libera la caja detenida por la fotocélula: usá el mismo criterio que definas para el nivel 2. El momento de la emergencia (sugerencia: después de la caja 3) también lo definís vos.
```

## Mejora: escenarios de clasificación

"Desvío por Empujador" (INICIAL, 12 tags) y "Cinta Transportadora con Clasificador" (INTERMEDIO, 15 tags) pasan a ser un solo escenario, **"Clasificación por Empujadores"**, que tiene 5 niveles. La dificultad sube de a una cosa por nivel: un empujador, dos rampas con conteo, velocidad variable, clasificación por color y, al final, todo junto con cálculo de tiempos.

Cómo funcionan hoy los niveles 1 que se reutilizan:

- **Desvío por Empujador, "Desviar cada caja".** La cinta arranca en sentido REV (Cinta\_1\_Motor\_REV). Sensor\_1 está medio metro antes del empujador 1: cuando ve una caja, un TON de 0,3 s espera a que llegue frente al vástago y Piston\_1 da un pulso (SET, y RESET con otro TON de 1 s). La caja cae por la rampa 1. Son 6 cajas, y una caja que llega al final sin desviarse es falla.
- **Cinta Transportadora con Clasificador, "Clasificá 5 cajas".** El operador carga la velocidad con −/+ en el display del puesto. El programa la lee en Cinta\_1\_Velocidad\_SP y la escribe en Cinta\_1\_Velocidad, con un tope, porque a cualquier velocidad los empujadores no llegan. Son 5 cajas descartadas en los contenedores.

| # | Nivel | Origen | Qué enseña | Ítems de la misión |
| --- | --- | --- | --- | --- |
| 1 | Desviar cada caja | N1 de Desvío por Empujador, sin cambios | Sensor + TON de espera + pulso de pistón | Marcha en REV → 6 cajas a la rampa 1 |
| 2 | Reparto cinco y cinco (nuevo) | Variante del N1 con las dos rampas | Contador (CTU) que cambia el destino | Marcha → 5 cajas a la rampa 1 → 5 cajas a la rampa 2 |
| 3 | Clasificación a velocidad variable | N1 del Clasificador + reparto 5 y 5 | Leer el SP del operador, limitarlo y escribir la velocidad | Velocidad desde el SP con tope → 5 en el contenedor 1 → 5 en el contenedor 2 |
| 4 | Clasificar por color (nuevo) | Variante del N2 con sensores de color | Comparar el color leído y decidir qué empujador actúa | Cajas de colores al azar → 5 rojas a la rampa 1 → 5 azules a la rampa 2 → el resto pasa de largo |
| 5 | Clasificación completa (nuevo) | Integra N1 a N4 | Calcular el tiempo de viaje según distancia y velocidad | Velocidad desde el SP → sensores en otra posición → 5 rojas y 5 azules → el resto pasa de largo |

Nombres alternativos: escenario "Desvío y Clasificación en Cinta"; nivel 2 "Llenar dos rampas" o "Desvío por conteo".

### Nivel 2: Reparto cinco y cinco

Es el nivel 1 con las dos rampas en juego. Las primeras 5 cajas van a la rampa 1 con Piston\_1. Al llegar a 5, un CTU cambia el destino y las 5 siguientes pasan de largo por el empujador 1 y se desvían a la rampa 2 con Sensor\_2 + TON + Piston\_2. Fallas: una sexta caja en la rampa 1, una caja en la rampa 2 antes de completar la rampa 1, o una caja que llega al final de la cinta.

### Nivel 3: Clasificación a velocidad variable

Es el reparto 5 y 5 del nivel 2 con la mecánica de velocidad del Clasificador. El programa lee Cinta\_1\_Velocidad\_SP, lo limita al máximo al que los empujadores llegan y lo escribe en Cinta\_1\_Velocidad. Durante la misión se sube la velocidad, así que el alumno también tiene que ajustar la espera antes del pulso, que ya no puede ser un 0,3 s fijo.

### Nivel 4: Clasificar por color

Los sensores fotoeléctricos se reemplazan por sensores de color, uno antes de cada empujador y en la misma posición que hoy. Entran cajas de distintos colores en orden aleatorio: las rojas van a la rampa 1, las azules a la rampa 2 y cualquier otro color pasa de largo hasta el final de la cinta. El nivel termina con 5 rojas y 5 azules.

La velocidad de la cinta y el temporizado quedan fijos en el estándar del nivel 1 (velocidad 50, espera de 0,3 s y pulso de 1 s). El escenario ubica los sensores de color sincronizados para que, con ese pulso estándar de empujar y volver, la caja caiga en la rampa. Así el alumno se concentra solo en la lógica de color. Fallas: una caja en la rampa equivocada, una caja de otro color desviada, o una roja o azul que llega al final.

### Nivel 5: Clasificación completa

Combina todo lo anterior: velocidad de la cinta desde el SP del operador con tope, reparto de 5 cajas por rampa y clasificación por color. La diferencia es que los sensores de color están en otra posición, más lejos del empujador, así que la espera de 0,3 s ya no sirve. El alumno tiene que calcular cuánto tarda la caja en llegar del sensor al empujador (tiempo = distancia / velocidad) y, como la velocidad puede cambiar, lo ideal es que el programa lo calcule a partir de la velocidad actual.

### Decisiones

Definido: la card del escenario pasa a **INTERMEDIO**. El resto de los puntos los define el programador al implementar, y los prompts ya se lo piden.

| Punto | Estado | Sugerencia de este informe |
| --- | --- | --- |
| Dificultad de la card | Definido: INTERMEDIO | — |
| Quién sube la velocidad en los niveles 3 y 5 | Lo define el programador | El nivel la sube solo después de la caja 5, para obligar a que el programa se adapte |
| Escena | Lo define el programador | Una sola línea (la de Desvío) para los 5 niveles, con display del operador en 3 y 5 y sensores de color en 4 y 5 |
| Cómo entrega el color el sensor | Lo define el programador | Un entero por sensor (0 = sin caja, 1 = roja, 2 = azul, 3 = otro color) |
| Distancia del sensor en el nivel 5 | Lo define el programador | Fija y publicada en la ficha del nivel |
| Conversión de velocidad a m/s | Lo define el programador (medida en el simulador) | Publicarla en la ficha, por ejemplo 100 % = 1 m/s |
| Ítem de Parada final | Lo define el programador | Agregarlo en todos los niveles, como en las otras dos mejoras |
| Récords | Definido | Se migra solo el nivel 1 de Desvío |

### Impacto sumado

Con las tres mejoras de este informe, Básico pasa de 8 a 3 escenas: "Funcionamientos Básicos de Cinta Transportadora", "Acumulación y Despacho" y "Clasificación por Empujadores". Además, el Clasificador ya escribe Cinta\_1\_Velocidad desde el programa, así que ese mecanismo se reutiliza en los niveles 4 y 5 de Acumulación y Despacho.

## Prompts de cambio: clasificación

Un prompt por cambio, para pasar en este orden. Pegar el contexto antes de cada prompt. Se numeran C-1 a C-5.

### Contexto (clasificación)

```
Trabajás sobre ARSIM Studio, un simulador de PLC con escena 3D de planta, editor Ladder, panel de tags (entradas/salidas/variables) y misiones por nivel con ítems de progreso y puntaje. En la categoría "Básico" existen los escenarios "Desvío por Empujador" (nivel 1 "Desviar cada caja") y "Cinta Transportadora con Clasificador" (nivel 1 "Clasificá 5 cajas"), cada uno con 2 niveles. Primero localizá en el código dónde se definen escenarios, niveles, ítems de misión, validaciones, tags y layout de escena (buscá esos nombres). Respetá las convenciones existentes y no rompas los demás escenarios. Si ya existe soporte de escena y tags por nivel, reutilizalo. Antes de cambiar código, resumime qué archivos vas a tocar.
```

### Prompt C-1: escenario unificado + nivel 1

```
Creá un escenario nuevo en la categoría Básico llamado "Clasificación por Empujadores", con 5 niveles, que reemplace a "Desvío por Empujador" y "Cinta Transportadora con Clasificador". Retirá esos dos escenarios y sus niveles 2. Dificultad: INTERMEDIO.

Niveles: 1) Desviar cada caja; 2) Reparto cinco y cinco; 3) Clasificación a velocidad variable; 4) Clasificar por color; 5) Clasificación completa.

Escena: la línea de "Desvío por Empujador" (cinta en sentido REV, Sensor_1..3, empujadores Piston_1 y Piston_2 con sus dos rampas) para los 5 niveles. En los niveles 3 y 5 se agrega el display del operador con Cinta_1_Velocidad_SP. En los niveles 4 y 5 los sensores fotoeléctricos se reemplazan por sensores de color. Si hoy el layout es por escenario, llevalo a nivel. Si ves una opción de escena mejor, usala y explicame por qué.

Ítem de Parada: decidí si cada nivel termina con el ítem "Detené la línea con Parada", como en los otros escenarios unificados (sugerencia: sí, en todos). Si ya existe como ítem reutilizable, usalo.

El nivel 1 es una copia exacta del nivel 1 de "Desvío por Empujador" ("Desviar cada caja": Sensor_1 medio metro antes del empujador 1, TON de 0,3 s, pulso de Piston_1 con SET y RESET por TON de 1 s, 6 cajas a la rampa 1, falla si una caja llega al final sin desviarse).

Migración de progreso: trasladá el récord (puntaje y estrellas) del nivel 1 de "Desvío por Empujador" al nivel 1 nuevo. Los demás récords de los dos escenarios viejos se descartan. La migración corre una sola vez sin duplicar datos. Si no hay forma de migrar, avisame antes de borrar nada.

Actualizá la card: 5 niveles, 15 estrellas (3 por nivel), cantidad de tags del escenario, y una descripción como: "Una línea clasificadora con dos empujadores: desviá cajas con un pulso de pistón, repartilas entre dos rampas con un contador, clasificá aunque el operador cambie la velocidad y separá las cajas por color."
```

### Prompt C-2: Nivel 2 "Reparto cinco y cinco" (nuevo)

```
Creá el nivel 2 de "Clasificación por Empujadores", titulado "Reparto cinco y cinco". Misma escena y tags que el nivel 1, con los dos empujadores en juego.

Lógica esperada: las primeras 5 cajas se desvían a la rampa 1 (Sensor_1 + TON + pulso de Piston_1, como en el nivel 1). Al llegar a 5, un contador (CTU) cambia el destino: las 5 cajas siguientes pasan de largo por el empujador 1 y se desvían a la rampa 2 con Sensor_2 + TON + pulso de Piston_2.

Ítems:
1) Poné en marcha la cinta.
2) Cajas en la rampa 1: 0/5.
3) Cajas en la rampa 2: 0/5.

Fallas: una sexta caja en la rampa 1, una caja en la rampa 2 antes de completar las 5 de la rampa 1, una caja que llega al final de la cinta sin desviarse.

Enunciado sugerido: "Ahora las dos rampas trabajan: mandá las primeras cinco cajas a la rampa 1 y, cuando esté completa, desviá las cinco siguientes a la rampa 2. Un contador decide qué empujador actúa."

Agregá a la ficha del nivel una explicación breve del CTU y de cómo su salida habilita un empujador u otro.
```

### Prompt C-3: Nivel 3 "Clasificación a velocidad variable"

```
Creá el nivel 3 de "Clasificación por Empujadores", titulado "Clasificación a velocidad variable". Es el reparto 5 y 5 del nivel 2 con la mecánica de velocidad del nivel 1 de "Cinta Transportadora con Clasificador": el operador carga la velocidad con −/+ en el display del puesto, el programa la lee en Cinta_1_Velocidad_SP y la escribe en Cinta_1_Velocidad con un tope, porque por encima de cierta velocidad los empujadores no llegan. Reutilizá esa mecánica y sus tags.

Durante la misión la velocidad sube. Definí vos si la sube el alumno con +/− o si el nivel la sube solo (sugerencia: el nivel, después de la caja 5, para obligar a que el programa se adapte). Con más velocidad la caja llega antes al empujador, así que la espera fija de 0,3 s ya no alcanza: el programa tiene que acertar el pulso a la velocidad que esté corriendo (por ejemplo, ajustando la espera según la velocidad o disparando el pistón con otro sensor).

Ítems:
1) Poné en marcha la cinta.
2) La velocidad de la cinta sigue al SP del operador y respeta el tope (se cumple si Cinta_1_Velocidad la escribe el programa).
3) Cajas en el contenedor/rampa 1: 0/5.
4) Cajas en el contenedor/rampa 2: 0/5.

Fallas: velocidad por encima del tope, caja que no se desvía, reparto fuera de orden (igual que el nivel 2).

Enunciado sugerido: "El operador quiere más ritmo: la velocidad llega desde el display y tu programa la aplica con un límite seguro. Repartí cinco y cinco aunque la cinta vaya más rápido."

Antes de implementar, probá en el simulador hasta qué velocidad los empujadores llegan a desviar la caja y fijá el tope con ese valor.
```

### Prompt C-4: Nivel 4 "Clasificar por color" (nuevo)

```
Creá el nivel 4 de "Clasificación por Empujadores", titulado "Clasificar por color". Misma línea que el nivel 2 (sentido REV, dos empujadores, dos rampas), pero los sensores fotoeléctricos Sensor_1 y Sensor_2 se reemplazan por sensores de color, uno antes de cada empujador.

Sensor de color: agregá el componente a la escena y sus tags. Propuesta: un valor INT por sensor (Color_1 y Color_2: 0 = sin caja, 1 = roja, 2 = azul, 3 = otro color), que el alumno compara con una instrucción de comparación. Si el simulador se adapta mejor a una señal BOOL por color, elegí esa opción y explicá la decisión en la ficha del nivel.

Cajas: el generador manda en orden aleatorio (distinto en cada intento) 5 rojas, 5 azules y entre 4 y 6 de otros colores (verde, amarilla), con separación suficiente para que haya una sola caja por vez entre cada sensor y su empujador.

Lógica esperada: roja -> rampa 1 con Piston_1; azul -> rampa 2 con Piston_2; otro color -> pasa de largo hasta el final de la cinta sin fallar.

Velocidad y tiempos fijos: en este nivel la velocidad de la cinta queda en 50 y no se puede modificar, y el temporizado es el estándar del nivel 1 (espera de 0,3 s con TON y pulso de pistón de 1 s). Ubicá los sensores de color en la escena de forma que, con ese temporizado estándar de empujar y volver, la caja caiga en la rampa. Verificalo en el simulador: si no cae, ajustá la posición del sensor, no el tiempo.

Ítems:
1) Poné en marcha la cinta.
2) Cajas rojas en la rampa 1: 0/5.
3) Cajas azules en la rampa 2: 0/5.

Fallas: una caja en la rampa equivocada, una caja de otro color desviada, una roja o azul que llega al final de la cinta.

Enunciado sugerido: "Llegan cajas de colores mezcladas: las rojas van a la rampa 1, las azules a la rampa 2 y el resto sigue de largo. La cinta y los tiempos ya están calibrados: vos solo decidís qué empujador actúa según el color."

Agregá a la ficha del nivel cómo leer el sensor de color y un ejemplo de comparación.
```

### Prompt C-5: Nivel 5 "Clasificación completa" (nuevo, integrador)

```
Creá el nivel 5 de "Clasificación por Empujadores", titulado "Clasificación completa". Integra los niveles 1 a 4: sensores de color, dos empujadores, reparto de 5 cajas por rampa y velocidad de la cinta leída del SP del operador con tope (mecánica del nivel 3).

Diferencia clave: los sensores de color están en otra posición, más lejos de su empujador que en el nivel 4, así que la espera de 0,3 s ya no sirve. El alumno tiene que calcular cuánto tarda la caja en llegar del sensor al empujador: tiempo = distancia / velocidad. Como la velocidad puede cambiar durante la misión, lo ideal es que el programa calcule el preset del TON a partir de la velocidad actual.

Requisitos para que se pueda resolver:
- La ficha del nivel publica la distancia de cada sensor a su empujador (definí vos si es fija o al azar en cada intento leída desde un tag; sugerencia: fija) y la equivalencia de la velocidad de la cinta en m/s (por ejemplo, 100 % = 1 m/s). Medí esa equivalencia en el simulador.
- Verificá que el simulador Ladder tenga operaciones aritméticas (MUL/DIV) y que el TON acepte un preset desde una variable. Si falta algo, agregalo a la librería.
- El generador mantiene una sola caja por vez entre cada sensor y su empujador, para no exigir colas (FIFO) en este nivel.

Cajas: igual que el nivel 4 (5 rojas, 5 azules y otros colores al azar). Durante la misión la velocidad sube, con el mismo criterio que definas para el nivel 3.

Ítems:
1) Poné en marcha la cinta.
2) La velocidad sigue al SP del operador y respeta el tope.
3) Cajas rojas en la rampa 1: 0/5.
4) Cajas azules en la rampa 2: 0/5.

Fallas: velocidad por encima del tope, caja en la rampa equivocada, caja de otro color desviada, roja o azul que llega al final de la cinta.

Enunciado sugerido: "Todo junto: el operador marca la velocidad, los sensores de color están más lejos y el pistón tiene que salir justo cuando llega la caja. Calculá el tiempo de viaje según la distancia y la velocidad, y clasificá cinco rojas y cinco azules."

Agregá a la ficha del nivel la fórmula tiempo = distancia / velocidad con un ejemplo numérico.
```
