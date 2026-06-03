# Analisis de mejora UX - Mensajes de error e informativos-GOLSCRUM-248

## Objetivo

Unificar visualmente los mensajes mostrados en GOL para que sigan un criterio consistente segun su tipo:

- Mensajes de error: texto, icono y borde en fucsia; fondo con tinte/grisado fucsia.
- Mensajes informativos: texto, icono y borde en azul; fondo con tinte/grisado azul.

La mejora apunta a reemplazar el criterio visual rojo actual en errores por fucsia, manteniendo una lectura clara del estado del mensaje y alineandolo con la identidad visual de la plataforma.

Importante: no se deben modificar todos los mensajes azules de forma masiva. Los mensajes que realmente son informativos deben seguir siendo azules, por ejemplo avisos del estilo "chequea tu e-mail" u otros mensajes de ayuda/recordatorio. El ajuste aplica sobre mensajes que semanticamente son errores, aunque hoy esten declarados o estilizados como `info`, `warn` o con algun azul informativo.

## Alcance detectado

Los casos a revisar son:

- Login: alertas de error de inicio de sesion, reemplazando rojo por fucsia.
- Datos personales: mensajes de error en flujos con preguntas de seguridad.
- Pagina principal: mensajes informativos dentro de las cards/secciones de productos.
- Resumen: mensajes de error en `/resumen;fecha=...`.
- Cuotificate: mensajes de error en `/auth/cuotificate`.
- Recuperacion de clave: mensajes de error en el flujo de "Olvide mi clave".
- Solicitud adicional: tratar el mensaje "Lo sentimos..." como error, por lo tanto debe aplicar el criterio fucsia.
- Casos donde se use `info` o `warn` para comunicar un error real. Ejemplo detectado: en "Tus prestamos", el mensaje "Ha ocurrido un error al obtener la informacion de sus prestamos" se muestra azul, pero debe tratarse como error y pasar al criterio fucsia.

## Estado actual del codigo

El componente base para la mayoria de los mensajes es `fuse-alert`:

- `src/@fuse/components/alert/alert.component.ts`
- `src/@fuse/components/alert/alert.component.html`
- `src/@fuse/components/alert/alert.component.scss`

Actualmente `fuse-alert` ya centraliza estilos por:

- `appearance`: `border`, `fill`, `outline`, `soft`.
- `type`: `primary`, `accent`, `warn`, `basic`, `info`, `success`, `warning`, `error`.

El problema principal es que los tipos `error` y `warn` siguen usando paletas rojas/warn en varias apariencias, especialmente en `outline`, que es la mas usada en los ejemplos del DOC.

Tambien existe una base de variables/estilos utiles en `src/styles/styles.scss`:

- `--fucsia-la: #EB3AB0`
- `.fucsia-error-accent`

Eso permite reutilizar el color institucional sin introducir valores sueltos.

## Propuesta tecnica

### 1. Centralizar colores de alertas en variables globales

Agregar o extender variables en `src/styles/styles.scss` para evitar colores hardcodeados:

```scss
:root {
    --fucsia-la: #EB3AB0;
    --gol-alert-error-text: var(--fucsia-la);
    --gol-alert-error-border: var(--fucsia-la);
    --gol-alert-error-bg: rgba(235, 58, 176, 0.12);
    --gol-alert-info-text: #0D2B88;
    --gol-alert-info-border: #6EA8FF;
    --gol-alert-info-bg: rgba(13, 43, 136, 0.08);
}
```

Los valores exactos de azul/fondo pueden ajustarse segun el criterio visual de QA/diseno.

### 2. Ajustar `fuse-alert` como fuente principal de estilos

Modificar `src/@fuse/components/alert/alert.component.scss` para que:

- `fuse-alert-type-error` use fucsia en texto, icono y borde.
- `fuse-alert-type-warn` se evalue por uso semantico: si se usa como error, llevar ese caso a fucsia; si se usa como advertencia real, dejarlo separado.
- `fuse-alert-type-info` mantenga azul en texto, icono, borde y fondo cuando sea informacion real.
- `appearance="outline"` sea prioridad, porque es el patron que aparece en las capturas.

Esto evita tener que corregir modulo por modulo cuando todos usan el mismo componente.

### 3. Revisar iconografia de error

