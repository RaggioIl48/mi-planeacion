# Mi planeación

Planeador de lecciones por grado y grupo (6°-9°, A/B/C, más Electiva), con progreso por grupo, enlaces a diapositivas/worksheets, notas, y resaltado automático de "hoy tienes clase con este grupo" usando el calendario académico del colegio.

## Publicarla en GitHub Pages (acceso desde cualquier dispositivo)

1. Crea un repositorio nuevo en GitHub (puede ser público o privado; Pages en repos privados requiere plan GitHub Pro).
2. Sube este proyecto:
   ```bash
   git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
   git branch -M main
   git push -u origin main
   ```
3. En el repo: **Settings → Pages → Source → Deploy from a branch**, elige `main` y carpeta `/ (root)`.
4. En 1-2 minutos tu página estará en `https://TU_USUARIO.github.io/TU_REPO/` — accesible desde cualquier dispositivo con internet.

## Conectar Firebase (para que los datos se sincronicen entre dispositivos)

Sin esto, la app funciona en "modo local": cada navegador guarda sus propios datos y no se comparten entre dispositivos.

1. Ve a [https://console.firebase.google.com](https://console.firebase.google.com) e inicia sesión con tu cuenta de Google.
2. **Agregar proyecto** → dale un nombre (ej. `mi-planeacion`) → puedes desactivar Google Analytics, no se necesita.
3. En el panel del proyecto, entra a **Compilación → Firestore Database → Crear base de datos**.
   - Elige la ubicación más cercana.
   - Modo de inicio: **modo de prueba** (test mode) para empezar rápido. *(Ver nota de seguridad abajo.)*
4. Ve a **Configuración del proyecto** (ícono de engranaje) → pestaña **General** → sección "Tus apps" → clic en el ícono `</>` (Web) para registrar una app web.
   - Dale un nombre (ej. `planeacion-web`), no necesitas Firebase Hosting.
   - Copia el objeto `firebaseConfig` que te muestra, algo así:
     ```js
     const firebaseConfig = {
       apiKey: "AIza...",
       authDomain: "mi-planeacion.firebaseapp.com",
       projectId: "mi-planeacion",
       storageBucket: "mi-planeacion.appspot.com",
       messagingSenderId: "123456789",
       appId: "1:123456789:web:abcdef"
     };
     ```
5. Abre [index.html](index.html), busca el bloque `const firebaseConfig = {...}` cerca del inicio del `<script>`, y reemplaza los valores `"TU_API_KEY"`, `"TU_PROYECTO"`, etc. con los tuyos.
6. Guarda, haz commit y push. Al recargar la página verás **"Guardado automático · varios dispositivos"** en vez de "Modo local".

## Conectar el calendario académico (resaltar "hoy tienes clase con...")

El colegio publica un calendario público donde cada día de clase tiene un evento "Día 1", "Día 2"... "Día 6" (el ciclo rotativo de tu horario de campana). La app puede leer ese evento automáticamente y resaltar en la pantalla principal qué grupos te tocan hoy.

1. Ve a [https://console.cloud.google.com](https://console.cloud.google.com) e inicia sesión con la misma cuenta de Google que usaste para Firebase (puedes usar el mismo proyecto, ej. `mi-planeacion`, o crear uno nuevo).
2. En el buscador superior escribe **"Calendar API"**, entra a **Google Calendar API** y haz clic en **Habilitar**.
3. Ve a **APIs y servicios → Credenciales → Crear credenciales → Clave de API**. Se genera una clave (algo como `AIzaSy...`).
4. Restringe la clave para que no la pueda usar cualquiera con tu proyecto:
   - Clic en la clave recién creada → **Restricciones de la aplicación** → **Referentes HTTP (sitios web)** → agrega `https://TU_USUARIO.github.io/*` (y opcionalmente `http://localhost/*` para probar localmente).
   - **Restricciones de API** → **Restringir clave** → selecciona solo **Google Calendar API**.
   - Guarda.
5. Abre [index.html](index.html), busca `const CALENDAR_CONFIG = {...}` y reemplaza `'YOUR_GOOGLE_API_KEY'` con tu clave.
6. Guarda, haz commit y push. Cuando recargues la página en un día de clase, verás un texto **"Today: Day N"** bajo el encabezado, y las tarjetas de los grupos que te tocan ese día tendrán un borde de color y una etiqueta "Today".

### Si cambia tu horario de campana

El mapeo de qué grupos ves cada día (D1-D6) está en `const SCHEDULE_BY_DAY = {...}` dentro de `index.html`. Si el colegio te cambia el horario, edita esas listas con los grupos correctos por día (usa `'EL'` para Electiva) y vuelve a hacer commit/push.

Los horarios de campana (para el indicador "🕐 Period N" junto a "Today: Day N") están en `const MS_SCHEDULE` (6°-8° y Electiva) y `const HS_SCHEDULE` (9°), un poco más abajo en el mismo archivo.

## Atajo para Classroom

Cada lección tiene un botón 🏫 que copia el título + links de esa lección al portapapeles y abre Google Classroom para que pegues y publiques el borrador en segundos.

Para que te lleve directo al curso correcto de cada grupo (en vez de la portada general de Classroom), busca `const CLASSROOM_LINKS` en `index.html` y pega el link de cada grupo — entra a ese grupo en Classroom y copia lo que salga en la barra de direcciones, por ejemplo:
```js
const CLASSROOM_LINKS = {
  '6A':'https://classroom.google.com/c/NzE4MjM0NTY3ODkw',
  '6B':'',
  ...
};
```
Puedes pegar el link completo tal cual (la app le saca el ID sola) o solo el ID si lo prefieres.

## Enviar archivos a Classroom como borrador (automático)

Además del atajo 🏫 (copiar + abrir Classroom), cada lección tiene un botón **📤** que sube un archivo desde tu computador a tu Drive y crea directamente una tarea en modo **borrador** en Classroom, con el archivo adjunto — cada estudiante recibe su propia copia para trabajar. Tú entras a Classroom cuando quieras, revisas, y le das "Publicar".

Esto sí requiere que autorices el acceso una vez (a diferencia de la API key de arriba, que es de solo lectura, esto crea contenido en tu cuenta). Pasos:

1. En el mismo proyecto de Google Cloud que usaste para el calendario ([console.cloud.google.com](https://console.cloud.google.com)):
   - Busca y habilita **"Google Classroom API"**.
   - Busca y habilita **"Google Drive API"**.
2. Ve a **APIs y servicios → Pantalla de consentimiento de OAuth**:
   - Tipo de usuario: **Externo**.
   - Completa nombre de la app, tu correo, etc.
   - En "Scopes" (permisos), agrega:
     - `.../auth/classroom.coursework.students`
     - `.../auth/drive.file`
   - En "Usuarios de prueba" (Test users), agrega tu propio correo institucional/Google.
   - Guarda — la app queda en modo **"Prueba"**, lo cual es normal y suficiente para uso personal (evita el proceso de verificación de Google, que toma semanas y no aplica aquí).
3. Ve a **APIs y servicios → Credenciales → Crear credenciales → ID de cliente de OAuth**:
   - Tipo de aplicación: **Aplicación web**.
   - En "Orígenes autorizados de JavaScript" agrega `https://TU_USUARIO.github.io`.
   - Crea, y copia el **Client ID** (termina en `.apps.googleusercontent.com`).
4. Abre [index.html](index.html), busca `const OAUTH_CONFIG = {...}` y reemplaza `'YOUR_OAUTH_CLIENT_ID.apps.googleusercontent.com'` con tu Client ID.
5. Guarda, haz commit y push.

**La primera vez que uses el botón 📤**, Google te va a mostrar una pantalla de advertencia tipo "Esta app no está verificada". Es normal — haz clic en **"Avanzado"** → **"Ir a [nombre de tu app] (no seguro)"** y acepta los permisos. Solo te lo pedirá ocasionalmente, no cada vez.

## Número de salón por grupo

Cada tarjeta de grupo (y el encabezado al entrarle) muestra su salón con 🚪. Si cambian los salones de un año a otro, edita `const ROOM_NUMBERS` en `index.html`.

### Nota de seguridad sobre Firestore en "modo de prueba"

El modo de prueba deja la base de datos abierta a cualquiera que tenga tu `firebaseConfig` (que queda visible en el código fuente de la página, ya que es una app 100% del lado del cliente). Para un planeador de clases sin datos sensibles esto suele ser aceptable, pero si te preocupa:

- En Firestore → **Reglas**, cambia las reglas para restringir escritura, por ejemplo limitando por fecha de expiración (el modo de prueba ya expira en 30 días por defecto — tendrás que renovarlo o poner reglas permanentes como esta para seguir funcionando):
  ```
  rules_version = '2';
  service cloud.firestore {
    match /databases/{database}/documents {
      match /{document=**} {
        allow read, write: if true;
      }
    }
  }
  ```
- Para más seguridad, se necesitaría agregar autenticación (Firebase Auth), lo cual implica más cambios en el código. Si más adelante quieres eso, dímelo.
