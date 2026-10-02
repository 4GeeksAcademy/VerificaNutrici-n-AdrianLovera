# SnackCheck: revisión nutricional

## Propósito

Este flujo de n8n recibe un código de barras, consulta Open Food Facts y aplica las reglas indicadas para azúcar, sal, grasa y Nutri-Score. Devuelve un veredicto breve y fácil de entender. Open Food Facts no requiere clave; Groq es la única credencial necesaria.

## Cómo funciona

1. El webhook recibe `POST /nutrition-check` con un cuerpo como `{ "barcode": "3017620422003" }`.
2. Dos nodos **Si** usan condiciones integradas de n8n: el primero comprueba si el texto está vacío; el segundo usa **coincide con expresión regular** con `^[0-9]+$` para aceptar solo dígitos. El primero devuelve documentación si falta el código; el segundo envía códigos mal formados a HTTP 400.
3. La salida **Verdadero** de la validación numérica tiene un solo cable: va directamente a la consulta v2 de Open Food Facts. El nodo HTTP deja desactivada la opción **Include Response Headers and Status**, por lo que entrega el JSON de la API directamente en la raíz (sin envoltorio `body`); **Never Error** permite procesar el cuerpo 404. El nodo **Campos - Conservar solo datos necesarios** convierte `status` a número y lee directamente `status`, `product` y sus claves. Conserva estado, nombre, Nutri-Score, azúcar, sal, grasa, energía opcional y la bandera de datos válidos.
4. El nodo **Si - Producto no encontrado** usa la condición numérica **no es igual a 1**. Si no se encontró el producto, devuelve JSON con HTTP 404. Otro nodo **Si** exige que azúcar, sal y grasa **existan** mediante tres comprobaciones numéricas integradas unidas con AND; el cero existe y es válido. Si falta cualquiera, responde con JSON HTTP 200 indicando datos insuficientes.
5. En **Campos - Clasificar semáforo nutricional**, las comparaciones `<` y `>` implementan los umbrales solicitados, no son validaciones extra: azúcar bajo <5 g/100 g, medio entre 5 y 22,5 inclusive, alto >22,5; sal alta >1,5; grasa alta >17,5. Cero nutrientes altos es saludable, uno moderado y dos o tres no saludable. Nutri-Score a/b equivale a saludable, c a moderado y d/e a no saludable. Se elige el peor resultado; si la calificación falta o no se reconoce, se usa solo el semáforo.
6. El nodo Código calcula en un solo elemento el indicador de preocupación (0–3). Groq redacta únicamente el párrafo explicativo, con el tono apropiado al veredicto. El flujo de éxito es lineal: **Código → IA → Formateador → Respuesta**. El formateador lee los datos deterministas del nodo Código mediante la vinculación de elementos de n8n; no hace falta ningún nodo de combinación.

Consulta el [diagrama SnackCheck](SnackCheck.excalidraw) y el [flujo de trabajo de n8n](SnackCheck.workflow.json).

## Configuración

1. Abre el diagrama de Excalidraw y revisa el flujo antes de importarlo.
2. En n8n, importa `SnackCheck.workflow.json` como flujo de trabajo.
3. Abre **IA - Modelo de chat Groq** y selecciona la credencial de Groq que ya tienes. Las credenciales no se incluyen en el archivo. Si `llama-3.3-70b-versatile` no está disponible, elige un modelo de chat que sí lo esté.
4. Comprueba que tu versión de n8n admita las versiones de nodos incluidas y las opciones **Include Response Headers and Status** y **Never Error** del nodo HTTP Request. Configúralas —o usa las opciones equivalentes— para que la respuesta 404 llegue a la rama de producto no encontrado.
5. Guarda y ejecuta las pruebas de [TEST-LOG.md](TEST-LOG.md). Revisa los datos de ejecución y el estado y `Content-Type` de cada respuesta. Después de **Campos - Conservar solo datos necesarios**, el objeto debe contener únicamente los campos seleccionados, sin encabezados ni el JSON completo de Open Food Facts. Antes de activar el webhook de producción, comprueba también que el formateador pueda leer los datos del nodo Código después de recibir la respuesta de IA.
6. Cuando pasen las pruebas, activa el flujo de trabajo y usa su **URL de producción**, no la URL de prueba.

## Uso

Solicitud: `POST <url-de-producción-de-n8n>/webhook/nutrition-check`

Encabezado: `Content-Type: application/json`

Cuerpo: `{ "barcode": "3017620422003" }`

Envía el código como texto para conservar los ceros iniciales. Un cuerpo vacío o un código ausente devuelve la documentación del servicio. Para un producto válido, la respuesta es texto sin formato e incluye azúcar, sal, grasa, Nutri-Score y el indicador de preocupación por cada 100 g. La energía se muestra si está disponible, pero no afecta la evaluación.

Códigos de barras útiles para probar datos reales: Nutella `3017620422003` (calificación e), Coca-Cola `5449000000996` (calificación e aunque no tiene nutrientes altos en el semáforo) y copos de avena Bjorg `3229820019307` (calificación a y ningún nutriente alto). El código numérico `0000000000000` sirve para probar la respuesta de producto no encontrado.

## Manejo de errores

| Situación | Estado HTTP | Tipo de contenido | Respuesta |
|---|---:|---|---|
| Cuerpo vacío o código ausente/vacío | 200 | `application/json` | Instrucciones de uso |
| Código con caracteres no numéricos | 400 | `application/json` | Error `codigo_de_barras_invalido` y ejemplo |
| Open Food Facts no encuentra el producto | 404 | `application/json` | Error `producto_no_encontrado` y ejemplo válido |
| Falta azúcar, sal o grasa, o su valor no es numérico | 200 | `application/json` | Error `datos_nutricionales_insuficientes` |
| Producto encontrado con datos requeridos | 200 | `text/plain; charset=utf-8` | Veredicto formateado |

La disponibilidad de Open Food Facts o Groq, los errores de red, los límites de uso y los tiempos de espera pueden impedir una respuesta correcta. La consulta HTTP tiene un límite de 15 segundos y deja pasar respuestas HTTP de error; confirma el comportamiento en la versión de n8n instalada antes de usar el flujo en producción.

## Limitaciones

- La comunidad mantiene los datos de Open Food Facts, que pueden ser incompletos o imprecisos. Esta herramienta orienta sobre productos; no es consejo médico ni dietético.
- Las reglas se aplican por cada 100 g. Algunos productos muestran datos por 100 ml; el flujo no convierte unidades.
- Solo azúcar, sal, grasa y Nutri-Score determinan el veredicto. La energía es informativa.
- Si falta Nutri-Score, se usa el resultado del semáforo. Si falta cualquiera de los tres nutrientes requeridos o no tiene un valor numérico, no se clasifica.
- Groq puede añadir latencia y variar la redacción, pero no determina ni modifica el veredicto.
- El archivo JSON es una exportación portable. Desde este espacio de trabajo no se puede activar el flujo en la instancia externa de n8n ni adjuntar la credencial privada de Groq. La importación, selección de credencial, prueba y activación se completan en esa instancia.