En `src/@fuse/components/alert/alert.component.html`, el `type="error"` actualmente usa `heroicons_solid:x-circle`.

Para no tocar mas de lo requerido, la propuesta es mantener el mismo icono de error que ya viene usando la aplicacion y cambiar solo el criterio visual: color, borde, fondo y tipo del mensaje cuando corresponda.

### 4. Auditar usos puntuales de alertas

Aunque el cambio central deberia cubrir la mayoria de los casos, conviene revisar componentes con estilos custom o configuraciones propias.

La auditoria debe distinguir entre:

- `info` real: se conserva azul.
- `warn` real: se conserva como advertencia si no representa un error.
- `info` o `warn` usado como error: se corrige a `error` o se lleva visualmente al criterio fucsia.

Archivos/modulos candidatos:

- Login:
  - `src/app/modules/auth/sign-in/sign-in_v1.component.html`
  - `src/app/modules/auth/sign-in/sign-in_v1.alerts.ts`
  - `src/app/modules/auth/sign-in/sign-in_v1.component.ts`
- Olvide mi clave:
  - `src/app/modules/auth/forgot-password/forgot-password.component_v2.html`
  - `src/app/modules/auth/forgot-password/alerts.ts`
  - `src/app/modules/auth/forgot-password/forgot-password.component.ts`
- Datos personales:
  - `src/app/modules/datos-personales/components/datos-personales/datos-personales.component.html`
  - `src/app/modules/datos-personales/components/cambio-clave/*`
  - `src/app/modules/datos-personales/components/cambio-celular/*`
  - `src/app/modules/datos-personales/components/cambio-email/*`
- Pagina principal:
  - `src/app/modules/home-page/home-page.component.html`
  - `src/app/modules/home-page/components/prestamos-error/*`
  - `src/app/modules/tarjeta-de-credito/components/carousel/carousel-tarjetas.component_v2.html`
- Resumen:
  - `src/app/modules/resumenes/components/resumen/resumen.component.html`
  - `src/app/modules/resumenes/components/resumen/resumen.component.ts`
  - `src/app/modules/resumenes/resumenes.component.ts`
- Cuotificate:
  - `src/app/modules/cuotificate/cuotificate.component.html`
  - `src/app/modules/cuotificate/cuotificate.component.ts`
- Solicitud adicional:
  - `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.component.html`
  - `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.component.ts`
- Alertas compartidas:
  - `src/app/shared/alerts/alerts.ts`


## Criterio de implementacion sugerido

1. Definir tokens globales para error e info.
2. Ajustar `fuse-alert` para aplicar el criterio por `type`.
3. Revisar si `warn` debe comportarse como error o como advertencia independiente.
4. Revisar componentes del alcance para detectar overrides/custom styles que sigan forzando rojo.
5. Corregir usos puntuales donde el tipo del mensaje este mal asignado.
6. Validar visualmente los escenarios del DOC.

Ejemplos de criterio:

- Mantener azul: mensajes de informacion real, ayuda, recordatorios o confirmaciones no bloqueantes.
- Pasar a fucsia: mensajes con "ha ocurrido un error", "no fue posible", "no hemos podido", "lo sentimos" o equivalentes, incluso si hoy estan implementados como `type="info"` o `type="warn"`.
- Mantener iconos: usar el mismo icono de error existente para evitar cambios visuales fuera del alcance pedido.

## Confirmaciones realizadas

- Los `mat-error` ya estan cubiertos por la tarjeta anterior: existe `.fucsia-error-accent` en `src/styles/styles.scss`, con variables de Angular Material apuntando a `--fucsia-la`.
- En adicionales, el mensaje "Lo sentimos..." ya esta modelado como `type: 'error'` en `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.component.ts`, por lo que debe verse con el criterio de error fucsia.
- Los mensajes informativos reales no deberian cambiar de estilo. La implementacion debe evitar un cambio global que convierta todo `info` en fucsia.

## Resumen

La mejora conviene encararla desde el componente base `fuse-alert`, no como parches por pantalla. El cambio principal seria reemplazar el rojo actual de errores por fucsia institucional, dejar los informativos reales en azul y corregir los casos donde un error este modelado como `info` o `warn`. Luego se revisarian casos puntuales con estilos custom o configuraciones inconsistentes.
