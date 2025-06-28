import json # Importa el módulo para trabajar con archivos JSON (guardar y cargar datos estructurados).
import os # Importa el módulo para interactuar con el sistema operativo (ej., verificar si un archivo existe).
from datetime import datetime, date # Importa las clases datetime (para fecha y hora) y date (solo para fecha).
from collections import defaultdict # Importa defaultdict, una subclase de dict útil para valores por defecto.

# Importaciones específicas para la generación de PDF con ReportLab
from reportlab.lib.pagesizes import letter # Importa el tamaño de página 'letter' (carta) para los PDFs.
from reportlab.pdfgen import canvas # Importa la clase 'canvas' para crear y dibujar en archivos PDF.

CATEGORIAS_DISPONIBLES = [ # Define una lista (constante) de categorías de gastos predefinidas.
    "Comida",     # Categoría para gastos de alimentación.
    "Transporte", # Categoría para gastos de movilidad.
    "Ocio",       # Categoría para gastos de entretenimiento.
    "Salud",      # Categoría para gastos médicos o de bienestar.
    "Hogar",      # Categoría para gastos relacionados con el hogar.
    "Otros"       # Categoría genérica para gastos que no encajan en las anteriores.
]

# --- Sección de Carga y Guardado de Datos ---

def cargar_datos(): # Define la función para cargar los gastos desde un archivo JSON.
    if not os.path.exists("gastos.json"): # Verifica si el archivo 'gastos.json' NO existe en el directorio actual.
        return [] # Si el archivo no existe, devuelve una lista vacía (no hay gastos previos).
    with open("gastos.json", "r", encoding="utf-8") as archivo: # Abre el archivo 'gastos.json' en modo lectura ('r') con codificación UTF-8.
        data = json.load(archivo) # Carga (lee) los datos JSON del archivo y los guarda en la variable 'data'.
    if isinstance(data, dict): # Comprueba si los datos cargados son un diccionario (esto es para compatibilidad con versiones anteriores de cómo se guardaban los datos).
        try: # Intenta convertir los valores del diccionario a una lista.
            return list(data.values()) # Si 'data' es un diccionario, devuelve una lista con sus valores (que son los gastos).
        except Exception: # Si ocurre algún error durante la conversión (por si el formato del diccionario es inesperado).
            return [] # Devuelve una lista vacía en caso de error para evitar fallos.
    return data # Si los datos ya son una lista (el formato esperado), la devuelve directamente.

def guardar_datos(datos): # Define la función para guardar la lista de gastos en un archivo JSON.
    with open("gastos.json", "w", encoding="utf-8") as archivo: # Abre el archivo 'gastos.json' en modo escritura ('w') con codificación UTF-8.
        # 'w' sobrescribe el archivo si existe, o lo crea si no existe.
        json.dump(datos, archivo, indent=4, ensure_ascii=False) # Escribe (guarda) la lista 'datos' en el archivo como JSON.
        # 'indent=4' hace que el JSON sea más legible con indentación de 4 espacios.
        # 'ensure_ascii=False' permite que caracteres no ASCII (como 'ñ', tildes) se guarden directamente sin ser escapados.

# --- Sección de Visualización de Gastos ---

