# Analisis de mejoras UX en Solicitud de Adicionales

## Objetivo

La mejora apunta a agilizar la carga de datos y hacer mas evidente la forma de interactuar con los campos del formulario de Solicitud de Adicionales de GOL.

El comportamiento esperado es:

- Reemplazar la seleccion de fecha de nacimiento mediante calendario por ingreso manual.
- Permitir el ingreso de fecha en formato `dd/mm/aaaa`.
- Mostrar una referencia visible del formato esperado.
- Validar automaticamente el formato, la existencia de la fecha y la edad minima requerida.
- Incorporar busqueda y filtrado en los catalogos extensos de Nacionalidad, Ocupacion y Sucursal de entrega.
- Mostrar placeholders descriptivos antes de seleccionar una opcion.
- Mejorar el contraste del icono desplegable en los selects indicados por la card.
- Reducir la cantidad de interacciones necesarias sin cambiar las reglas de negocio ni el contrato actual con backend.

## Alcance detectado

- Solicitud de Adicionales, primer paso del formulario.
- Campo Fecha de nacimiento:
  - Ingreso manual en formato `DD/MM/AAAA`.
  - Eliminacion del mecanismo visible de seleccion por calendario.
  - Placeholder con el formato esperado.
  - Validacion de formato, fecha real y edad minima.
- Campo Nacionalidad:
  - Reemplazar la lista extensa por un autocomplete con filtrado por texto.
  - Incorporar un placeholder descriptivo.
- Campo Ocupacion:
  - Reemplazar la lista extensa por un autocomplete con filtrado por texto.
  - Incorporar un placeholder descriptivo.
- Campo Sucursal de entrega:
  - Reemplazar la lista extensa por un autocomplete con filtrado por texto.
  - Incorporar un placeholder descriptivo.
- Campos con listas desplegables:
  - Mejorar visibilidad y contraste de la flecha en Tipo/Numero de documento, Genero, Estado civil y Vinculo, sujeto a la aclaracion indicada en pendientes.
- Flujo de envio:
  - Mantener los IDs de catalogo y el formato de request que actualmente consume backend.

## Tipo de tarea

- Mejora UX/UI.
- Validacion/formulario.
- Accesibilidad.

Es una mejora de UX porque reduce clics y facilita la busqueda. Tambien afecta validaciones del formulario, ya que el texto escrito en un autocomplete no debe considerarse valido hasta seleccionar una opcion real. El contraste y la identificacion de controles desplegables tienen impacto de accesibilidad visual.

## Impacto estimado

Impacto medio.

El cambio se concentra en el primer paso del formulario, pero modifica varios controles, el tipo de interaccion de la fecha y la forma de representar catalogos. Debe preservarse el modelo actual del formulario porque el envio espera una fecha `DateTime` y IDs de catalogo.

No se detecta necesidad de cambiar endpoints ni estructuras de respuesta del backend.

## Estado actual del codigo

El formulario se encuentra centralizado en:

- `src/app/modules/tarjeta-de-credito/pages/adicionales/step1/step1/step1.component.html`
- `src/app/modules/tarjeta-de-credito/pages/adicionales/step1/step1/step1.component.ts`
- `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.component.ts`

### Fecha de nacimiento

Actualmente el campo utiliza:

- Un input asociado a `mat-datepicker`.
- Un `mat-datepicker-toggle` con icono `keyboard_arrow_down`.
- Un `MatDatepicker`.
- El adaptador compartido `MyDateAdapter`.
- Locale `es-AR`.
- Validadores `required`, fecha minima desde 1900 y edad minima de 14 anos.

El adaptador compartido ya contempla el parseo de strings con formato `dd/MM/yyyy`. Esto permite mantener el input conectado al pipeline de Angular Material para que el valor valido siga llegando como `DateTime`, aun si se elimina el acceso visual al calendario.

El envio actual ejecuta:

```ts
(step1Values.fechaNacimiento as DateTime).toFormat('dd-MM-yyyy')
```

Por este motivo no conviene reemplazar el campo por un input de texto desacoplado que deje un string crudo en el formulario. Eso romperia la validacion de edad y el armado del request.

