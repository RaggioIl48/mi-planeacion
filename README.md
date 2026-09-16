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