def ver_gastos(datos): # Define la función para mostrar todos los gastos registrados en la consola.
    if not datos: # Comprueba si la lista 'datos' (de gastos) está vacía (condición "truthy").
        print("No hay gastos registrados.") # Si está vacía, informa al usuario.
        return # Sale de la función, ya que no hay nada que mostrar.
    # Imprime la cabecera de la tabla con formato fijo para cada columna.
    # '<4', '<25', etc., especifican alineación a la izquierda y el ancho total del campo.
    print(f"{'ID':<4} {'Descripción':<25} {'Categoría':<12} {'Monto':<10} {'Fecha':<12}")
    total = 0.0 # Inicializa una variable para acumular el total de los montos.
    for idx, g in enumerate(datos, start=1): # Itera sobre cada gasto 'g' en la lista 'datos', obteniendo su índice 'idx' (que empieza en 1).
        id_val = g.get("id", idx) # Obtiene el 'id' del gasto; si la clave 'id' no existe, usa 'idx' como ID por defecto.
        desc = g.get("descripcion", "")[:25] # Obtiene la descripción (o cadena vacía), limitándola a los primeros 25 caracteres.
        cat = g.get("categoria", "") # Obtiene la categoría (o cadena vacía).
        try: # Intenta convertir el monto a un número flotante (decimal).
            monto = float(g.get("monto", 0)) # Obtiene el monto (o 0 si no existe) y lo convierte a flotante.
        except: # Si ocurre un error durante la conversión (ej., el monto no es un número válido).
            monto = 0.0 # Asigna 0.0 al monto para evitar que el programa falle.
        fecha = g.get("fecha", "") # Obtiene la fecha (o cadena vacía).
        total += monto # Suma el monto actual al total acumulado.
        # Imprime los detalles del gasto actual, formateados para alinearse con la cabecera.
        print(f"{id_val:<4} {desc:<25} {cat:<12} {monto:<10.2f} {fecha:<12}")
    print("-" * 70) # Imprime una línea separadora para el total.
    fecha_hoy = date.today().strftime('%d/%m/%Y') # Obtiene la fecha actual y la formatea como DD/MM/YYYY.
    # Imprime la fila del total, alineada con las columnas de la tabla.
    print(f"{'':<4} {'TOTAL':<25} {'':<12} {total:<10.2f} {fecha_hoy:<12}")

# --- Sección de Registro de Nuevo Gasto ---

def elegir_categoria(): # Define una función auxiliar para que el usuario seleccione una categoría.
    print("Categorías disponibles:") # Imprime un encabezado.
    for i, c in enumerate(CATEGORIAS_DISPONIBLES, start=1): # Itera sobre la lista de categorías, numerándolas desde 1.
        print(f"{i}. {c}") # Imprime el número y el nombre de cada categoría.
    op = input("Elegí categoría (número): ").strip() # Pide al usuario que ingrese el número de la categoría y elimina espacios.
    try: # Intenta convertir la entrada del usuario a un número entero.
        idx = int(op) - 1 # Convierte la opción a un índice basado en 0 (porque las listas son base 0).
        if 0 <= idx < len(CATEGORIAS_DISPONIBLES): # Verifica si el índice calculado es válido y está dentro del rango de la lista.
            return CATEGORIAS_DISPONIBLES[idx] # Si es válido, devuelve el nombre de la categoría seleccionada.
    except: # Si ocurre un error (ej., el usuario no ingresó un número) o el índice está fuera de rango.
        pass # No hace nada, simplemente salta al siguiente paso.
    print("Categoría no válida, se asigna 'Otros'.") # Informa al usuario que la categoría no es válida.
    return "Otros" # Devuelve "Otros" como categoría por defecto.

def registrar_gasto(datos): # Define la función principal para registrar un nuevo gasto.
    descripcion = input("Descripción: ").strip() # Pide la descripción del gasto y elimina espacios.
    while True: # Inicia un bucle infinito para validar la entrada del monto.
        try: # Intenta convertir el monto a un número flotante.
            monto = float(input("Monto (número): ").strip()) # Pide el monto, lo convierte a flotante y elimina espacios.
            break # Si la conversión es exitosa, sale del bucle de validación.
        except: # Si ocurre un error (ej., el usuario no ingresó un número).
            print("Monto inválido. Ingresá un número.") # Informa al usuario y el bucle se repite.
    fecha = input("Fecha (DD-MM-YYYY): ").strip() # Pide la fecha del gasto en formato DD-MM-YYYY y elimina espacios.
    categoria = elegir_categoria() # Llama a la función auxiliar para que el usuario elija una categoría.
    # Calcula un nuevo ID único:
    # 1. Crea una lista de todos los IDs existentes (usando 0 si un gasto no tiene ID).
    # 2. Encuentra el ID máximo de esa lista (o 0 si la lista de datos está vacía, gracias a 'default=0').
    # 3. Suma 1 para obtener el siguiente ID disponible.
    nuevo_id = max([g.get("id", 0) for g in datos], default=0) + 1
    datos.append({ # Agrega un nuevo diccionario (representando el gasto) a la lista 'datos'.
        "id": nuevo_id,           # Asigna el ID único generado.
        "descripcion": descripcion, # Asigna la descripción ingresada por el usuario.
        "categoria": categoria,   # Asigna la categoría seleccionada.
        "monto": monto,           # Asigna el monto validado.
        "fecha": fecha            # Asigna la fecha ingresada.
    })
    print("Gasto registrado.") # Informa al usuario que el gasto ha sido registrado.