El mensaje actual no contempla explicitamente el error `matDatepickerParse`. Debe incorporarse para informar fechas con formato o consistencia invalida.

### Catalogos desplegables

Nacionalidad, Ocupacion y Sucursal de entrega se implementan con `mat-select` y cargan sus opciones desde:

```txt
getListas/genero,vinculo,estadocivil,nacionalidad,tipodoc,ocupaciones,localidades
```

Las listas ya llegan al frontend como elementos con:

```ts
interface Item {
  id: string;
  nombre: string;
}
```

Actualmente se ordenan en el template mediante `OrderByPipe`, pero no existe filtrado ni busqueda.

El formulario guarda los IDs seleccionados y el request envia esos mismos valores en:

- `nacionalidad`
- `ocupacion`
- `puntoDeVenta`

No hay actualmente `MatAutocompleteModule` ni streams de opciones filtradas en `Step1Component`.

### Flechas de los selects

Los campos Tipo de documento, Genero, Estado civil y Vinculo usan `mat-select`. Numero de documento, en cambio, es un input de texto y no tiene flecha desplegable.

El tema Fuse define globalmente la flecha de `mat-select` con `text-secondary`, pero la card indica que el contraste sigue siendo insuficiente en determinados estados. Conviene aplicar el ajuste dentro del formulario de Adicionales para evitar alterar selects de toda la aplicacion.

## Propuesta tecnica

### 1. Convertir Fecha de nacimiento en ingreso manual

Mantener el input conectado a `matDatepicker` y al `MyDateAdapter` para conservar el parseo a `DateTime`, pero retirar del template:

- `mat-datepicker-toggle`.
- El icono de apertura.
- La posibilidad visible de abrir el calendario.

Agregar:

- Placeholder `DD/MM/AAAA`.
- `maxlength="10"`.
- `inputmode="numeric"` para facilitar el teclado numerico en mobile.
- Mensaje especifico para `matDatepickerParse`.
- Validacion de fecha requerida, fecha real, limite inferior y edad minima.

Se puede evaluar una mascara `00/00/0000` si el proyecto decide incorporarla en este componente. No debe impedir pegado, borrado ni correccion manual del valor.

Conviene revisar `MyDateAdapter` con fechas bisiestas y no bisiestas para confirmar que rechace correctamente, por ejemplo:

- `29/02/2024`: valida.
- `29/02/2023`: invalida.
- `31/04/2000`: invalida.

### 2. Implementar autocomplete para catalogos extensos

Importar `MatAutocompleteModule` en `Step1Component`.

Para Nacionalidad, Ocupacion y Sucursal de entrega:

- Reemplazar `mat-select` por input con `mat-autocomplete`.
- Filtrar por `nombre` de forma case-insensitive.
- Normalizar acentos para que una busqueda como `argentina` encuentre `Argentina` y terminos equivalentes no dependan de tildes.
- Ordenar las opciones sin mutar los arrays originales recibidos desde el servicio.
- Mostrar el nombre al usuario.
- Conservar el `id` como valor efectivo del formulario.
- Incorporar placeholders como:
  - `Buscá y seleccioná una nacionalidad`.
  - `Buscá y seleccioná una ocupación`.
  - `Buscá y seleccioná una sucursal`.

Cada control debe incluir validacion de pertenencia al catalogo. Escribir texto libre sin seleccionar una opcion no debe satisfacer `Validators.required` ni habilitar el boton Siguiente.

La implementacion puede usar una funcion `displayWith` que resuelva el nombre a partir del ID y streams filtrados construidos desde `valueChanges`.

### 3. Mantener el contrato actual con backend

El autocomplete no debe enviar nombres visibles. Los valores finales deben seguir siendo:

- ID de nacionalidad.
- ID de ocupacion.
- ID de localidad o punto de venta.

No se recomienda modificar `NewAdicionalReq`, `AdicionalesService` ni el endpoint `newAdicional`.

Antes de avanzar al paso de token, el formulario debe tener una opcion valida de cada catalogo y no un texto parcial.

