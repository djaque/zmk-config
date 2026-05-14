# ZMK Djaque — Firmware para Eyelash Corne

Configuración personalizada de firmware ZMK para un teclado split **Eyelash Peripherals Corne** (variante con joystick, encoder rotatorio y pantallas nice!view).

---

## Hardware

| Componente | Valor |
|---|---|
| Board izquierdo | `eyelash_corne_left` |
| Board derecho | `eyelash_corne_right` |
| Shield (displays) | `nice_view` (ambos lados) |
| Controladores | Nice!Nano v2 (nRF52840, BLE) |
| Extras | Joystick analógico, encoder rotatorio, RGB underglow (WS2812) |

---

## Estructura del repositorio

```
config/
  eyelash_corne.keymap   # Mapa de teclas y comportamientos (fuente principal)
  eyelash_corne.conf     # Kconfig: sleep, RGB, Bluetooth, debounce, etc.
  west.yml               # Manifest: ZMK v0.3.0 + módulo eyelash_corne
build.yaml               # Matriz de CI (GitHub Actions)
.github/workflows/
  build.yml              # Llama al workflow reutilizable de ZMK v0.3
keymap-diagram.html      # Diagrama interactivo de capas (referencia visual)
```

---

## Versiones fijadas

- **ZMK**: `v0.3.0` (pinned en `west.yml` y en el workflow `@v0.3`)
- **Módulo eyelash_corne**: rama `main` del repo `a741725193/zmk-new_corne`

> No actualizar ZMK a main/nightly sin verificar compatibilidad del módulo. Cambios de API en ZMK nightly pueden romper la build.

---

## Capas del teclado

| # | Nombre | Propósito |
|---|---|---|
| 0 | QWERTY | Capa base — layout ES Latino para macOS |
| 1 | NUMBER | Números, flechas, Bluetooth, atajos Cmd, RGB |
| 2 | SYMBOL | Símbolos, mouse buttons, output USB/BLE |
| 3 | Fn | F1–F12, bootloader, capturas de pantalla, sys_reset |
| 4 | NUMPAD | Teclado numérico (lado derecho) — toggle desde NUMBER+G |

### Thumb cluster (fila inferior, 6 teclas)

```
[ LALT ] [ mo 1 ] [ td_spc_esc ]   [ lt 3 SPACE ] [ mo 2 ] [ LGUI ]
```

- `td_spc_esc`: tap = Space, doble tap = ESC, hold = capa Fn (layer 3)
- `lt 3 SPACE`: tap = Space, hold = capa Fn (layer 3)
- Ambos thumbs internos producen Space/Fn — diseño intencional

### Encoder rotatorio

| Capa | Giro ↑ | Giro ↓ | Botón |
|---|---|---|---|
| QWERTY | Volumen + | Volumen − | Mute |
| NUMBER | Scroll arriba | Scroll abajo | Click derecho |
| SYMBOL | Scroll arriba | Scroll abajo | — |
| Fn | Scroll arriba | Scroll abajo | Mute |

### Joystick

| Capa | Función |
|---|---|
| QWERTY | Teclas de flecha + Enter central |
| NUMBER / SYMBOL / Fn | Mouse movement (mmv) + click izq. central |

---

## Combos

| Combo | Teclas | Capa | Acción |
|---|---|---|---|
| Soft-off | pos 1 + 15 + 29 | cualquiera | Apagado suave del teclado |
| BT_CLR | pos 32 + 37 (V + M en Fn) | Fn (3) | Borrar perfil BT activo |
| BT_CLR_ALL | pos 28 + 32 + 37 (Ctrl + V + M en Fn) | Fn (3) | Borrar todos los perfiles BT |

> BT_CLR y BT_CLR_ALL están en la capa Fn con teclas `&trans` para evitar activaciones accidentales desde home row.

---

## Configuración relevante (`.conf`)

