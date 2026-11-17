# Cómo abrir los apuntes sin configurar SQL Server

Para **leer** los apuntes no hace falta instalar SQL Server, Docker ni configurar la extensión SQL Server de VS Code. Esa configuración ([Guia_Setup.md](Guia_Setup.md)) solo es necesaria para **ejecutar** las consultas. Los archivos ipynb pueden abrirse desde github online o en una IDE.

## Qué hay en cada carpeta

| Carpeta | Qué tiene | Formato |
|---|---|---|
| `1-apuntes&Ejercicios/02-ApuntesJupyter/` | Clases 1 a 7 (teoría + ejemplos) y `EjerciciosGuia.ipynb`, con **todos los ejercicios de la guía resueltos** (SQL y T-SQL) | `.ipynb` |
| `1-apuntes&Ejercicios/03-ApuntesTeoricos/` | Resúmenes para los parciales | `.md` y `.pdf` |
| `1-apuntes&Ejercicios/05-EjerciciosSueltos/` | Ejercicios de clase y soluciones de parciales | `.sql` |
| `1-apuntes&Ejercicios/00-parcial/` | Práctica de parcial | `.ipynb` |
| `1-apuntes&Ejercicios/01-apuntesSQL/` | Práctica inicial (tablas de bancos) | `.sql` |
| `0-guias/` | Estas guías | `.md` |

Si es la primera vez, empezá por `03-ApuntesTeoricos/Parcial-1-LeerAntesDePracticar` (teoría desde cero) y seguí con los cuadernos `Clase1` a `Clase7`.

---

## Archivos `.pdf`

Se abren con cualquier visor de PDF. Los dos resúmenes de parcial (`Parcial-1-...` y `Parcial-2-...`) están también en PDF: es la forma más simple de leerlos.

## Archivos `.md` (resúmenes y guías)

- **VS Code (sin extensiones):** abrí el archivo y presioná `Cmd+Shift+V` (macOS) o `Ctrl+Shift+V` (Windows) para ver la vista previa con formato. Se ven los títulos, tablas y el código SQL coloreado.
- **Otros visores de Markdown** (GitHub, Obsidian, Typora, etc.): el texto y el código se leen bien, pero puede que no aparezcan los colores, porque vienen de estilos HTML que algunos visores ignoran.

## Archivos `.sql`

Son texto plano: se abren con cualquier editor (VS Code, Bloc de notas, TextEdit, etc.). Cada ejercicio tiene el enunciado en un comentario (`--` o `/* ... */`) arriba de la solución. VS Code los colorea sin necesidad de extensiones.

## Notebooks `.ipynb` (clases y ejercicios)

Un notebook mezcla celdas de texto (explicaciones) con celdas de código SQL. Tené en cuenta que **la mayoría de las celdas de código no tienen el resultado guardado**: vas a ver la consulta y la explicación, pero no la tabla que devuelve. Para ver resultados hay que ejecutarlas (ver el final de esta guía).

Elegí una de estas opciones:

### Opción A: VS Code + extensión Jupyter (recomendada)

1. En VS Code instalá solo la extensión **Jupyter** de Microsoft (`Cmd+Shift+X` / `Ctrl+Shift+X` → buscar "Jupyter"). Si abrís un `.ipynb` sin tenerla, VS Code te ofrece instalarla.
2. Abrí la carpeta de los apuntes (*File → Open Folder*) y hacé doble clic en el notebook.
3. Se ve todo con formato y colores. Si arriba a la derecha aparece **Select Kernel**, ignoralo: el kernel solo hace falta para ejecutar.

Funciona igual en macOS y Windows.

### Opción B: sin instalar nada, en el navegador

- **Google Colab:** entrá a [colab.research.google.com](https://colab.research.google.com), *Archivo → Subir notebook* y elegí el `.ipynb`. Se lee el contenido, aunque algunos colores pueden no mostrarse y no se puede ejecutar el SQL ahí.
- **GitHub:** si los apuntes están subidos a un repositorio, GitHub muestra los `.ipynb` directamente al abrirlos. Los notebooks muy pesados pueden no cargar; en ese caso usá la opción A o C.

### Opción C: Jupyter en la computadora

Si ya tenés Python:

```bash
pip install notebook
jupyter notebook
```

Se abre el navegador; navegá hasta el `.ipynb`. Va a avisar que no encuentra el kernel **SQL**: elegí **No Kernel** (o cerrá el aviso) y leé normalmente. En Windows, si `pip` no se reconoce, probá con `py -m pip install notebook` y `py -m notebook`.

### Opción D: convertirlos a HTML para compartir

Quien comparte los apuntes puede convertir los notebooks a páginas HTML una sola vez, y así cualquiera los abre con el navegador, con doble clic:

```bash
pip install nbconvert
cd "1-apuntes&Ejercicios/02-ApuntesJupyter"
jupyter nbconvert --to html *.ipynb
```

En Windows (PowerShell) los comandos son los mismos (con `py -m` delante si `jupyter` no se reconoce). Se genera un `.html` al lado de cada `.ipynb`.

---

## ¿Y si quiero ejecutar algo sin configurar la extensión?

Para ejecutar SQL sí hace falta una base de datos, así que como mínimo necesitás **Docker y la base restaurada**: pasos 0.1 y 1 a 6 de [Guia_Setup.md](Guia_Setup.md). Lo que te podés ahorrar es la extensión SQL Server y los perfiles de conexión de VS Code: el contenedor trae su propia herramienta de línea de comandos, `sqlcmd`.

1. Copiá la consulta que quieras probar (de un `.sql` o de una celda de un notebook) a un archivo, por ejemplo `prueba.sql`.
2. Con el contenedor `sql-gestion2016` encendido, copialo adentro y ejecutalo:

macOS:
```bash
docker cp prueba.sql sql-gestion2016:/tmp/prueba.sql
docker exec sql-gestion2016 /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C -d GESTION2016 -W \
  -i /tmp/prueba.sql
```

Windows (PowerShell):
```powershell
docker cp prueba.sql sql-gestion2016:/tmp/prueba.sql
docker exec sql-gestion2016 /opt/mssql-tools18/bin/sqlcmd `
  -S localhost -U sa -P "$env:MSSQL_SA_PASSWORD" -C -d GESTION2016 -W `
  -i /tmp/prueba.sql
```

(`$MSSQL_SA_PASSWORD` / `$env:MSSQL_SA_PASSWORD` es la contraseña de `sa`; si abriste una terminal nueva, definila otra vez como en el paso 3 de la guía de configuración, o escribila directamente entre comillas.)

El resultado sale como texto en la terminal. Sirve para probar consultas sueltas; para los resultados en tabla y para ejecutar los notebooks celda por celda, conviene la configuración completa de VS Code.

> Se usa `docker cp` en vez de pasar el archivo con `<` o `|` para que en Windows no se rompan los acentos.
