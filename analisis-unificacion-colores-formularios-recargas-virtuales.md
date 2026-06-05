# Analisis de unificacion de colores en formularios de Recargas Virtuales - GOLSCRUM - 254

## Objetivo

La mejora apunta a unificar el criterio visual de los campos de los formularios de Recargas Virtuales, principalmente en las pantallas de recarga celular con monto abierto y recarga DirecTV.

El comportamiento esperado es:

- Todo valor ingresado por el usuario debe visualizarse en negro.
- El borde del campo en estado de foco debe mantenerse fucsia.
- Los mensajes renderizados mediante `mat-error` deben mantenerse fucsia.
- Los placeholders o textos indicativos dentro de los campos deben visualizarse en un gris derivado del negro.
- Los prefijos fijos que no son ingresados por el usuario, como `$` y `05700`, deben visualizarse en el mismo criterio gris.

Para esta card, la expresion "mensaje informativo" se interpreta como el placeholder dentro del campo. No corresponde cambiar los `mat-error`, ya que su color fucsia es correcto.

## Alcance detectado

- Recarga celular con monto abierto:
  - Cambiar a gris el placeholder `Ingresa el importe a cargar`.
  - Cambiar a gris el signo `$` fijo del campo Importe.
  - Mantener en negro el monto ingresado por el usuario.
- Recarga DirecTV:
  - Cambiar a gris el placeholder `Ingresa los ultimos 13 digitos`.
  - Cambiar a gris el prefijo fijo `05700`.
  - Cambiar a gris el placeholder `Ingresa el importe a cargar`.
  - Cambiar a gris el signo `$` fijo del campo Importe.
  - Mantener en negro los 13 digitos y el importe ingresados por el usuario.
- Estados de foco:
  - Mantener el borde fucsia de los campos de los formularios involucrados.
- Estados de error:
  - Mantener fucsias todos los mensajes `mat-error`.
  - Mantener el criterio visual fucsia de los campos invalidos.
- Recarga de transporte:
  - No aparece como caso explicito en las capturas de la card.
  - Debe revisarse por posible impacto, ya que comparte la clase global `recarga-form-accent` y usa el mismo patron de placeholder y prefijo `$`.

## Tipo de tarea

- Mejora UX/UI.
- Bug visual.

Se clasifica de esta manera porque no requiere cambiar validaciones ni reglas de negocio. El problema se encuentra en la asignacion de colores a distintos tipos de contenido dentro de los campos.

## Impacto estimado

Impacto medio.

Aunque el ajuste es visual y puntual, los estilos actuales estan centralizados en `src/styles/styles.scss` mediante la clase `recarga-form-accent`, compartida por los formularios de celular, DirecTV y transporte. Un cambio sobre estos selectores puede afectar varias pantallas, por lo que debe aplicarse de forma controlada y validarse en los tres flujos.

## Estado actual del codigo

Los formularios de celular, DirecTV y transporte utilizan simultaneamente las clases `fucsia-error-accent` y `recarga-form-accent`.

En `src/styles/styles.scss`:

- `fucsia-error-accent` centraliza correctamente el color fucsia de focus, error y `mat-error`.
- `recarga-form-accent` fuerza actualmente a fucsia:
  - Los placeholders de los inputs.
  - Los `mat-hint`.
  - Los prefijos de Angular Material mediante `.mat-mdc-form-field-text-prefix`.

Esto mezcla estados visuales que deben tener semanticas diferentes. Los errores y el focus deben permanecer fucsia, mientras que placeholders y prefijos fijos deben ser grises.

Ademas, los templates agregan directamente la clase `text-fucsia-la` a prefijos que la card solicita en gris:

- Signo `$` del importe de recarga celular.
- Prefijo `05700` de DirecTV.
- Signo `$` del importe de DirecTV.
- Signo `$` del importe de transporte, como posible impacto compartido.

No se detecta una necesidad de modificar la logica TypeScript, las validaciones, los formularios reactivos ni la construccion del codigo completo de DirecTV.

## Propuesta tecnica

### 1. Separar el color de errores del color informativo

Mantener sin cambios la clase `fucsia-error-accent` y sus variables de Angular Material, porque actualmente cubre el borde de focus, los estados invalidos y el texto de `mat-error`.

Ajustar los selectores de `recarga-form-accent` para que los placeholders y prefijos fijos utilicen un gris consistente con el resto de los campos. Conviene reutilizar un token o tono ya presente en el proyecto, por ejemplo el criterio `slate-500` usado en las flechas de los selects, en lugar de introducir un color aislado.

### 2. Eliminar overrides fucsias puntuales en los prefijos

Quitar o reemplazar la clase `text-fucsia-la` de los prefijos `$` y `05700` en los templates involucrados. El color deberia resolverse desde una clase semantica compartida o desde `recarga-form-accent`.

Los prefijos `0` y `15` del telefono tambien deben conservar el criterio gris por tratarse de valores fijos y no ingresados por el usuario.

### 3. Asegurar el color negro del texto ingresado

Verificar que `.mat-mdc-input-element` y el valor seleccionado de `mat-select` mantengan el color negro o el token de texto principal del tema cuando el usuario ingresa o selecciona un valor.

El selector no debe afectar `::placeholder`, prefijos ni mensajes de error. En DirecTV debe distinguirse visualmente el prefijo fijo `05700` gris de los 13 digitos ingresados en negro.

### 4. Mantener sin cambios los estados correctos

