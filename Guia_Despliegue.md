# Uso habitual: levantar una base y trabajar

Hacé estos pasos cada vez que quieras trabajar. Suponen que ya hiciste la configuración inicial de [Guia_Setup.md](Guia_Setup.md).

Los comandos `docker ...` son iguales en macOS (Terminal) y en Windows (PowerShell), salvo donde se aclara.

## 1. `GESTION2016` (clases prácticas y apuntes)

### 1. Abrí Docker Desktop

Esperá a que indique que está iniciado. (Si usás la [alternativa de Windows sin Docker](Guia_Setup.md#alternativa-windows-sql-server-instalado-sin-docker), salteá los pasos 1 a 3: el servidor arranca solo.)

### 2. Dejá levantada solo esta base

macOS:
```bash
docker stop sql-gestion2016 sql-gd2c2026 2>/dev/null || true
docker start sql-gestion2016
```

Windows (PowerShell):
```powershell
docker stop sql-gestion2016 sql-gd2c2026 2>$null
docker start sql-gestion2016
```

Si no configuraste la base del TP, el `docker stop` avisa que `sql-gd2c2026` no existe: es normal.

### 3. Esperá a que SQL Server termine de iniciar

En Docker Desktop, confirmá que `sql-gestion2016` esté en estado **Running** y esperá unos segundos más (en Mac con Apple Silicon puede tardar hasta un minuto). Si no inicia, ejecutá `docker logs --tail 50 sql-gestion2016`.

### 4. Conectate desde VS Code

En la vista **SQL Server** de VS Code, conectate con el perfil **ClasesPracticas**. Si te pide la contraseña, es la de `sa` que elegiste en la configuración inicial.

### 5. Trabajá

- **Archivos `.sql`:** abrí el archivo, confirmá en la barra inferior que usa el perfil **ClasesPracticas** y ejecutá con **Run** (▷) o `Cmd+Shift+E` (macOS) / `Ctrl+Shift+E` (Windows). Si seleccionás una parte del código, se ejecuta solo eso. Para probar la conexión: [Primera_consulta_ClasesPracticas.sql](Primera_consulta_ClasesPracticas.sql).
- **Notebooks `.ipynb`:** abrilos, elegí el kernel **SQL** en *Select Kernel* (arriba a la derecha) y la conexión **ClasesPracticas**. Ejecutá cada celda con ▷.

### 6. Al terminar (opcional)

```bash
docker stop sql-gestion2016
```

Los datos quedan guardados en el volumen; la próxima vez arrancás desde el paso 1.

---

## 2. `GD2C_2026` (Trabajo Práctico)

Igual que la sección anterior, cambiando el contenedor y el perfil:

macOS:
```bash
docker stop sql-gestion2016 sql-gd2c2026 2>/dev/null || true
docker start sql-gd2c2026
```

Windows (PowerShell):
```powershell
docker stop sql-gestion2016 sql-gd2c2026 2>$null
docker start sql-gd2c2026
```

Después conectate en VS Code con el perfil **TrabajoPractico** y probá con [Primera_consulta_TrabajoPractico.sql](Primera_consulta_TrabajoPractico.sql).

---

## Volver a dejar la base como recién restaurada

Los ejercicios de T-SQL crean triggers, procedimientos y tablas, y algunos modifican datos. Si la base queda "sucia", podés volver a restaurarla desde el `.bak` repitiendo el paso 6 de [Guia_Setup.md](Guia_Setup.md) con `WITH REPLACE` agregado al final del `RESTORE`:

```sql
RESTORE DATABASE [GESTION2016] FROM DISK = '/var/opt/mssql/data/GD2015C1.bak'
WITH REPLACE,
     MOVE 'GESTION2009_Data' TO '/var/opt/mssql/data/GESTION2016.mdf',
     MOVE 'GESTION2009_Log'  TO '/var/opt/mssql/data/GESTION2016_log.ldf';
```

Antes hay que volver a copiar el `.bak` al contenedor (`docker cp`) y desconectarse de la base en VS Code.

---

## Problemas frecuentes

- **`Login failed for user 'sa'` (Error 18456):** el perfil tiene la contraseña vacía o equivocada. Tiene que ser la misma que usaste al crear el contenedor.
- **No conecta y el contenedor está `Running`:** SQL Server todavía está iniciando. Esperá unos segundos y reintentá.
- **Puerto ocupado:** quedó el otro contenedor encendido (o, en Windows, un SQL Server instalado en la PC). Ejecutá el paso 2 de nuevo.
- **El contenedor se apaga al iniciarse:** `docker logs --tail 100 NOMBRE_DEL_CONTENEDOR` muestra el motivo.
- **Error de certificado:** activá **Trust server certificate** en el perfil de conexión.

Más casos en la sección de problemas de [Guia_Setup.md](Guia_Setup.md#problemas-frecuentes).
