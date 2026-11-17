# Guía: dejar VS Code sin IA para el parcial

Tu VS Code (versión 1.144) trae **GitHub Copilot integrado de fábrica**. No aparece como extensión instalada, así que no se puede desinstalar: hay que **apagarlo desde la configuración**. Además tenés instalada la extensión **Claude Code**, que también es IA y también se ve en pantalla.

La forma más prolija es crear un **perfil "Parcial"** sin IA. Para rendir cambiás a ese perfil, y para estudiar volvés al de siempre. No tenés que tocar nada de tu configuración normal.

> Hacelo **unos días antes** y probalo, no el día del parcial.

---

## 1. Crear el perfil "Parcial"

1. Abrí VS Code.
2. Abajo a la izquierda, hacé clic en el **engranaje ⚙️** → **Profiles** (Perfiles).
3. Clic en **New Profile…** (Nuevo perfil).
4. Nombre: `Parcial`.
5. En **Copy from** (Copiar desde), elegí **Default**. Así se copian tu conexión a `GESTION2016`, el tema y la extensión `mssql`.
6. Clic en **Create** (Crear).
7. Fijate que el perfil activo sea `Parcial`: el nombre aparece en el engranaje de abajo a la izquierda.

## 2. Apagar Copilot en ese perfil

Con el perfil `Parcial` activo:

1. Abrí la configuración con **Cmd + ,**.
2. En el buscador, escribí: `disable AI features`.
3. Tildá **Chat: Disable AI Features** ("Disable and hide built-in AI features provided by GitHub Copilot, including chat and inline suggestions"). Con esto se van el chat, el ícono de Copilot y las sugerencias en gris.

**Refuerzo** (por si alguna opción queda prendida):

1. **Cmd + Shift + P** → escribí `Preferences: Open User Settings (JSON)` → Enter.
2. Antes de la última `}`, agregá estas líneas. Cuidado: la línea anterior tiene que terminar en coma.

```json
    "chat.disableAIFeatures": true,
    "github.copilot.enable": { "*": false },
    "editor.inlineSuggest.enabled": false
```

3. Guardá con **Cmd + S**.

## 3. Desactivar Claude Code en ese perfil

1. Abrí la vista de extensiones con **Cmd + Shift + X**.
2. Buscá `Claude Code`.
3. Clic en la extensión → **Disable** (Deshabilitar). Si te pregunta, elegí deshabilitarla **en este perfil**.
4. Si aparece **Reload** / **Restart Extensions**, hacé clic.

> No la desinstales: solo deshabilitala en el perfil `Parcial`. En tu perfil de siempre sigue funcionando.

## 4. Verificar (checklist)

Con el perfil `Parcial` activo, abrí un archivo `.sql` y revisá:

- [ ] No hay ícono de Copilot ni de chat en la barra de arriba, en la barra lateral ni en la barra de estado de abajo.
- [ ] **Cmd + Ctrl + I** (abrir chat) no abre ningún panel.
- [ ] Escribís `SELECT * FROM Fac` y esperás unos segundos: **no aparece texto en gris** sugiriendo el resto de la línea.
- [ ] No aparece el ícono ni el panel de Claude Code.
- [ ] La conexión a `GESTION2016` sigue funcionando: ejecutá un `SELECT TOP 5 * FROM Cliente`.

> **IntelliSense no es IA.** La lista desplegable con nombres de tablas y columnas mientras escribís (por ejemplo, `clie_codigo`) la da la extensión `mssql` leyendo la base. Lo que tiene que desaparecer es el **texto gris** que completa líneas enteras. Si no sabés si el autocompletado está permitido en el parcial, preguntale al profe antes.

## 5. El día del parcial

1. Engranaje ⚙️ → **Profiles** → **Parcial**.
2. Repasá rápido el checklist del paso 4.
3. Cerrá las otras apps con IA: la app de Claude, las pestañas del navegador con ChatGPT o Claude, y las terminales con `claude` abierto.

## 6. Después del parcial

Engranaje ⚙️ → **Profiles** → **Default**. Todo vuelve a estar como antes, con Claude Code incluido.