No modificar:

- El color fucsia de `mat-error`.
- El borde fucsia en focus.
- El borde y estado fucsia de campos invalidos.
- Los validadores y textos de error.
- La concatenacion del prefijo `05700` con la parte variable enviada al backend.
- El formato y las reglas de montos abiertos o cerrados.

## Criterio de implementacion sugerido

1. Definir o reutilizar un gris del sistema para placeholders y prefijos fijos.
2. Ajustar los selectores globales de `.recarga-form-accent` sin alterar `.fucsia-error-accent`.
3. Remover las clases `text-fucsia-la` de los prefijos alcanzados por la card.
4. Verificar que los valores ingresados se rendericen en negro tanto en estado normal como en focus.
5. Validar celular y DirecTV con campos vacios, completos e invalidos.
6. Revisar transporte por compartir los estilos, sin ampliar su logica funcional.
7. Evitar cambios en TypeScript salvo que una prueba demuestre que el valor visual se genera mediante una clase dinamica.

## Archivos candidatos a revisar

### Archivos probablemente involucrados

- `src/styles/styles.scss`: contiene los selectores globales que actualmente colorean placeholders, hints y prefijos en fucsia.
- `src/app/modules/recargas-virtuales/recargar-celular/recarga-celular.component.html`: declara el placeholder de Importe y el prefijo `$` con `text-fucsia-la`.
- `src/app/modules/recargas-virtuales/recargar-directv/recarga-direct-tv.component.html`: declara los placeholders, el prefijo `05700` y el signo `$` con `text-fucsia-la`.

### Archivos a revisar por posible impacto

- `src/app/modules/recargas-virtuales/recargar-transporte/recarga-transporte.component.html`: comparte `recarga-form-accent` y utiliza un prefijo `$` fucsia.
- `src/app/modules/recargas-virtuales/recargar-celular/recarga-celular.component.scss`: contiene estilos locales de selects que usan un gris de referencia.
- `src/app/modules/recargas-virtuales/recargar-directv/recarga-direct-tv.component.scss`: contiene estilos locales de selects que usan un gris de referencia.
- `src/app/modules/recargas-virtuales/recargar-transporte/recarga-transporte.component.scss`: permite comprobar consistencia con el flujo de transporte.
- `src/app/modules/recargas-virtuales/recargar-celular/recarga-celular.component.spec.ts`: revisar si conviene agregar una validacion estructural minima del template.
- `src/app/modules/recargas-virtuales/recargar-transporte/recarga-transporte.component.spec.ts`: revisar solo por impacto compartido.

### Archivos que conviene evitar tocar

- `src/app/modules/recargas-virtuales/recargar-celular/recarga-celular.component.ts`: la logica de montos y validaciones no forma parte del problema visual.
- `src/app/modules/recargas-virtuales/recargar-directv/recarga-direct-tv.component.ts`: el prefijo `05700`, las longitudes y validaciones deben conservarse.
- `src/app/modules/recargas-virtuales/recarga/recarga.component.ts`: no debe modificarse la composicion de datos enviados al backend.
- `src/app/shared/mat-errors/mat-errors.ts`: los errores existentes deben continuar funcionando y mostrandose en fucsia.
- `src/app/modules/recargas-virtuales/recargas-virtuales.service.ts`: no hay cambios de integracion requeridos.

## Casos a validar

- Recarga celular de monto abierto sin importe:
  - Placeholder de Importe gris.
  - Signo `$` gris.
  - Hint inferior legible y sin cambio semantico no solicitado.
- Recarga celular de monto abierto con importe valido:
  - Importe ingresado en negro.
  - Signo `$` gris.
  - Borde fucsia mientras el campo tiene focus.
- Recarga celular con importe invalido:
  - `mat-error` fucsia.
  - Estado invalido del campo fucsia.
  - El valor ingresado no debe adoptar el color del placeholder.
- DirecTV sin codigo:
  - `05700` gris.
  - Placeholder de los ultimos 13 digitos gris.
- DirecTV con codigo parcial o completo:
  - `05700` gris.
  - Parte ingresada por el usuario en negro.
- DirecTV con codigo invalido:
  - Mensaje `mat-error` fucsia.
  - El prefijo fijo debe continuar gris.
- DirecTV sin importe:
  - Signo `$` y placeholder grises.
- DirecTV con importe ingresado:
  - Signo `$` gris.
  - Importe en negro.
- Validar focus en todos los campos involucrados:
  - Borde fucsia uniforme.
- Validar monto cerrado en celular y DirecTV:
  - Valores seleccionados legibles y en color de texto principal.
  - Flechas y textos auxiliares sin regresiones.
- Validar desktop y mobile.
- Revisar el formulario de transporte para confirmar que el cambio compartido no introduzca inconsistencias.
- Confirmar un caso donde no debe cambiar nada:
  - Todos los `mat-error` deben seguir viendose fucsias.


## Resumen final

Conviene resolver primero la separacion semantica de colores en `src/styles/styles.scss`: errores y focus en fucsia, placeholders y prefijos fijos en gris, y valores ingresados en negro. Luego deben eliminarse los overrides `text-fucsia-la` de `$` y `05700` en los templates de celular y DirecTV.

Antes de cerrar la card se deben validar estados vacios, con datos, focus y error, comprobando especialmente que los `mat-error` permanezcan fucsias. No deben modificarse validaciones, reglas de montos, composicion del codigo DirecTV ni contratos con backend.