### 4. Mejorar el contraste de flechas desplegables

Agregar una clase semantica al formulario o a los `mat-form-field` alcanzados y ajustar localmente:

- `.mat-mdc-select-arrow-wrapper`
- `.mat-mdc-select-arrow`

El color debe utilizar un token de texto con contraste suficiente, por ejemplo `--fuse-text-secondary` o un tono mas oscuro aprobado por diseno. Debe verificarse en estados normal, focus, error y disabled.

No conviene modificar el override global de Fuse salvo que QA confirme que el problema aplica a todos los selects de GOL.

Los autocompletes de Nacionalidad, Ocupacion y Sucursal pueden mantener un icono indicativo si diseno lo requiere, pero no deben parecer selects sin posibilidad de escritura.

### 5. Centralizar helpers de filtrado y validacion

Dentro de `Step1Component`, conviene centralizar:

- Normalizacion de texto.
- Filtrado de opciones.
- Resolucion `id -> nombre`.
- Validacion de opcion perteneciente al catalogo.

La abstraccion debe permanecer local al componente mientras solo se use en estos tres campos. No se justifica crear un componente compartido nuevo salvo que se detecten mas formularios con el mismo requerimiento.

## Criterio de implementacion sugerido

1. Conservar el `MyDateAdapter` y el valor `DateTime` del control Fecha de nacimiento.
2. Retirar toggle y calendario visibles, y agregar placeholder `DD/MM/AAAA`.
3. Incluir `matDatepickerParse` en el manejo de errores del campo.
4. Validar fechas reales, limite desde 1900 y edad minima de 14 anos.
5. Importar `MatAutocompleteModule`.
6. Preparar opciones ordenadas y streams filtrados para nacionalidad, ocupacion y localidades.
7. Implementar `displayWith` y validacion de pertenencia a cada lista.
8. Mantener los IDs seleccionados en los controles existentes.
9. Agregar placeholders descriptivos.
10. Aplicar el contraste de flechas solo a los selects alcanzados.
11. Confirmar que el boton Siguiente permanezca deshabilitado con texto libre o fecha invalida.
12. Evitar cambios en endpoint, resolver y estructura del request.

## Archivos candidatos a revisar

### Archivos probablemente involucrados

- `src/app/modules/tarjeta-de-credito/pages/adicionales/step1/step1/step1.component.html`: reemplazo de fecha y selects extensos por controles de ingreso manual/autocomplete.
- `src/app/modules/tarjeta-de-credito/pages/adicionales/step1/step1/step1.component.ts`: imports, filtrado, `displayWith`, validadores de pertenencia y manejo de opciones.
- `src/app/modules/tarjeta-de-credito/pages/adicionales/step1/step1/step1.component.scss`: contraste local de flechas y estilos de los nuevos autocompletes.
- `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.component.ts`: validacion de fecha, inicializacion del formulario y confirmacion de que el request recibe un `DateTime` y IDs validos.

### Archivos a revisar por posible impacto

- `src/app/shared/customDateAdapter/custom.date.adapter.ts`: parseo `dd/MM/yyyy`, dias validos y anos bisiestos.
- `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.service.ts`: confirmar que los catalogos mantienen la interfaz `{ id, nombre }` y que el request espera IDs.
- `src/app/modules/tarjeta-de-credito/pages/adicionales/order-by.pipe.ts`: hoy ordena mediante `sort` mutando el array original; los autocompletes deberian ordenar copias o reemplazar su uso en esos campos.
- `src/@fuse/styles/overrides/angular-material.scss`: referencia del estilo global actual de flechas, sin recomendar su modificacion directa.
- `src/styles/styles.scss`: revisar tokens disponibles si se necesita un color compartido aprobado.

### Archivos que conviene evitar tocar

- `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.service.ts`: no requiere cambios funcionales ni de endpoint.
- `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.resolver.ts`: la carga de listas ya resuelve los datos necesarios.
- `src/app/modules/tarjeta-de-credito/pages/adicionales/adicionales.routes.ts`: no hay cambios de navegacion.
- Componentes de Recargas Virtuales y otros formularios: no forman parte del alcance.
- Overrides globales de Angular Material: evitar cambios transversales hasta confirmar que el criterio de flechas es global.

