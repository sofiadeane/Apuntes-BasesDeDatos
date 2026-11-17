# Guía de configuración: SQL Server con Docker y VS Code (desde cero)

Esta guía arma todo el entorno para correr los apuntes y el TP: **SQL Server en Docker** + **VS Code** con la extensión **SQL Server** y **Jupyter** (para los notebooks `.ipynb`).

> ¿Solo querés **leer** los apuntes, sin ejecutar nada? No hace falta nada de esto: mirá [Guia_Abrir_Apuntes.md](Guia_Abrir_Apuntes.md).
>
> ¿Ya tenés todo configurado? El uso de todos los días está en [Guia_Despliegue.md](Guia_Despliegue.md).

## Cómo queda configurado

- Dos contenedores de SQL Server, uno por base:
  - `sql-gestion2016` → base `GESTION2016` (la de las clases prácticas y los apuntes).
  - `sql-gd2c2026` → base `GD2C_2026` (la del Trabajo Práctico).
- Cada contenedor tiene su propio **volumen** (`gestion2016_data` y `gd2c2026_data`): detener o recrear el contenedor no borra los datos.
- Los dos usan el puerto `localhost:1433`, así que **se inicia uno por vez**.
- En VS Code hay dos perfiles de conexión: `ClasesPracticas` y `TrabajoPractico`.

Si solo vas a usar los apuntes, alcanza con `GESTION2016`: podés saltear los pasos del TP (7 y 8).

### Antes de empezar: qué tenés que tener

- **VS Code** instalado.
- El backup de la base de las clases: **`GD2015C1.bak`** (en estos apuntes está en `1-apuntes&Ejercicios/`).
- Opcional, para el TP: **`GD2C_2026_20260903_221157.bak`** (en `2-TP/Enunciado/Base de Datos/`).
- Unos 10 GB libres (imagen de SQL Server + volúmenes).

---

## Paso 0. Instalar Docker y las extensiones de VS Code

### 0.1 Docker Desktop

