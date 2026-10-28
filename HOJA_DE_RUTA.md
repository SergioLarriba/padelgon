# Padelgon: hoja de ruta

Próximas funcionalidades tras analizar el mercado de apps de pádel (octubre de 2026), en el orden en que conviene implementarlas, y qué hace falta para publicar en Google Play y App Store.

## Dónde estamos

Padelgon es una web instalable (PWA) en un solo `index.html`. Los datos se guardan solo en el móvil (`localStorage`). Hoy ya tiene:

- Marcador con ventaja, punto de oro, Star Point, tie-break y súper tie-break.
- Saque y cambio de campo automáticos, primer y segundo saque, y deshacer fiable: el partido se recalcula desde la lista de puntos.
- Estadísticas por pareja y por jugador: golpes ganadores, errores, saque, bolas de break y rachas.
- Balance de cada jugador y gráfica de cada set.
- Imagen del resultado y de las estadísticas para compartir.
- Historial, estadísticas generales y copia de seguridad en JSON.

## Lo que dice el mercado

| Tipo de app | Ejemplos | Precio habitual | Qué aprendemos |
|---|---|---|---|
| Marcador simple | Padel Watch, iWatch Padel, Padel Point | Gratis o pago único de unos 3 € | Solo un marcador no se puede cobrar |
| Marcador con estadísticas manuales | Padel scoreboard, Padel Stats AI, Padel Stats Pro, Padela, XPadel Score, PADEL'EM | De gratis a 50–70 €/año | Es nuestra competencia directa. Casi todas se usan **desde el reloj** y varias apuntan el **tipo de golpe** |
| Narración por voz | Padel Speaker | Suscripción con 14 días de prueba | Que el marcador se cante en voz alta es una función que se paga |
| Análisis por vídeo o sensor | Padelize, PadelIQ, Padelplay | 9–10 €/mes, 50–100 €/año, sensor de 119 € | Aquí está el dinero, pero hace falta mucha inversión (IA o hardware) |
| Para entrenadores | Dartfish | 60–80 €/mes | No hay casi nada pensado para academias con muchos alumnos. **Es el hueco** |

Conclusiones:
1. Las estadísticas detalladas ya no nos distinguen por sí solas: hay al menos seis apps que las tienen.
2. El mayor obstáculo es **apuntar cada punto mientras juegas**. La competencia lo resuelve con el reloj. Nosotros podemos resolverlo además con alguien que apunte desde fuera de la pista (un entrenador o un espectador).
3. **Modo entrenador o academia** es el espacio menos cubierto y donde mejor encaja lo que ya tenemos.

---

## Fase 0: Validar sin programar (2–4 semanas)

Antes de invertir más, comprobar que la app se usa.

- [ ] Pasar la app a 10–20 jugadores del club y a 2–3 entrenadores.
- [ ] Preguntar a los entrenadores: ¿apuntarían los partidos de sus alumnos? ¿qué informe querrían? ¿cuánto pagarían al mes?
- [ ] Medir: ¿cuántos siguen usándola al cabo de un mes? ¿cuántos activan «Apuntar cómo acaba cada punto»?

**Para seguir:** al menos un 30 % la sigue usando al cabo de un mes, o al menos un entrenador dice que pagaría.

---

## Fase 1: Base imprescindible

Sin esto, ninguna versión de pago es creíble.

### 1.1 Pruebas del motor de puntuación
Antes de que el código crezca, hay que blindar `step()` y `replay()`.
- Pruebas automáticas para cada modo de juego, tie-breaks, súper tie-break, orden de saque, cambios de campo y deshacer.
- Sacar el motor a un módulo propio (`motor.js`) para poder probarlo con Node y reutilizarlo después en el reloj y en el servidor.

### 1.2 Cuentas y sincronización en la nube
- Inicio de sesión con Google, Apple o enlace por email.
- Backend con un servicio gestionado: **Supabase** (Postgres y autenticación) o **Firebase**. Para empezar no hace falta servidor propio.
- La app sigue funcionando sin conexión: se guarda en local y se sincroniza cuando hay red.
- Al iniciar sesión por primera vez, migrar lo que haya en `localStorage`.
- Mantener la exportación e importación en JSON.

### 1.3 Partidos compartidos entre los cuatro jugadores
- Enlace de solo lectura al partido, que se abre aunque no tengas la app.
- Cada jugador puede «reclamar» su nombre del partido para que le cuente en sus estadísticas.