# --- Sección de Modificación de Gasto Existente ---

def modificar_gasto(datos): # Define la función para modificar un gasto existente.
    if not datos: # Verifica si la lista 'datos' está vacía.
        print("No hay gastos para modificar.") # Si no hay gastos, informa al usuario.
        return # Sale de la función.
    try: # Intenta convertir la entrada del ID a un número entero.
        id_buscar = int(input("ID del gasto a modificar: ").strip()) # Pide el ID del gasto a modificar y lo limpia/convierte.
    except: # Si ocurre un error (ej., el ID no es un número).
        print("ID inválido.") # Informa al usuario.
        return # Sale de la función.
    
    gasto_encontrado = False # Bandera para saber si se encontró el gasto.
    for g in datos: # Itera sobre cada diccionario de gasto 'g' en la lista 'datos'.
        if g.get("id") == id_buscar: # Comprueba si el 'id' del gasto actual coincide con el 'id_buscar'.
            gasto_encontrado = True # Marca la bandera como verdadera.

            # Modificar Descripción:
            # Muestra la descripción actual y pide una nueva. Si el usuario deja vacío, no cambia.
            nueva_desc = input(f"Nueva descripción (actual: {g.get('descripcion')}): ").strip()
            if nueva_desc: # Si la nueva descripción NO está vacía (condición "truthy").
                g["descripcion"] = nueva_desc # Actualiza la descripción del gasto.

            # Modificar Categoría:
            # Muestra la categoría actual y pide una nueva. Si el usuario deja vacío, no cambia.
            nueva_cat = input(f"Nueva categoría (actual: {g.get('categoria')}), Enter para mantener: ").strip()
            if nueva_cat: # Si la nueva categoría NO está vacía.
                g["categoria"] = nueva_cat # Actualiza la categoría del gasto.

            # Modificar Monto (con validación y opción de no cambiar):
            while True: # Inicia un bucle para validar la entrada del monto.
                nm = input(f"Nuevo monto (actual: {g.get('monto')}): ").strip() # Pide el nuevo monto, mostrando el actual.
                if not nm: # Si el usuario no ingresa nada (la cadena 'nm' está vacía), es decir, no quiere cambiar el monto.
                    break # Sale del bucle, manteniendo el monto actual.
                try: # Intenta convertir el nuevo monto a flotante.
                    g["monto"] = float(nm) # Actualiza el monto del gasto.
                    break # Si la conversión es exitosa, sale del bucle.
                except: # Si la conversión falla (ej., no es un número).
                    print("Monto inválido. Ingresá un número.") # Informa al usuario y el bucle se repite.

            # Modificar Fecha:
            # Muestra la fecha actual y pide una nueva. Si el usuario deja vacío, no cambia.
            nf = input(f"Nueva fecha (actual: {g.get('fecha')}): ").strip()
            if nf: # Si la nueva fecha NO está vacía.
                g["fecha"] = nf # Actualiza la fecha del gasto.

            print("Gasto modificado.") # Informa al usuario que el gasto ha sido modificado.
            return # Sale de la función, ya que el gasto fue encontrado y modificado.
            
    if not gasto_encontrado: # Si el bucle 'for' terminó y la bandera 'gasto_encontrado' es False (no se encontró el ID).
        print("No se encontró un gasto con ese ID.") # Informa al usuario que el ID no fue encontrado.

# --- Sección de Eliminación de Gasto ---