## Casos a validar

- Fecha vacia:
  - Muestra el placeholder `DD/MM/AAAA`.
  - El campo permanece invalido.
- Fecha valida escrita manualmente:
  - Acepta `15/06/1990`.
  - El control conserva un `DateTime`.
  - El request envia `15-06-1990`.
- Fecha con formato invalido:
  - Rechaza `1990-06-15`.
  - Rechaza `15/6/90` si se exige estrictamente `DD/MM/AAAA`.
  - Muestra un mensaje claro de formato.
- Fecha inexistente:
  - Rechaza `31/04/2000`.
  - Rechaza `30/02/2000`.
- Ano bisiesto:
  - Acepta `29/02/2024`.
  - Rechaza `29/02/2023`.
- Limite inferior:
  - Rechaza fechas anteriores a 1900.
- Edad minima:
  - Valida exactamente el dia en que la persona cumple 14 anos.
  - Rechaza una fecha que represente menos de 14 anos.
- Nacionalidad:
  - Filtra mientras se escribe.
  - Permite buscar sin depender de mayusculas o tildes.
  - Guarda el ID al seleccionar.
  - Rechaza texto libre no seleccionado.
- Ocupacion:
  - Filtra listas extensas.
  - Guarda el ID correcto.
  - Rechaza texto parcial.
- Sucursal de entrega:
  - Filtra por nombre.
  - Guarda el ID de localidad/punto de venta.
  - Rechaza una opcion inexistente.
- Opciones sin coincidencias:
  - Muestra un estado comprensible o una lista vacia sin romper el campo.
- Edicion posterior:
  - Al volver al paso, muestra los nombres seleccionados y conserva los IDs.
  - Al editar un valor seleccionado, invalida el control hasta elegir otra opcion valida.
- Flechas:
  - Son visibles en estado normal, focus y error.
  - No se confunden con controles disabled.
- Responsive:
  - Los paneles de autocomplete son utilizables en desktop y mobile.
  - El teclado mobile permite cargar la fecha con facilidad.
- Flujo completo:
  - El boton Siguiente solo se habilita con todos los valores validos.
  - El paso de token y el envio final continúan funcionando.

## Riesgos o puntos de atencion

- Convertir Fecha de nacimiento a string puro rompería `.toFormat('dd-MM-yyyy')` y el validador de edad.
- El `MyDateAdapter` debe verificarse especialmente con anos bisiestos y fechas inexistentes.
- Una mascara puede interferir con pegado, seleccion, borrado o accesibilidad si se aplica de forma rigida.
- `Validators.required` no impide por si solo enviar texto libre en un autocomplete.
- Guardar el nombre visible en vez del ID rompería el contrato actual con backend.
- `OrderByPipe` utiliza `sort` sobre el array recibido y puede mutar las listas compartidas.
- Tres suscripciones nuevas deben cancelarse con `_unsubscribeAll` para evitar fugas.
- Los paneles de autocomplete deben manejar correctamente listas `null`, vacias o aun no cargadas.
- Un override global de la flecha puede afectar todos los `mat-select` del sistema.
- La card no define el texto exacto de placeholders ni mensajes de error.
- La captura es de baja resolucion; no permite confirmar con precision todos los estados visuales.


## Resumen final

Conviene comenzar por Fecha de nacimiento, manteniendo el `DateAdapter` y el valor `DateTime` para no alterar validaciones ni envio, pero retirando el calendario visible y agregando formato y errores claros.

Luego se deben implementar autocompletes locales para Nacionalidad, Ocupacion y Sucursal, filtrando por nombre pero conservando los IDs actuales. El contraste de flechas debe resolverse de manera acotada al formulario y validar la ambigüedad entre Tipo y Numero de documento antes de implementar.

No deberian modificarse endpoints, rutas, el resolver ni el contrato `NewAdicionalReq`. Antes de cerrar la card se debe validar fecha manual, opciones inexistentes, persistencia al volver de paso, responsive y flujo completo hasta el envio.