### 1.4 Apuntar desde otro móvil
- Un móvil (el de un entrenador, un espectador o el jugador que descansa) lleva el marcador, y los demás lo ven en directo.
- Es la respuesta a «nadie apunta mientras juega» sin necesitar reloj.
- Requiere tiempo real: canales de Supabase o Firestore.

### 1.5 Que apuntar cueste menos
- Botones de detalle más grandes y con atajos: por ejemplo, mantener pulsado para «error no forzado».
- Opción «solo marcador» clara para quien no quiere estadísticas.
- Vibración distinta para fin de juego, de set y de partido.

### 1.6 Cantar el marcador en voz alta
- Usar `speechSynthesis` del navegador: «Quince, cero», «Juego para Sergio y Ana», «Star Point».
- Poco trabajo y mucho efecto, y la competencia lo cobra (Padel Speaker).

### 1.7 Evolución del jugador a lo largo del tiempo
- Tendencias por mes: % de puntos ganados, ganadores por partido, errores no forzados por partido y % de primeros saques.
- **Mejores compañeros** y **rivales más difíciles**: con quién ganas más y contra quién pierdes más.
- Racha de victorias y nivel estimado (tipo Elo) a partir de los resultados.

### 1.8 Idioma inglés
- Pasar los textos a un archivo de traducciones. Inglés como mínimo: el mercado de pádel crece fuera de España (Reino Unido, Suecia, Países Bajos, Estados Unidos y Oriente Medio).

---

## Fase 2: Diferenciación con un modo entrenador o academia

Es lo que justifica cobrar.

### 2.1 Alumnos y grupos
- El entrenador crea alumnos (que no necesitan cuenta) y los organiza en grupos o clases.
- Si el alumno se crea una cuenta, puede vincularla y ver sus datos.

### 2.2 Tipo de golpe
Al apuntar un ganador o un error, elegir el golpe: **saque, resto, volea, bandeja, víbora, remate, globo, salida de pared, chiquita**.
- Estadísticas por golpe: «el 40 % de tus errores no forzados son de bandeja».
- Para jugadores, opcional y oculto por defecto. Para entrenadores, activado.

### 2.3 Posición en pista
- Drive o revés de cada jugador, para comparar el rendimiento según el lado.

### 2.4 Notas y objetivos
- Notas del entrenador en cada partido y en cada alumno.
- Objetivos medibles («menos de 5 errores no forzados por set») con su seguimiento en el tiempo.

### 2.5 Informes
- Informe del alumno por periodo, en PDF o imagen: evolución, golpes fuertes y débiles, y notas.
- Enviarlo al alumno o a los padres con un enlace.
- Sirve también como herramienta comercial del entrenador.

### 2.6 Entrenamientos, no solo partidos
- Sesiones de ejercicios con conteo (por ejemplo, «20 bandejas, 14 dentro»), además de partidos.

### 2.7 Panel web para el entrenador
- Ver todos los alumnos en pantalla grande y comparar grupos. Se puede reutilizar la misma web.

**Privacidad:** las academias tienen alumnos **menores**. Hacen falta política de privacidad, consentimiento de padres o tutores, la opción de borrar los datos de un alumno y servidores en la UE (RGPD).

---

## Fase 3: Reloj

Es lo que más usa la competencia, pero el proyecto más grande. Solo merece la pena si las fases anteriores funcionan.