def eliminar_gasto(datos): # Define la función para eliminar un gasto específico.
    if not datos: # Verifica si la lista 'datos' está vacía.
        print("No hay gastos para eliminar.") # Si no hay gastos, informa al usuario.
        return # Sale de la función.
    try: # Intenta convertir la entrada del ID a un número entero.
        id_buscar = int(input("ID del gasto a eliminar: ").strip()) # Pide el ID del gasto a eliminar y lo limpia/convierte.
    except: # Si ocurre un error (ej., el ID no es un número).
        print("ID inválido.") # Informa al usuario.
        return # Sale de la función.
    
    gasto_eliminado = False # Bandera para saber si se encontró y eliminó el gasto.
    for i, g in enumerate(datos): # Itera sobre la lista 'datos', obteniendo tanto el índice 'i' como el gasto 'g'.
        # El índice 'i' es crucial porque 'pop()' necesita la posición del elemento a eliminar.
        if g.get("id") == id_buscar: # Comprueba si el 'id' del gasto actual coincide con el 'id_buscar'.
            # Pide confirmación al usuario antes de eliminar (medida de seguridad).
            # .lower() convierte la respuesta a minúsculas para una comparación flexible ('S' o 's').
            if input(f"Querés eliminar '{g.get('descripcion')}'? (s/n): ").lower() == 's':
                datos.pop(i) # Elimina el gasto de la lista usando su índice 'i'.
                print("Gasto eliminado.") # Informa al usuario.
                gasto_eliminado = True # Marca la bandera como verdadera.
            else: # Si el usuario no confirma (no escribe 's').
                print("Operación cancelada.") # Informa que la operación fue cancelada.
            return # Sale de la función, ya que el gasto fue procesado (eliminado o cancelación).
    
    if not gasto_eliminado: # Si el bucle 'for' terminó y la bandera 'gasto_eliminado' es False (no se encontró el ID).
        print("No se encontró un gasto con ese ID.") # Informa al usuario que el ID no fue encontrado.

# --- Sección de Estadísticas ---

def calcular_estadisticas(datos): # Define la función para calcular varias estadísticas sobre los gastos.
    montos = [] # Lista para almacenar solo los montos de los gastos.
    for g in datos: # Itera sobre cada gasto.
        try: # Intenta convertir el monto a flotante.
            montos.append(float(g.get("monto", 0))) # Agrega el monto a la lista 'montos'.
        except: # Si el monto no es válido.
            pass # Ignora el gasto y continúa.
    
    total = sum(montos) # Calcula la suma total de todos los montos.
    # Calcula el promedio; usa 0.0 si no hay montos para evitar división por cero.
    promedio = total / len(montos) if montos else 0.0
    mayor = max(montos) if montos else 0.0 # Encuentra el monto máximo; 0.0 si no hay montos.
    menor = min(montos) if montos else 0.0 # Encuentra el monto mínimo; 0.0 si no hay montos.
    
    suma_cat = defaultdict(float) # Crea un defaultdict que asigna 0.0 por defecto a nuevas claves (categorías).
    for g in datos: # Itera sobre cada gasto.
        # Suma el monto del gasto a su categoría correspondiente.
        # Si la categoría no existe en 'suma_cat', defaultdict la crea con 0.0 antes de sumar.
        suma_cat[g.get("categoria", "Otros")] += float(g.get("monto", 0))
    
    # Calcula el porcentaje de cada categoría sobre el total de gastos.
    porc_cat = {cat: (m / total * 100 if total else 0.0) for cat, m in suma_cat.items()}
    # Encuentra la categoría con el mayor gasto, usando sus valores para la comparación.
    cat_may = max(suma_cat, key=suma_cat.get) if suma_cat else None
    
    # Devuelve un diccionario con todas las estadísticas calculadas.
    return {"total": total, "promedio": promedio, "mayor": mayor, "menor": menor, "por_categoria": porc_cat, "categoria_mayor": cat_may}

def ver_estadisticas(datos): # Define la función para mostrar las estadísticas en la consola.
    stats = calcular_estadisticas(datos) # Llama a la función para obtener las estadísticas.
    print(f"Total gastado: {stats['total']:.2f}") # Imprime el total gastado, formateado a 2 decimales.
    print(f"Promedio:        {stats['promedio']:.2f}") # Imprime el promedio.
    print(f"Gasto máximo:    {stats['mayor']:.2f}") # Imprime el gasto máximo.
    print(f"Gasto mínimo:    {stats['menor']:.2f}\n") # Imprime el gasto mínimo.
    
    print("--- Por categoría ---") # Encabezado para la sección por categoría.
    print(f"{'Categoría':<12} {'% Gasto':>8}") # Cabecera de la tabla de categorías (alineación a la derecha para porcentaje).
    for cat, pct in stats['por_categoria'].items(): # Itera sobre cada categoría y su porcentaje.
        print(f"{cat:<12} {pct:8.2f}%") # Imprime la categoría y su porcentaje formateado.
    
    if stats.get('categoria_mayor'): # Si existe una categoría con mayor gasto.
        print(f"\nCategoría con mayor gasto: {stats['categoria_mayor']}" ) # Imprime el nombre de la categoría con mayor gasto.

