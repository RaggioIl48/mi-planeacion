# Mi planeación

Planeador de lecciones por grado y grupo (6°-9°, A/B/C), con progreso por grupo, enlaces a diapositivas/worksheets y notas.

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