```ini
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=3600000   # Sleep tras 1 hora
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_RGB_UNDERGLOW=y
CONFIG_ZMK_RGB_UNDERGLOW_ON_START=n     # RGB apagado al iniciar
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y
CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y
CONFIG_ZMK_POINTING=y                   # Mouse / pointing habilitado
CONFIG_ZMK_POINTING_SMOOTH_SCROLLING=y
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y         # Mayor alcance BLE
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=8
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=8
CONFIG_ZMK_PM_SOFT_OFF=y
```

---

## Comportamientos personalizados

```c
// Tap-dance: Shift (tap) / Caps Lock (doble tap)
td0: td0 { compatible = "zmk,behavior-tap-dance"; ... }

// Tap-dance: Space (tap) / ESC (doble tap) / Fn layer (hold)
td_spc_esc: td_spc_esc { compatible = "zmk,behavior-tap-dance"; tapping-term-ms = <200>; ... }

// Layer-tap global: 280ms tapping-term, 175ms quick-tap
&lt { tapping-term-ms = <280>; quick-tap-ms = <175>; }
```

---

## CI / Build

El firmware se compila automáticamente en GitHub Actions en cada push/PR.

- Workflow: `.github/workflows/build.yml` → reutiliza `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`
- Artefactos: firmware `.uf2` para `eyelash_corne_left` y `eyelash_corne_right`
- No hay pasos locales de build — todo ocurre en CI. Para flashear, descargar los artefactos del workflow.

---

## Reglas al modificar el keymap

### Actualizar el diagrama HTML
**Siempre que se modifique `eyelash_corne.keymap`, se debe actualizar `keymap-diagram.html`** para reflejar los cambios. El diagrama es la referencia visual principal del layout; si queda desincronizado pierde su utilidad.

### Convención de opciones por tecla en la capa QWERTY
El diagrama debe mostrar **todas las opciones** de cada tecla en la capa QWERTY, indicadas como 1ª, 2ª, 3ª opción según el modificador usado. Esto incluye modificadores que **no sean de capa** (Shift, Option/Alt, Control, Command), por ejemplo:

| Posición en celda | Significado |
|---|---|
| 1ª opción | Tecla sola (tap normal) |
| 2ª opción | + Shift |
| 3ª opción | + Option (⌥) |
| 4ª opción | + Command (⌘) o + Control |

Ejemplo: la tecla `E` mostraría: 1ª `e`, 2ª `E`, 3ª `´e` → `é` (con dead acute del SO).

El objetivo es poder leer el diagrama sin necesidad de memorizar qué produce cada combinación.

---

## Decisiones de diseño y contexto

- **macOS layout: US con ABC Extended**: el teclado físico usa layout US en macOS. ABC Extended provee dead keys vía Option: `RA(E)` = dead acute para tildes, `RA(N)` = dead tilde para ñ. Los símbolos `@ # {} []` funcionan directamente en el SYMBOL layer sin ajustes de layout del SO.
- **ñ/Ñ**: macro mod-morph en la posición de `;` — tap = ñ (envía `RA(N)+N`), hold Shift = Ñ (envía `RA(N)+LS(N)`).
- **Dead acute tildes**: posición LBKT envía `RA(E)` (ABC Extended), luego la vocal produce á é í ó ú.
- **BT_CLR en combos**: fue movido desde la home row porque se activaba accidentalmente. Ahora requiere capa Fn + dos teclas simultáneas.
- **`CONFIG_ZMK_OUTPUT_DEFAULT` eliminado**: no está definido en ZMK v0.3 y aborta el build si se incluye.
- **`ZMK_EXT_POWER` deshabilitado → revertido**: intentar deshabilitar EXT_POWER rompe el linker en esta placa; se dejó el comportamiento por defecto.
- **RGB_UNDERGLOW_ON_START=n**: el RGB empieza apagado para ahorrar batería; se activa manualmente desde la capa NUMBER.
- **Diagrama HTML**: `keymap-diagram.html` es un archivo de referencia interactivo generado junto al keymap. Actualizar siempre que cambie el layout (ver regla arriba).
