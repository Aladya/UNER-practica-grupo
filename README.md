📝Apuntes para aplicacion de Gestión de Gastos

Aquí tienes un resumen conciso de los puntos más importantes a practicar para tu examen oral, centrándote en las funciones de registrar, modificar y eliminar gastos. ¡Llévalos como tus notas!

Conceptos Generales de Python (importantes para cualquier función):

.strip():

Propósito: Elimina espacios en blanco al principio y al final de una cadena de texto.

Uso: variable =input(...).strip() para limpiar la entrada del usuario.

try-except bloques:

Propósito: Manejar errores (excepciones) de forma controlada.

Funcionamiento:

try:: Intenta ejecutar el código.

except:: Si ocurre un error dentro del try, el código salta aquí para manejarlo (ej., imprimir un mensaje de error).

while True con break:

Propósito: Crear un bucle que se repita indefinidamente hasta que una condición específica se cumpla y se ejecute break.

Uso: Ideal para validar entradas del usuario (ej., pedir un número hasta que se ingrese uno válido).

f-strings con especificadores de formato (ej., {variable:<25}, {monto:.2f}):

Propósito: Crear cadenas de texto formateadas y alineadas.

::Indica que viene un formato.

<:Alineación a la izquierda.

>:Alineación a la derecha.

^:Centrado.

25: Ancho total del campo en caracteres.

.2f: Formato de número flotante con 2 decimales.

.get("clave", valor_por_defecto):

Propósito: Acceder a un valor en un diccionario de forma segura.

Uso: Si la "clave" no existe, devuelve valor_por_defecto en lugar de un error (KeyError).

if variable_de_cadena: (Truthiness):

Propósito: Evaluar si una cadena de texto NO está vacía.

Funcionamiento:

Si la variable_de_cadena contiene texto (no es ""),la condición es True.

Si la variable_de_cadena está vacía (""),la condición es False.

Uso: Muy común para permitir que el usuario deje un campo en blanco para no modificarlo.

Función: registrar_gasto(datos) (Opción 2)

Propósito Principal: Permite al usuario añadir un nuevo gasto a la aplicación.

Flujo Clave:

Descripción: Pide la descripción (input().strip()).

Monto (¡Validación Crítica!):

Usa un while True con try-except para asegurar que el usuario ingrese un número.

try: Intenta float(input()).

break si es exitoso.

except: Si hay error (ValueError), imprime "Monto inválido" y el bucle vuelve a pedir el monto.

Fecha: Pide la fecha (input().strip()).

Categoría (elegir_categoria()):

Llama a la sub-función elegir_categoria().

Esta muestra las CATEGORIAS_DISPONIBLES numeradas (enumerate(..., start=1)).

Captura la elección del usuario (op), la convierte a entero y la ajusta al índice de lista (idx =int(op) -1).

¡Validación!: El try-except maneja si el op no es un número. El if 0 <=idx <len(...) valida si el índice está en el rango correcto.

Devuelve el nombre de la categoría (ej. "Comida"), no el número/índice. Si es inválido, devuelve "Otros".

Generación de ID (nuevo_id):

max([g.get("id", 0) for g in datos], default=0) +1

Explicar: Busca el ID más alto existente (max), usa 0 si un gasto no tiene ID o la lista está vacía (default=0), y luego suma 1 para asegurar que sea un ID único y consecutivo. Es automático, el usuario no lo ingresa.

Almacenamiento: Crea un diccionario con todos los datos del gasto y lo append() a la lista datos.

Confirmación: print("Gasto registrado.").

Función: modificar_gasto(datos) (Opción 3)

Propósito Principal: Permite al usuario actualizar los detalles de un gasto existente.

Flujo Clave:

Verificación Inicial: if not datos: para chequear si hay gastos para modificar.

Pedir ID a Modificar:

Pide el id_buscar (input().strip()).

try-except: Para validar que el ID ingresado sea un número entero (ValueError). Si no lo es, print("ID inválido.") y return.

Buscar el Gasto:

for g in datos:: Recorre la lista de gastos.

if g.get("id") ==id_buscar:: Cuando encuentra el gasto por su ID, entra a modificarlo.

Modificar Campos (¡Clave!):

Para Descripción, Categoría, Fecha:

Muestra el valor (actual: ...)y pide un nuevo valor (input().strip()).

if nueva_desc: (o nueva_cat, nf): Esta es la clave. Si el usuario escribió algo (la cadena NO está vacía), entonces se actualiza el valor en el diccionario g["campo"] =nuevo_valor.

Si el usuario presiona Enter (cadena vacía), la condición if es False, y el valor del campo se mantiene como estaba.

Para Monto (¡Doble Validación!):

Usa un while True con try-except anidado para asegurar un número válido.

if not nm: break: ¡Importante! Si el usuario presiona Enter (cadena nm vacía), not nm es True, y el break hace que se mantenga el monto actual sin cambios.

Si escribe algo, el try intenta float(nm). Si es válido, se actualiza y break. Si no, except avisa y pide de nuevo.

Confirmación y Salida: print("Gasto modificado.") y return. El return es crucial: una vez modificado el gasto, la función termina para no seguir recorriendo la lista.

Gasto No Encontrado: Si el bucle for termina y el gasto con ese ID no se encontró, se imprime el mensaje print("No se encontró un gasto con ese ID.").

Función: eliminar_gasto(datos) (Opción 4)

Propósito Principal: Permite al usuario remover un gasto específico de la lista.

Flujo Clave:

Verificación Inicial: if not datos: para chequear si hay gastos para eliminar.

Pedir ID a Eliminar:

Pide el id_buscar (input().strip()).

try-except: Para validar que el ID ingresado sea un número entero (ValueError). Si no lo es, print("ID inválido.") y return.

Buscar el Gasto (¡con Índice!):

for i, g in enumerate(datos):: ¡Clave! Usamos enumerate para obtener tanto el índice (i) como el gasto (g). Esto es necesario para eliminar el elemento por su índice.

if g.get("id") ==id_buscar:: Cuando encuentra el gasto por su ID, entra a procesarlo.

Confirmación (¡Doble Seguridad!):

if input(f"Querés eliminar '{g.get('descripcion')}'? (s/n): ").lower() =='s':

Explicar: Pide confirmación al usuario, mostrando la descripción del gasto. Convierte la respuesta a minúsculas (.lower()) y solo si es exactamente 's' se procede.

Eliminación o Cancelación:

Si confirma ('s'): datos.pop(i) elimina el gasto de la lista usando su índice i. print("Gasto eliminado.").

Si no confirma: print("Operación cancelada.").

Salida: En ambos casos (eliminación o cancelación), return sale de la función inmediatamente, ya que el trabajo para ese ID ha terminado.

Gasto No Encontrado: Si el bucle for termina y el gasto con ese ID no se encontró, se imprime el mensaje print("No se encontró un gasto con ese ID.").
