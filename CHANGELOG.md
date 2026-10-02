# Registro de cambios

Aquí se documentan los cambios importantes del proyecto. Se sigue el control de versiones semántico (SemVer).

## [0.7.0] - 2026-10-02

### Simplificación
- Se reemplazaron las expresiones de los nodos **Si** por condiciones integradas de n8n: texto vacío, coincidencia de expresión regular, número distinto de 1 y existencia de tres valores numéricos.
- Se quitaron del nodo de campos la bandera calculada `nutrientsUsable` y su expresión `typeof`; los tres campos ahora se comprueban directamente en un IF con AND.
- No se agregaron nodos. Se conservaron las comparaciones de umbral en el nodo de clasificación porque representan las reglas nutricionales pedidas, no validaciones redundantes.

### Verificación
- Pendiente importar y probar en n8n. Se comprobará que las condiciones integradas se muestren correctamente en la versión instalada.

## [0.6.2] - 2026-10-02

### Corrección
- Se ajustó la extracción al formato que realmente emite el nodo HTTP en n8n: al tener desactivado **Include Response Headers and Status**, el JSON se entrega directamente en la raíz y no dentro de `body`.
- El nodo de campos ahora lee `$json.status` y `$json.product...` directamente. No se añadieron fallbacks `||` ni nodos nuevos.
- Se actualizó la nota del nodo HTTP, el README y el test log para documentar esta configuración.

### Verificación
- El workflow se validó localmente con una respuesta de ejemplo en formato raíz; falta ejecutar de nuevo el webhook de prueba en n8n.

## [0.6.1] - 2026-10-02

### Corrección
- Se convierte explícitamente el `status` de Open Food Facts a número antes de compararlo con `1`. Esto evita que un estado representado como texto desvíe un producto encontrado a la rama 404, sin añadir alternativas `||`.

### Verificación
- La consulta en vivo de Nutella devuelve HTTP 200 y `status: 1`. Falta confirmar la ruta del IF en la instancia n8n.

## [0.6.0] - 2026-10-02

### Cambios
- Se reemplazaron las rutas alternativas de extracción por las claves directas conocidas de `body` y `product`.
- Se quitó `brands` porque no interviene en las reglas ni en el veredicto mostrado; se conservan nombre, Nutri-Score, azúcar, sal, grasa y energía opcional.
- Se simplificó `nutrientsUsable` a tres comprobaciones `typeof ... === 'number'`; los valores cero siguen siendo válidos.
- Se retiraron conversiones numéricas redundantes de las comparaciones, la lectura alternativa del cuerpo HTTP y el campo `nutriScoreVerdict` que no se utilizaba.
- Se simplificaron el prompt de Groq y el formato final; solo quedan alternativas para el código ausente, Nutri-Score desconocido y energía opcional.

### Verificación
- Pendiente reimportar y ejecutar pruebas funcionales en n8n. El JSON y las referencias del flujo se validarán localmente.

## [0.5.0] - 2026-10-02

### Cambios
- Se consolidaron la normalización y extracción de la respuesta de Open Food Facts en un solo nodo, **Campos - Conservar solo datos necesarios**.
- Se desactivó el paso de campos no seleccionados para descartar el JSON grande de la API y conservar solo estado, nombre, marca, Nutri-Score, azúcar, sal, grasa, energía opcional y disponibilidad de nutrientes.
- Se eliminó el nodo duplicado **Campos - Extraer datos nutricionales** y se conectó directamente el filtro de producto encontrado con la decisión de datos nutricionales.
- No se agregaron nodos; se simplificó la ruta y se actualizó la documentación y el registro de regresión.

### Verificación
- Pendiente validar importación y regresión funcional en n8n. Se comprobará localmente la estructura del grafo y el descarte de campos extra.

## [0.4.0] - 2026-10-02

### Cambios
- Se tradujeron al español los nombres y las notas de los nodos, los mensajes del webhook, el prompt de Groq, el veredicto y el diagrama.
- Se tradujeron al español el README, el registro de pruebas y el changelog.
- Se conservaron en inglés los nombres técnicos que deben coincidir con la API y con la solicitud, como `barcode`, `sugars_100g` y `nutriscore_grade`.

### Verificación
- Pendiente validar el workflow traducido y ejecutar pruebas de regresión en n8n.

## [0.3.0] - 2026-10-02

### Cambios
- Se eliminó el nodo de combinación final de veredicto/IA para que la ruta de éxito sea lineal: indicador de preocupación → párrafo de Groq → formateador → respuesta del webhook.
- El formateador lee los datos deterministas del nodo Código mediante la vinculación de elementos de n8n y recibe el párrafo de IA directamente del nodo anterior.
- Se actualizaron el README, el diagrama de Excalidraw y las pruebas de regresión para reflejar el flujo sin nodos de combinación.

### Verificación
- El JSON y la estructura del grafo se validaron localmente. La ruta lineal revisada aún requiere pruebas de regresión en la instancia de n8n de la persona usuaria.

## [0.2.0] - 2026-10-02

### Cambios
- Se simplificó la salida Verdadero de la validación numérica: ahora tiene un solo cable hacia la consulta de Open Food Facts.
- Se eliminó el nodo de combinación entre la solicitud y la API; la respuesta de datos insuficientes recupera el código desde el nodo de normalización vinculado.
- Se conservó el nodo de combinación final para unir los datos deterministas del veredicto con el texto de Groq.
- Se actualizaron el README, el diagrama y el registro de pruebas de regresión.

### Verificación
- El JSON y la estructura del grafo se validaron localmente. La exportación revisada aún requiere pruebas de regresión en la instancia de n8n de la persona usuaria.

## [0.1.0] - 2026-10-01

### Añadido
- Exportación inicial del workflow de n8n para evaluar productos SnackCheck mediante su código de barras.
- Diagrama de Excalidraw, documentación de configuración y uso, y registro de pruebas TC-XXX.
- Ramas específicas para documentación, validación del código, producto no encontrado y datos nutricionales insuficientes.
- Combinación del semáforo y Nutri-Score, indicador de preocupación, instrucción de Groq según el nivel y respuesta formateada en texto sin formato.

### Notas
- Esta es una plantilla preliminar. Aún no se ha importado, configurado con credenciales, activado ni probado de extremo a extremo en la instancia de n8n de la persona usuaria.
