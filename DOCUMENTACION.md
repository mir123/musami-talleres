# Documentación: Contador de Tiempo para Talleres de la Alcaldía de San Miguelito

## Descripción General

Aplicación web standalone (HTML/JavaScript) para gestionar y proyectar el tiempo de actividades durante talleres. La aplicación muestra en pantalla completa la actividad actual con un contador de tiempo restante, alertas visuales y sonoras, y permite ajustar dinámicamente los horarios durante la sesión sin modificar los datos guardados permanentemente.

## Características Principales

### 1. Visualización en Pantalla Completa
- Muestra la actividad actual en letras grandes (blanco sobre negro)
- Contador de tiempo restante con alertas visuales:
  - **5 minutos restantes**: Texto naranja
  - **3 minutos restantes**: Texto naranja oscuro
  - **1 minuto restante**: Countdown en segundos (texto rojo, pulsante)
  - **10 segundos restantes**: Beep cada segundo
- Alarma de 2 segundos al finalizar cada actividad
- Reloj con hora actual (HH:MM:SS) en la esquina superior izquierda
- Logo de la Alcaldía en la esquina superior derecha

### 2. Gestión de Actividades
- Agregar, editar y eliminar actividades
- Formato de tiempo 12 horas (AM/PM)
- Validación de colisiones entre actividades
- Persistencia en `localStorage`
- Importar/Exportar CSV
- Pre-llenado automático del formulario con la hora de fin de la actividad anterior

### 3. Control de Tiempo Dinámico (Delta de Tiempo)
Sistema que permite ajustar los horarios durante la sesión sin modificar los datos guardados:

- **Delta en segundos**: Diferencia entre la hora actual y la hora de inicio de la actividad actual
- **Aplicación**: El delta se aplica a todos los eventos para mantener sus relaciones relativas
- **Persistencia**: Los datos originales se mantienen intactos; solo se ajustan visualmente durante la sesión

#### Botones de Control:
- **⏭️ Adelantar**: Pasa a la siguiente actividad (incluso si la actual ya terminó)
  - Calcula: `delta = hora actual - hora original de inicio de la siguiente actividad`
  - Si es negativo, todo empieza más temprano
  
- **⏮️ Retroceder**: Reinicia la actividad actual o pasa a la anterior
  - Si han pasado ≤ 3 segundos: un click reinicia
  - Si han pasado > 3 segundos: requiere doble click (dentro de 1 segundo)
  - Calcula: `delta = hora actual - hora original de inicio de la actividad objetivo`
  
- **⏯️ Pausar/Reanudar**: Pausa el contador
  - Al pausar: inicia un intervalo que aumenta el delta en 1 segundo cada segundo
  - Al reanudar: detiene el intervalo
  - El delta sigue aumentando mientras está pausado
  
- **📅 Resetear**: Restaura el delta a 0, volviendo a los horarios originales

### 4. Resolución Automática de Colisiones
Al editar una actividad que causa colisión con la siguiente, se ofrece un modal con dos opciones:

1. **Mover eventos subsiguientes**: Mueve el siguiente evento (y subsiguientes si es necesario) para que comiencen después del evento editado, manteniendo sus duraciones
2. **Reducir duración del siguiente**: Mueve el siguiente evento para que comience al final del editado y reduce su duración para que termine a la misma hora que terminaba originalmente

### 5. Configuración
- Selección de fuente personalizada (fuentes locales instaladas)
- Manual de usuario integrado
- Visualización del delta de tiempo actual (para debug)
- Botón de resetear delta desde el panel de configuración

## Estructura del Código

### Archivo Principal
- **`contador_tiempo_taller.html`**: Archivo único que contiene todo el código (HTML, CSS, JavaScript)

### Variables Globales Principales

```javascript
let activities = [];              // Array de actividades actuales (con tiempos ajustados)
let originalActivities = [];      // Array de actividades originales (sin ajustes)
let timeDelta = 0;                // Delta de tiempo en segundos
let isPaused = false;             // Estado de pausa
let pauseIntervalId = null;       // ID del intervalo de pausa
let editingIndex = -1;            // Índice de la actividad siendo editada
let pendingEditData = null;       // Datos pendientes para resolución de colisión
```

### Funciones Clave

#### Gestión de Actividades
- `loadActivities()`: Carga actividades desde `localStorage`
- `saveActivities()`: Guarda actividades en `localStorage` y actualiza `originalActivities`
- `updateActivitiesDisplay()`: Actualiza la lista de actividades en el panel de configuración
- `hasCollision(newTime, newDuration, excludeIndex)`: Detecta colisiones considerando el delta

#### Cálculo de Tiempo
- `getTimeRemaining()`: Calcula la actividad actual y tiempo restante
- `getTimeRemainingAtTime(currentMinutes, currentSeconds)`: Versión que acepta un tiempo específico
- `parseTime(timeStr)`: Convierte "HH:MM" a minutos desde medianoche
- `formatTime(minutes)`: Convierte minutos a "HH:MM"
- `formatTime12h(minutes)`: Convierte minutos a formato 12 horas con AM/PM

#### Ajuste de Tiempos
- `adjustActivityTimes()`: Aplica el delta a todas las actividades
  - Convierte tiempos originales a segundos
  - Suma el delta
  - Convierte de vuelta a minutos (redondeando)
  - Maneja cruces de medianoche

#### Control de Tiempo
- `nextActivity()`: Adelanta a la siguiente actividad
- `previousActivity()`: Retrocede a la actividad anterior
- `togglePause()`: Pausa/reanuda el contador
- `resetToSchedule()`: Resetea el delta a 0