**macOS**
1. Descargá Docker Desktop desde [docker.com](https://www.docker.com/products/docker-desktop/) (elegí *Apple Silicon* si tu Mac tiene chip M1/M2/M3/M4, o *Intel*).
2. Instalalo, abrilo y aceptá los permisos que pida.
3. **Mac con Apple Silicon:** la imagen de SQL Server es solo para procesadores Intel/AMD (`amd64`), así que corre emulada. En Docker Desktop → *Settings* → *General*, dejá activada **"Use Rosetta for x86_64/amd64 emulation on Apple Silicon"** (en las versiones recientes viene activada). Es más lento que en una PC, pero para estas bases alcanza.

**Windows**
1. Docker Desktop en Windows necesita **WSL 2**. Abrí **PowerShell como administrador** y ejecutá:
   ```powershell
   wsl --install
   ```
   Reiniciá la PC cuando termine.
2. Si al iniciar Docker aparece un error de virtualización, hay que activar la **virtualización** (Intel VT-x / AMD-V, a veces llamada *SVM*) en la BIOS/UEFI.
3. Descargá e instalá Docker Desktop desde [docker.com](https://www.docker.com/products/docker-desktop/). En el instalador dejá marcado **"Use WSL 2 instead of Hyper-V"**.
4. Abrí Docker Desktop y esperá a que diga que el motor está iniciado.

> **Windows sin Docker:** si Docker no anda en tu PC (por ejemplo, una notebook sin virtualización), mirá la [alternativa con SQL Server instalado en Windows](#alternativa-windows-sql-server-instalado-sin-docker) al final.

Para comprobar que Docker funciona, abrí una terminal (Terminal en macOS, **PowerShell** en Windows) y ejecutá:

```bash
docker run --rm hello-world
```

Tiene que aparecer el mensaje `Hello from Docker!`.

### 0.2 Extensiones de VS Code

En VS Code, abrí la vista de extensiones (`Cmd+Shift+X` en macOS, `Ctrl+Shift+X` en Windows) e instalá:

| Extensión | Para qué |
|---|---|
| **SQL Server (mssql)** de Microsoft | Conectarse a la base, ejecutar archivos `.sql` y el kernel **SQL** de los notebooks |
| **Jupyter** de Microsoft | Abrir y ejecutar los notebooks `.ipynb` |

Son las mismas en macOS y Windows.

---

## Paso 1. Configuración inicial (una sola vez)

### Cómo leer los comandos

Cada comando está en dos versiones:
- **macOS (Terminal, bash/zsh)**: las líneas largas se cortan con `\` al final.
- **Windows (PowerShell)**: las líneas largas se cortan con un acento grave `` ` `` al final. **Usá PowerShell, no `cmd`**: en `cmd` los comandos no funcionan como están escritos (y la carpeta `1-apuntes&Ejercicios` da problemas por el `&`).

Hacé toda la configuración inicial **en la misma ventana de terminal**: los pasos 2 y 3 guardan variables que usan los pasos siguientes. Si cerrás la terminal, volvé a correr esos dos pasos.

### 1. Abrí Docker Desktop

Esperá a que indique que está iniciado.

### 2. Indicá dónde están los `.bak`

Cambiá las rutas por las de tu máquina si guardaste los archivos en otro lado.

macOS:
```bash
BAK_GESTION="$HOME/Documents/UTN/BD/1-apuntes&Ejercicios/GD2015C1.bak"
BAK_TP="$HOME/Documents/UTN/BD/2-TP/Enunciado/Base de Datos/GD2C_2026_20260903_221157.bak"
ls -lh "$BAK_GESTION" "$BAK_TP"
```

Windows (PowerShell):
```powershell
$BAK_GESTION = "$HOME\Documents\UTN\BD\1-apuntes&Ejercicios\GD2015C1.bak"
$BAK_TP = "$HOME\Documents\UTN\BD\2-TP\Enunciado\Base de Datos\GD2C_2026_20260903_221157.bak"
Get-Item $BAK_GESTION, $BAK_TP
```

Si aparece `No such file or directory` (macOS) o `Cannot find path` (Windows), corregí la ruta antes de seguir. **Las comillas son obligatorias**: las rutas tienen espacios y `&`.

> En Windows, si tus Documentos están sincronizados con OneDrive, la ruta puede ser `$HOME\OneDrive\Documentos\...`. Buscá el archivo en el Explorador, hacé clic derecho → *Copiar como ruta de acceso* y pegala entre las comillas.

### 3. Elegí la contraseña de `sa`

`sa` es el usuario administrador de SQL Server. La contraseña tiene que tener **al menos 8 caracteres** y combinar mayúsculas, minúsculas y números o símbolos. Si no cumple, el contenedor se apaga apenas arranca. Evitá los caracteres `!`, `$`, `"`, `'` y `` ` ``, porque las terminales los interpretan de forma especial. Usá la misma contraseña en los dos contenedores y guardala en un lugar privado (no en estos apuntes si los vas a compartir).

macOS (no se ve mientras la escribís):
```bash
printf 'Contraseña de sa: '
read -s MSSQL_SA_PASSWORD
printf '\n'
export MSSQL_SA_PASSWORD
```

Windows (PowerShell):
```powershell
$env:MSSQL_SA_PASSWORD = Read-Host "Contraseña de sa"
```

### 4. Liberá el puerto `1433`

Mirá qué contenedores existen:

```bash
docker ps -a
```

Si tenés algún contenedor de SQL Server viejo que use el puerto 1433, detenelo (no hace falta borrarlo):

```bash
docker stop NOMBRE_DEL_CONTENEDOR
```

En Windows también puede ocupar el puerto un **SQL Server instalado en la PC**. Si en el paso 5 aparece un error de puerto ocupado, abrí *Servicios* (`services.msc`) y detené **SQL Server (MSSQLSERVER)**, o mirá la sección de problemas frecuentes.

### 5. Creá el contenedor para `GESTION2016`

macOS:
```bash
docker volume create gestion2016_data

docker run --name sql-gestion2016 --hostname sql-gestion2016 \
  --platform linux/amd64 \
  -e 'ACCEPT_EULA=Y' \
  -e "MSSQL_SA_PASSWORD=$MSSQL_SA_PASSWORD" \
  -p 1433:1433 \
  -v gestion2016_data:/var/opt/mssql \
  -d mcr.microsoft.com/mssql/server:2022-latest
```

Windows (PowerShell):
```powershell
docker volume create gestion2016_data

docker run --name sql-gestion2016 --hostname sql-gestion2016 `
  -e "ACCEPT_EULA=Y" `
  -e "MSSQL_SA_PASSWORD=$env:MSSQL_SA_PASSWORD" `
  -p 1433:1433 `
  -v gestion2016_data:/var/opt/mssql `
  -d mcr.microsoft.com/mssql/server:2022-latest
```

La primera vez descarga la imagen (unos 1,5 GB). Esperá 20-30 segundos (más en Mac con Apple Silicon) y revisá el estado:

```bash
docker ps
```

`sql-gestion2016` tiene que aparecer con estado `Up`. Si no aparece, mirá el motivo con `docker logs --tail 50 sql-gestion2016` (lo más común es una contraseña que no cumple los requisitos: en ese caso borrá el contenedor con `docker rm sql-gestion2016` y repetí el paso 3 y este).

> **Si el `docker exec ... sqlcmd` del paso siguiente dice que no encuentra `/opt/mssql-tools18/bin/sqlcmd`:** en algunas versiones de la imagen la herramienta está en `/opt/mssql-tools/bin/sqlcmd` (sin el `18`, y sin la opción `-C`). Reemplazá la ruta en todos los comandos.

### 6. Restaurá `GESTION2016` desde el backup

Copiá el `.bak` adentro del contenedor:

macOS:
```bash
docker cp "$BAK_GESTION" sql-gestion2016:/var/opt/mssql/data/GD2015C1.bak
```

Windows (PowerShell):
```powershell
docker cp $BAK_GESTION sql-gestion2016:/var/opt/mssql/data/GD2015C1.bak
```

Consultá los nombres lógicos guardados dentro del backup:

macOS:
```bash
docker exec sql-gestion2016 /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C \
  -Q "RESTORE FILELISTONLY FROM DISK = '/var/opt/mssql/data/GD2015C1.bak';"
```

Windows (PowerShell):
```powershell
docker exec sql-gestion2016 /opt/mssql-tools18/bin/sqlcmd `
  -S localhost -U sa -P "$env:MSSQL_SA_PASSWORD" -C `
  -Q "RESTORE FILELISTONLY FROM DISK = '/var/opt/mssql/data/GD2015C1.bak';"
```

En el resultado, buscá la columna `LogicalName`. Para `GD2015C1.bak` son `GESTION2009_Data` y `GESTION2009_Log`. Si te aparecen otros nombres, reemplazalos en el comando siguiente; si aparecen más de dos archivos, agregá un `MOVE` por cada uno.

macOS:
```bash
docker exec sql-gestion2016 /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C \
  -Q "RESTORE DATABASE [GESTION2016] FROM DISK = '/var/opt/mssql/data/GD2015C1.bak' WITH MOVE 'GESTION2009_Data' TO '/var/opt/mssql/data/GESTION2016.mdf', MOVE 'GESTION2009_Log' TO '/var/opt/mssql/data/GESTION2016_log.ldf';"
```

Windows (PowerShell):
```powershell
docker exec sql-gestion2016 /opt/mssql-tools18/bin/sqlcmd `
  -S localhost -U sa -P "$env:MSSQL_SA_PASSWORD" -C `
  -Q "RESTORE DATABASE [GESTION2016] FROM DISK = '/var/opt/mssql/data/GD2015C1.bak' WITH MOVE 'GESTION2009_Data' TO '/var/opt/mssql/data/GESTION2016.mdf', MOVE 'GESTION2009_Log' TO '/var/opt/mssql/data/GESTION2016_log.ldf';"
```

Las rutas que empiezan con `/var/opt/mssql/...` son **de adentro del contenedor** (que es Linux): se escriben igual en macOS y en Windows.

Verificá que la base se haya restaurado (en Windows cambiá `\` por `` ` `` y `$MSSQL_SA_PASSWORD` por `$env:MSSQL_SA_PASSWORD`, como en los comandos anteriores):

```bash
docker exec sql-gestion2016 /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C \
  -Q "SELECT name FROM sys.databases ORDER BY name;"
```

Cuando `GESTION2016` aparezca en la lista, borrá **solo la copia interna** del `.bak` (el archivo original de tu computadora no se toca) y, si vas a configurar también el TP, detené el contenedor:

```bash
docker exec -u root sql-gestion2016 rm /var/opt/mssql/data/GD2015C1.bak
docker stop sql-gestion2016
```

### 7. (TP) Creá el contenedor para `GD2C_2026`

Es igual al paso 5, cambiando el nombre del contenedor y del volumen.

macOS:
```bash
docker volume create gd2c2026_data

docker run --name sql-gd2c2026 --hostname sql-gd2c2026 \
  --platform linux/amd64 \
  -e 'ACCEPT_EULA=Y' \
  -e "MSSQL_SA_PASSWORD=$MSSQL_SA_PASSWORD" \
  -p 1433:1433 \
  -v gd2c2026_data:/var/opt/mssql \
  -d mcr.microsoft.com/mssql/server:2022-latest
```

Windows (PowerShell):
```powershell
docker volume create gd2c2026_data

docker run --name sql-gd2c2026 --hostname sql-gd2c2026 `
  -e "ACCEPT_EULA=Y" `
  -e "MSSQL_SA_PASSWORD=$env:MSSQL_SA_PASSWORD" `
  -p 1433:1433 `
  -v gd2c2026_data:/var/opt/mssql `
  -d mcr.microsoft.com/mssql/server:2022-latest
```

Revisá con `docker ps` que esté `Up`.

### 8. (TP) Restaurá `GD2C_2026`

Copiá el backup (macOS: `"$BAK_TP"` con comillas; Windows: `$BAK_TP`):

```bash
docker cp "$BAK_TP" sql-gd2c2026:/var/opt/mssql/data/GD2C_2026.bak
```

Consultá los nombres lógicos:

```bash
docker exec sql-gd2c2026 /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C \
  -Q "RESTORE FILELISTONLY FROM DISK = '/var/opt/mssql/data/GD2C_2026.bak';"
```

En el backup del TP 2C2026 los nombres lógicos son `GD2C_2026` y `GD2C_2026_log`. Si te aparecen otros, reemplazalos:

```bash
docker exec sql-gd2c2026 /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C \
  -Q "RESTORE DATABASE [GD2C_2026] FROM DISK = '/var/opt/mssql/data/GD2C_2026.bak' WITH MOVE 'GD2C_2026' TO '/var/opt/mssql/data/GD2C_2026.mdf', MOVE 'GD2C_2026_log' TO '/var/opt/mssql/data/GD2C_2026_log.ldf';"
```

(En Windows, mismos cambios de siempre: `` ` `` en vez de `\` y `$env:MSSQL_SA_PASSWORD`.)

Verificá con `SELECT name FROM sys.databases ORDER BY name;` (como en el paso 6) y, cuando `GD2C_2026` aparezca, borrá la copia interna y detené el contenedor:

```bash
docker exec -u root sql-gd2c2026 rm /var/opt/mssql/data/GD2C_2026.bak
docker stop sql-gd2c2026
```

### 9. Configurá las conexiones en VS Code

Iniciá el contenedor de la base que vas a configurar (`docker start sql-gestion2016`). En VS Code abrí la vista **SQL Server** (ícono del cilindro en la barra lateral) → **Add Connection** y completá:

| Campo | `ClasesPracticas` | `TrabajoPractico` |
|---|---|---|
| Profile Name | `ClasesPracticas` | `TrabajoPractico` |
| Server name | `localhost` | `localhost` |
| Port | `1433` | `1433` |
| Authentication type | `SQL Login` | `SQL Login` |
| User name | `sa` | `sa` |
| Password | La contraseña del paso 3 | La misma contraseña |
| Save Password | Activado | Activado |
| Database name | `GESTION2016` | `GD2C_2026` |
| Trust server certificate | Activado | Activado |
| Encrypt | `Mandatory` | `Mandatory` |

**Trust server certificate tiene que estar activado**: el contenedor usa un certificado autofirmado y, sin esa opción, la conexión falla con un error de certificado. Cada perfil se prueba con su contenedor encendido.

### 10. Probá que todo funcione

1. **Archivo `.sql`:** abrí [Primera_consulta_ClasesPracticas.sql](Primera_consulta_ClasesPracticas.sql), elegí la conexión `ClasesPracticas` cuando la pida (o desde la barra inferior) y ejecutalo con el botón **Run** (▷) o `Cmd+Shift+E` / `Ctrl+Shift+E`. Tiene que mostrar la base conectada y la lista de tablas (`Cliente`, `Factura`, `Item_Factura`, ...).
2. **Notebook `.ipynb`:** abrí cualquier cuaderno de `1-apuntes&Ejercicios/02-ApuntesJupyter/`. Arriba a la derecha, en **Select Kernel**, elegí el kernel **SQL** (lo agrega la extensión SQL Server) y después la conexión `ClasesPracticas`. Ejecutá una celda con ▷.

Listo: a partir de acá seguí con [Guia_Despliegue.md](Guia_Despliegue.md) cada vez que quieras trabajar.

---

## Diferencias entre macOS y Windows (resumen)

| Qué | macOS (Terminal) | Windows (PowerShell) |
|---|---|---|
| Cortar un comando largo | `\` al final de la línea | `` ` `` al final de la línea |
| Variable de la contraseña | `$MSSQL_SA_PASSWORD` | `$env:MSSQL_SA_PASSWORD` |
| Rutas de tu computadora | `"$HOME/Documents/..."` | `"$HOME\Documents\..."` |
| Ignorar el error si un contenedor no existe | `docker stop a b 2>/dev/null \|\| true` | `docker stop a b 2>$null` |
| Ejecutar en VS Code | `Cmd+Shift+E` | `Ctrl+Shift+E` |
| Requisito de Docker | Rosetta activado si es Apple Silicon | WSL 2 + virtualización en la BIOS |
| Rutas dentro del contenedor (`/var/opt/mssql/...`) | Iguales | Iguales |
| Comandos `docker ...` | Iguales | Iguales |

---

## Alternativa Windows: SQL Server instalado (sin Docker)

Si en tu PC no se puede usar Docker, podés instalar SQL Server directamente en Windows:

1. Descargá **SQL Server 2022 Developer** (gratis) desde la [página de descargas de Microsoft](https://www.microsoft.com/es-es/sql-server/sql-server-downloads) e instalalo con la opción **Básica**. Se instala como el servicio `MSSQLSERVER` en el puerto 1433.
2. Copiá `GD2015C1.bak` a la carpeta de backups del servidor, para que el servicio de SQL Server tenga permiso de leerlo. Por defecto es `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\`.
3. En VS Code creá una conexión con **Server name** `localhost`, **Authentication type** `Integrated` (tu usuario de Windows) y **Trust server certificate** activado, sin elegir base de datos.
4. Abrí un archivo `.sql` nuevo con esa conexión y ejecutá:
   ```sql
   RESTORE FILELISTONLY FROM DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\GD2015C1.bak';

   RESTORE DATABASE [GESTION2016]
   FROM DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\GD2015C1.bak'
   WITH MOVE 'GESTION2009_Data' TO 'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\DATA\GESTION2016.mdf',
        MOVE 'GESTION2009_Log'  TO 'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\DATA\GESTION2016_log.ldf';
   ```
   (Si tu instalación usa otra versión, la carpeta puede llamarse `MSSQL15...` en vez de `MSSQL16...`: fijate cuál existe.)
5. Editá la conexión para que use la base `GESTION2016`. Desde ahí, todo funciona igual que con Docker. En este caso no hay contenedores que iniciar: el servicio arranca solo con Windows.

---

## Problemas frecuentes

- **`Login failed for user 'sa'` (Error 18456):** verificá que el perfil tenga la contraseña correcta y que no esté vacía. Tiene que coincidir con la que usaste al crear el contenedor.
- **Error de certificado al conectar (`The certificate chain was issued by an authority that is not trusted`):** activá **Trust server certificate** en el perfil.
- **Puerto ocupado (`port is already allocated` / `Address already in use`):** hay otro contenedor (o, en Windows, un SQL Server instalado) usando el 1433. Detené el otro contenedor (`docker stop sql-gestion2016 sql-gd2c2026`) o el servicio `MSSQLSERVER` de Windows, y volvé a iniciar solo el que necesitás.
- **El contenedor se apaga al iniciarse:** mirá `docker logs --tail 100 NOMBRE_DEL_CONTENEDOR`. Lo más común es una contraseña que no cumple los requisitos de complejidad o poca memoria asignada a Docker (SQL Server necesita al menos 2 GB: *Settings* → *Resources*).
- **Falla el `RESTORE DATABASE`:** ejecutá de nuevo `RESTORE FILELISTONLY` y comprobá los nombres lógicos. Tiene que haber un `MOVE` por cada archivo del backup.
- **En Mac con Apple Silicon todo anda muy lento o el contenedor no arranca:** revisá que esté activada la emulación con Rosetta (paso 0.1) y que Docker Desktop esté actualizado.
- **El notebook no muestra el kernel SQL:** confirmá que estén instaladas las extensiones **SQL Server (mssql)** y **Jupyter**, y recargá VS Code (`Cmd/Ctrl+Shift+P` → *Developer: Reload Window*).
- **Persistencia:** no borres los volúmenes `gestion2016_data` ni `gd2c2026_data` (`docker volume rm`): ahí están guardadas las bases.

## Referencias oficiales

- [Docker Desktop para Windows (WSL 2)](https://docs.docker.com/desktop/setup/install/windows-install/)
- [Docker Desktop para Mac](https://docs.docker.com/desktop/setup/install/mac-install/)
- [Microsoft Learn: SQL Server en contenedores Docker](https://learn.microsoft.com/es-es/sql/linux/quickstart-install-connect-docker)
- [Microsoft Learn: restaurar una base desde un backup en un contenedor](https://learn.microsoft.com/es-es/sql/linux/tutorial-restore-backup-in-sql-server-container)
- [Microsoft Learn: extensión SQL Server (mssql) para VS Code](https://learn.microsoft.com/es-es/sql/tools/visual-studio-code-extensions/mssql/mssql-extension-visual-studio-code)
- [Docker Docs: persistir datos con volúmenes](https://docs.docker.com/get-started/docker-concepts/running-containers/persisting-container-data/)