# --- Sección de Eliminación Total de Gastos ---

def eliminar_todos(datos): # Define la función para eliminar todos los gastos.
    # Pide confirmación al usuario, convierte la respuesta a minúsculas y elimina espacios.
    confirm = input("¿Estás seguro que querés eliminar todos los gastos? (s/n): ").strip().lower()
    if confirm == 's': # Si el usuario confirma con 's'.
        datos.clear() # Vacía la lista 'datos', eliminando todos los gastos.
        print("Todos los gastos han sido eliminados.") # Informa al usuario.
    else: # Si el usuario no confirma.
        print("Operación cancelada.") # Informa que la operación fue cancelada.

# --- Sección de Exportación de Datos (TXT y PDF) ---

def exportar_a_txt(datos): # Define la función para exportar los gastos a un archivo de texto (JSON).
    ahora = datetime.now() # Obtiene la fecha y hora actual.
    fecha_display = ahora.strftime("%d/%m/%Y") # Formatea la fecha para mostrarla en el archivo.
    fecha_file = ahora.strftime("%d-%m-%Y_%H-%M-%S") # Formatea la fecha para usarla en el nombre del archivo.
    nombre_archivo = f"gastos_exportados_{fecha_file}.txt" # Crea el nombre del archivo TXT.

    stats = calcular_estadisticas(datos) # Calcula las estadísticas de los gastos.
    contenido = { # Crea un diccionario que contendrá los datos y estadísticas a exportar.
        "titulo": "Listado de gastos", # Título del reporte.
        "estadisticas": { # Sub-sección para las estadísticas.
            "total": round(stats["total"], 2), # Total gastado, redondeado a 2 decimales.
            "promedio": round(stats["promedio"], 2), # Promedio.
            "mayor": round(stats["mayor"], 2), # Gasto máximo.
            "menor": round(stats["menor"], 2), # Gasto mínimo.
            "por_categoria": {cat: round(pct, 2) for cat, pct in stats["por_categoria"].items()}, # Porcentajes por categoría.
            "categoria_con_mayor_gasto": stats.get('categoria_mayor'), # Categoría con mayor gasto.
            "fecha_exportacion": fecha_display # Fecha en que se generó la exportación.
        },
        "datos": datos # Incluye la lista completa de datos de gastos.
    }
    with open(nombre_archivo, "w", encoding="utf-8") as archivo: # Abre el archivo TXT en modo escritura con UTF-8.
        archivo.write(json.dumps(contenido, indent=4, ensure_ascii=False)) # Escribe el diccionario 'contenido' como JSON formateado.
    print(f"Gastos exportados a {nombre_archivo}") # Informa al usuario sobre el archivo creado.