- **Wear OS**: app nativa en Kotlin con Compose for Wear OS. Casi nadie la ofrece (en la búsqueda, solo PADEL'EM), así que es una oportunidad.
- **Apple Watch**: app nativa en Swift con SwiftUI. Necesita Mac, Xcode y la cuenta de desarrollador de Apple.
- El reloj solo apunta puntos (toque o gesto por pareja) y envía la lista de puntos. El móvil y la nube recalculan todo con el mismo motor de la fase 1.1.

---

## Fase 4: Monetización

### Propuesta de planes

| | Gratis | Pro (jugador) | Entrenador |
|---|---|---|---|
| Marcador completo, todos los modos | ✓ | ✓ | ✓ |
| Imagen del resultado | ✓ | ✓ | ✓ |
| Estadísticas del partido | ✓ | ✓ | ✓ |
| Historial | Últimos 10 partidos | Ilimitado | Ilimitado |
| Sincronización y partidos compartidos | ✓ | ✓ | ✓ |
| Voz del marcador | — | ✓ | ✓ |
| Evolución, compañeros y rivales | Resumen | Completo | Completo |
| Tipo de golpe y estadísticas por golpe | — | ✓ | ✓ |
| Exportar estadísticas e informes | Imagen del partido | ✓ | ✓, con informes por alumno |
| Alumnos, grupos, notas y objetivos | — | — | ✓ |
| Panel web | — | — | ✓ |
| **Precio orientativo** | 0 € | 2,99 €/mes o 19,99 €/año | 9,99–19,99 €/mes según el número de alumnos |

Referencias: PADEL'EM cobra 49,99–69,99 €/año y Dartfish 60–80 €/mes. Entrar por debajo de los dos da margen para competir.

### Requisitos técnicos
- **Comprobar el pago en el servidor.** Hoy todo el código está en el navegador, así que cualquier bloqueo hecho solo en la app se puede saltar.
- Con la app en Google Play o App Store, usar su sistema de pago (Google Play Billing o compras de Apple), por ejemplo con **RevenueCat**, que gestiona los dos.
- Para la versión web, Stripe o Lemon Squeezy.
- Prueba gratis de 7 a 14 días para el plan Pro y el de entrenador.

---

## ¿Hay que subirla a Google Play o a la App Store?

**Respuesta corta:** a **Google Play sí**, en cuanto quieras cobrar o que te encuentren. A la **App Store, más adelante**, cuando haya demanda en iPhone o vayas a hacer la app del Apple Watch.

### Seguir solo como web (como ahora)
- ✅ Gratis, actualizaciones al instante y sin revisiones.
- ❌ Nadie la encuentra buscando «marcador pádel».
- ❌ La gente confía menos en una web que en una app de la tienda.
- ❌ Cobrar es más difícil.
- ❌ En iPhone, las web instalables tienen limitaciones: notificaciones, pantalla siempre encendida y que el sistema puede borrar los datos guardados si no se usan.

**Vale para la fase 0 y para el uso gratuito.**

### Google Play: recomendado
- Se puede publicar **la misma web, sin reescribirla**, envuelta como app de Android (TWA) con **Bubblewrap** o **PWABuilder**.
- Coste: pago único de 25 $ por la cuenta de desarrollador.
- Para cuentas personales nuevas, Google exige una **prueba cerrada con al menos 12 personas durante 14 días** antes de publicar. Los requisitos cambian, así que conviene comprobarlos al crear la cuenta.
- Hace falta: política de privacidad publicada, ficha con capturas, formulario de seguridad de datos y clasificación de contenido.
- Es la vía obligatoria para la app de Wear OS (fase 3).

Pasos:
1. [ ] Publicar la web en un dominio propio (por ejemplo, `padelgon.app`) en lugar de GitHub Pages, para no depender del usuario de GitHub.
2. [ ] Añadir `/.well-known/assetlinks.json` para vincular el dominio con la app.
3. [ ] Generar el paquete Android con PWABuilder o Bubblewrap.
4. [ ] Crear la cuenta de Google Play Console y hacer la prueba cerrada.
5. [ ] Escribir la política de privacidad y la ficha de la tienda (en español e inglés).

### App Store: más adelante
- Cuenta de desarrollador: 99 $ al año. Hace falta un Mac con Xcode.
- Apple **suele rechazar las apps que son solo una web envuelta** si no aportan nada de app nativa. Habría que envolverla con **Capacitor** y añadir funciones nativas: notificaciones, widgets o la pantalla siempre encendida de forma fiable.
- La app del Apple Watch tiene que ser nativa de todas formas.
- Para pagos dentro de la app, Apple obliga a usar su sistema (15 % de comisión en el programa para pequeños desarrolladores).

**Hacerlo cuando** haya usuarios de iPhone pidiéndolo o vayas a hacer la app del reloj.

---

## Fuera de alcance por ahora

- **Análisis de partidos por vídeo con IA** (Padelize, PadelIQ): requiere modelos de visión, servidores con GPU y mucha inversión. Se puede reconsiderar más adelante, por ejemplo para marcar los momentos clave en un vídeo grabado aparte.
- **Sensores propios en la pala** (Padelplay): es hardware.
- **Reservar pistas o buscar partido**: Playtomic domina ese terreno y no compensa competir con él.

## Resumen del orden

1. **Fase 0**: validar con el club y los entrenadores.
2. **Fase 1**: pruebas del motor, nube, partidos compartidos, apuntar desde otro móvil, voz y evolución → **publicar en Google Play**.
3. **Fase 2**: modo entrenador → **empezar a cobrar** (planes Pro y Entrenador).
4. **Fase 3**: Wear OS, y después Apple Watch → **App Store**.

---

*Precios y funciones de la competencia consultados en octubre de 2026 en fichas de App Store y Google Play, webs oficiales y reseñas (padeltech.io, padelmatrix.net). Cambian a menudo: revísalos antes de fijar precios.*