#### Resolución de Colisiones
- `showCollisionResolutionModal()`: Muestra el modal con opciones
- `resolveCollisionOption1()`: Mueve eventos subsiguientes manteniendo duración
- `resolveCollisionOption2()`: Reduce duración del siguiente evento
- `closeCollisionModal()`: Cierra el modal

#### Visualización
- `updateDisplay()`: Actualiza la pantalla principal con la actividad actual
- `updateClock()`: Actualiza el reloj de hora actual
- `playBeep()`: Reproduce un beep de 800Hz
- `playAlarm()`: Reproduce una alarma de 2 segundos

## Flujo de Trabajo

### Inicialización
1. Carga actividades desde `localStorage`
2. Crea copia en `originalActivities`
3. Aplica ajustes de tiempo (si hay delta)
4. Inicia intervalos de actualización (100ms para display, 1s para reloj)

### Durante la Sesión
1. El display se actualiza cada 100ms
2. Si hay pausa activa, el delta aumenta 1 segundo cada segundo
3. Los botones de control ajustan el delta según la acción
4. `adjustActivityTimes()` se llama cada vez que cambia el delta

### Al Agregar/Editar Actividades
1. Si hay delta, se aplica el delta inverso a la hora ingresada antes de guardar
2. Esto asegura que al resetear el delta, las relaciones se mantengan
3. Se verifica colisión considerando el delta
4. Si hay colisión con el siguiente evento (solo al editar), se muestra el modal de resolución

## Lógica del Delta de Tiempo

### Concepto
El delta representa la diferencia entre la hora actual y la hora de inicio original de la actividad actual. Se aplica a todos los eventos para mantener sus relaciones relativas.

### Ejemplo
- Actividad original: 14:00 (840 minutos desde medianoche)
- Hora actual: 14:05 (845 minutos)
- Delta: 845 - 840 = 5 minutos = 300 segundos

Todas las actividades se ajustan sumando 300 segundos a sus tiempos originales.

### Al Agregar Eventos con Delta
Si hay un delta de +120 segundos y el usuario ingresa 15:52:
- Se resta el delta: 15:52 - 120 segundos = 15:50
- Se guarda 15:50 como tiempo original
- Al resetear el delta, el evento aparecerá a las 15:50
- Con el delta activo, se mostrará a las 15:52

## Resolución de Colisiones

### Detección
- Solo se activa al editar (no al agregar)
- Solo se muestra si la colisión es con el evento inmediatamente siguiente
- Usa `hasCollision()` que considera el delta al verificar

### Opción 1: Mover Eventos Subsiguientes
1. Actualiza la actividad editada
2. Itera sobre eventos subsiguientes
3. Si hay colisión, mueve el evento para que comience justo después del anterior
4. Mantiene la duración original
5. Continúa hasta que no haya más colisiones

### Opción 2: Reducir Duración
1. Actualiza la actividad editada
2. Mueve el siguiente evento para que comience al final del editado
3. Calcula nueva duración: `duración_original - (nuevo_fin - fin_original)`
4. Si la duración sería ≤ 0, elimina el evento

## Persistencia de Datos

### localStorage
- Clave: `'workshopActivities'`
- Formato: JSON array de objetos `{time, duration, activity}`
- Se actualiza al agregar, editar o eliminar actividades

### CSV
- Formato: `Hora,Duración (minutos),Actividad`
- Exportación: Genera CSV desde `activities`
- Importación: Parsea CSV y valida colisiones antes de importar

## Notas Técnicas

### Precisión de Tiempo
- Todos los cálculos usan segundos enteros (no minutos decimales)
- Se redondea al convertir entre minutos y segundos
- Esto evita problemas de precisión flotante

### Manejo de Medianoche
- Los tiempos se normalizan al rango 0-1440 minutos (0-86400 segundos)
- Si un tiempo ajustado cruza medianoche, se ajusta al mismo día

### Estado de Pausa
- Cuando está pausado, el display muestra "⏸️ PAUSADO"
- El delta sigue aumentando mientras está pausado
- Al reanudar, el contador continúa desde donde se pausó

### Mensajes de Nueva Actividad
- Al adelantar o cuando termina una actividad, se muestra "Nueva actividad iniciada"
- Aparece pequeño arriba del tiempo restante
- Desaparece después de 3 segundos
- El tiempo restante siempre se muestra

## Áreas de Mejora Potencial

1. **Validación de CSV**: Mejorar detección de errores en formato
2. **Undo/Redo**: Implementar historial de cambios
3. **Notificaciones**: Agregar notificaciones del sistema al cambiar de actividad
4. **Temas**: Permitir personalización de colores
5. **Estadísticas**: Mostrar tiempo total de actividades, tiempo transcurrido, etc.
6. **Exportación de Logs**: Guardar historial de cambios de delta durante la sesión

## Dependencias

- Ninguna. La aplicación es completamente standalone.
- Usa APIs nativas del navegador:
  - `localStorage` para persistencia
  - `AudioContext` para sonidos
  - `Date` para manejo de tiempo

## Compatibilidad

- Navegadores modernos que soporten:
  - ES6+ JavaScript
  - CSS Grid/Flexbox
  - Web Audio API
  - localStorage

## Autor y Soporte

Desarrollado por Mir Rodríguez de la Dirección de Planificación Urbana.
Contacto: mirrodriguez@sanmiguelito.gob.pa