def exportar_a_pdf(datos): # Define la función para exportar los gastos a un archivo PDF.
    ahora = datetime.now() # Obtiene la fecha y hora actual.
    fecha_display = ahora.strftime("%d/%m/%Y") # Formatea la fecha para mostrarla en el PDF.
    fecha_file = ahora.strftime("%d-%m-%Y_%H-%M-%S") # Formatea la fecha para el nombre del archivo.
    nombre_archivo = f"gastos_exportados_{fecha_file}.pdf" # Crea el nombre del archivo PDF.

    stats = calcular_estadisticas(datos) # Calcula las estadísticas.
    total = stats["total"] # Obtiene el total de gastos de las estadísticas.

    c = canvas.Canvas(nombre_archivo, pagesize=letter) # Crea un objeto 'canvas' para dibujar en el PDF (tamaño carta).
    width, height = letter # Desempaqueta las dimensiones (ancho y alto) del tamaño 'letter' en puntos.

    c.setFont("Helvetica-Bold", 16) # Establece la fuente y tamaño para el título.
    c.drawString(50, height - 50, f"Listado de gastos") # Dibuja el título del reporte en la posición (X, Y).
    # (50 es el margen X, height - 50 es la posición Y desde arriba).

    c.setFont("Helvetica-Bold", 12) # Establece la fuente para las cabeceras de la tabla.
    y = height - 80 # Define la posición Y inicial para las cabeceras de la tabla (más abajo que el título).
    c.drawString(50, y, "ID") # Dibuja la cabecera "ID".
    c.drawString(100, y, "Descripción") # Dibuja la cabecera "Descripción".
    c.drawString(300, y, "Monto") # Dibuja la cabecera "Monto".
    c.drawString(400, y, "Fecha") # Dibuja la cabecera "Fecha".
    y -= 20 # Disminuye Y para la siguiente fila (mover hacia abajo).

    c.setFont("Helvetica", 10) # Establece la fuente para los datos de los gastos.
    for idx, g in enumerate(datos, start=1): # Itera sobre cada gasto para dibujarlo en el PDF.
        if y < 100: # Comprueba si la posición Y actual está muy cerca del final de la página.
            c.showPage() # Si es así, crea una nueva página.
            c.setFont("Helvetica-Bold", 12) # Restablece la fuente para la nueva cabecera.
            y = height - 80 # Restablece la posición Y al inicio de la nueva página.
            c.drawString(50, y, "ID") # Redibuja la cabecera "ID" en la nueva página.
            c.drawString(100, y, "Descripción") # Redibuja la cabecera "Descripción".
            c.drawString(300, y, "Monto") # Redibuja la cabecera "Monto".
            c.drawString(400, y, "Fecha") # Redibuja la cabecera "Fecha".
            y -= 20 # Disminuye Y para la primera fila de datos en la nueva página.
            c.setFont("Helvetica", 10) # Restablece la fuente para los datos.
        c.drawString(50, y, str(g.get('id', idx))) # Dibuja el ID del gasto.
        c.drawString(100, y, g.get('descripcion','')[:30]) # Dibuja la descripción (truncada a 30 caracteres).
        c.drawString(300, y, f"{float(g.get('monto',0)):.2f}") # Dibuja el monto formateado.
        c.drawString(400, y, g.get('fecha','')) # Dibuja la fecha.
        y -= 20 # Disminuye Y para el siguiente gasto.

    y -= 20 # Deja un espacio adicional después de la tabla de gastos.
    c.setFont("Helvetica-Bold", 12) # Establece la fuente para el total.
    c.drawString(50, y, f"TOTAL: {total:.2f} - Fecha de exportación: {fecha_display}") # Dibuja el total y la fecha de exportación.

    if y < 180: # Comprueba si queda poco espacio para las estadísticas.
        c.showPage() # Si es así, crea una nueva página.
        y = height - 80 # Restablece la posición Y.

    y -= 40 # Deja un margen antes de las estadísticas detalladas.
    c.setFont("Helvetica-Bold", 14) # Establece la fuente para el título de las estadísticas.
    c.drawString(50, y, "Estadísticas Detalladas") # Dibuja el título.
    y -= 20 # Disminuye Y.
    c.setFont("Helvetica", 12) # Establece la fuente para los ítems de estadísticas.
    c.drawString(50, y, f"Promedio: {stats['promedio']:.2f}") # Dibuja el promedio.
    y -= 20 # Disminuye Y.
    c.drawString(50, y, f"Gasto máximo: {stats['mayor']:.2f}") # Dibuja el gasto máximo.
    y -= 20 # Disminuye Y.
    c.drawString(50, y, f"Gasto mínimo: {stats['menor']:.2f}") # Dibuja el gasto mínimo.

    if y < 120: # Comprueba si queda poco espacio para la distribución por categoría.
        c.showPage() # Si es así, crea una nueva página.
        y = height - 80 # Restablece la posición Y.

    y -= 40 # Deja un margen antes de la distribución por categoría.
    c.setFont("Helvetica-Bold", 14) # Establece la fuente para el título de distribución.
    c.drawString(50, y, "Distribución por categoría") # Dibuja el título.
    y -= 20 # Disminuye Y.
    c.setFont("Helvetica", 12) # Establece la fuente para los ítems de categorías.
    for cat, pct in stats['por_categoria'].items(): # Itera sobre las categorías y sus porcentajes.
        if y < 100: # Comprueba si queda poco espacio para el siguiente ítem de categoría.
            c.showPage() # Si es así, crea una nueva página.
            y = height - 80 # Restablece la posición Y.
            c.setFont("Helvetica-Bold", 14) # Redibuja el título de distribución en la nueva página.
            c.drawString(50, y, "Distribución por categoría")
            y -= 20 # Disminuye Y.
            c.setFont("Helvetica", 12) # Restablece la fuente para los ítems de datos.
        c.drawString(50, y, f"{cat}: {pct:.2f}%") # Dibuja la categoría y su porcentaje.
        y -= 20 # Disminuye Y.

    if y < 60: # Comprueba si queda poco espacio para la categoría con mayor gasto.
        c.showPage() # Si es así, crea una nueva página.
        y = height - 80 # Restablece la posición Y.

    y -= 20 # Deja un margen adicional.
    c.setFont("Helvetica-Bold", 14) # Establece la fuente para la categoría con mayor gasto.
    c.drawString(50, y, f"Categoría con mayor gasto: {stats.get('categoria_mayor')}") # Dibuja el texto.

    c.save() # Guarda el archivo PDF al disco.
    print(f"Gastos exportados a {nombre_archivo}") # Informa al usuario sobre el archivo creado.

# --- Menú Principal de la Aplicación ---

def menu(): # Define la función del menú principal de la aplicación.
    datos = cargar_datos() # Al iniciar el programa, carga los datos de gastos existentes desde 'gastos.json'.
    while True: # Inicia un bucle infinito para mostrar el menú continuamente.
        print("\n--- Menú de Gastos ---") # Imprime el encabezado del menú.
        print("1. Ver lista de todos los gastos") # Opción 1.
        print("2. Registrar un nuevo gasto") # Opción 2.
        print("3. Modificar un gasto existente") # Opción 3.
        print("4. Eliminar un gasto") # Opción 4.
        print("5. Ver estadísticas") # Opción 5.
        print("6. Eliminar todos los gastos") # Opción 6.
        print("7. Exportar (TXT o PDF)") # Opción 7 (con sub-opciones).
        print("8. Salir y guardar") # Opción 8 (para terminar el programa y guardar).
        opcion = input("Elegí una opción: ").strip() # Pide al usuario que elija una opción y elimina espacios.

        # Serie de condiciones para ejecutar la función correspondiente a la opción elegida.
        if opcion == "1": ver_gastos(datos) # Si el usuario elige '1', llama a 'ver_gastos'.
        elif opcion == "2": registrar_gasto(datos) # Si el usuario elige '2', llama a 'registrar_gasto'.
        elif opcion == "3": modificar_gasto(datos) # Si el usuario elige '3', llama a 'modificar_gasto'.
        elif opcion == "4": eliminar_gasto(datos) # Si el usuario elige '4', llama a 'eliminar_gasto'.
        elif opcion == "5": ver_estadisticas(datos) # Si el usuario elige '5', llama a 'ver_estadisticas'.
        elif opcion == "6": eliminar_todos(datos) # Si el usuario elige '6', llama a 'eliminar_todos'.
        elif opcion == "7": # Si el usuario elige '7' (Exportar), muestra sub-opciones.
            print("1. Exportar a TXT") # Sub-opción para TXT.
            print("2. Exportar a PDF") # Sub-opción para PDF.
            fmt = input("Elegí formato: ").strip() # Pide el formato deseado.
            if fmt == "1": exportar_a_txt(datos) # Si elige '1', llama a 'exportar_a_txt'.
            elif fmt == "2": exportar_a_pdf(datos) # Si elige '2', llama a 'exportar_a_pdf'.
            else: print("Formato no válido.") # Si la sub-opción no es válida.
        elif opcion == "8": # Si el usuario elige '8' (Salir).
            guardar_datos(datos) # Llama a 'guardar_datos' para guardar todos los cambios antes de salir.
            print("Datos guardados. Saliendo...") # Informa al usuario.
            break # Rompe el bucle 'while True', terminando la ejecución de la función 'menu' y del programa.
        else: # Si la opción ingresada no coincide con ninguna de las válidas.
            print("Opción no válida.") # Informa al usuario.

if __name__ == "__main__": # Este bloque asegura que la función 'menu()' solo se ejecute cuando el script se corre directamente.
    print("El menú está por ejecutarse...") # Mensaje informativo (opcional).
    menu() # Llama a la función 'menu' para iniciar la aplicación.
