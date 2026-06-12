# Calendario de Permisos — Finanzas SLEP Petorca

Aplicación web colaborativa en tiempo real para gestionar permisos, feriados legales y licencias del Subdepartamento de Finanzas del Servicio Local de Educación Pública de Petorca.

---

## ¿Qué es esto?

Es un calendario compartido donde todos los integrantes del equipo pueden registrar, editar y eliminar ausencias. Los cambios son visibles para todos en tiempo real sin necesidad de recargar la página. No requiere instalación ni cuenta en servicios externos: el acceso es con el correo institucional Outlook.

---

## Acceso

**URL pública:** https://finanzas-slep-petorca.github.io/calendario-permisos/

### Cómo ingresar
1. Abre la URL en tu navegador.
2. Ingresa tu correo institucional (`nombre.apellido@sleppetorca.gob.cl`) y haz clic en **Enviar enlace de acceso**.
3. Revisa tu bandeja de entrada (incluido **Spam/Correo no deseado**) y haz clic en el enlace que llegará. El enlace expira en **1 hora**.
4. Al hacer clic en el enlace, quedarás autenticado automáticamente y verás el calendario.

### ¿No llegó el correo?
- Revisa la carpeta Spam o Correo no deseado.
- Verifica que escribiste bien el correo.
- Intenta de nuevo después de unos minutos.
- Si el problema persiste, contacta a Wilson Rojas (wilson.rojas@sleppetorca.gob.cl).

---

## Funcionalidades

- Vista de calendario mensual con navegación por los 12 meses de 2026.
- Registro de ausencias por tipo: Feriado Legal, Permiso Administrativo, Licencia Médica, Compensación, Otro.
- Asignación de reemplazo por ausencia, visible directamente en la celda del día.
- Detección automática de conflictos cuando 2 o más personas están ausentes el mismo día (celda roja con ⚠).
- Feriados nacionales de Chile 2026 precargados.
- Filtro por persona en el sidebar.
- Resumen anual de ausencias por persona.
- Panel de próximas ausencias (siguientes 30 días).
- Indicador de sincronización en tiempo real con nombre del último editor.
- Impresión del calendario mensual (horizontal).
- Impresión del listado anual tabular agrupado por mes (vertical).
- Exportar e importar respaldo en formato JSON.
- Funciona sin conexión (los cambios se sincronizan al reconectar).

---

## Gestión de usuarios

Para agregar o quitar un usuario autorizado es necesario hacer **dos cambios y un despliegue**:

### 1. Editar `index.html`
Busca el array `ALLOWED_EMAILS` y agrega o elimina el correo. Luego actualiza también el objeto `EMAIL_TO_MEMBER` con el memberId correspondiente.

### 2. Editar `firestore.rules`
Busca la función `isAllowed()` y agrega o elimina el mismo correo de la lista.

### 3. Publicar reglas en Firebase Console
Abre [Firebase Console](https://console.firebase.google.com/) → tu proyecto → **Firestore Database** → **Reglas** → pega el contenido actualizado de `firestore.rules` → **Publicar**.

### 4. Publicar cambios en GitHub Pages
```bash
git add .
git commit -m "Actualizar lista de usuarios autorizados"
git push
```
GitHub Pages actualizará el sitio en 1-2 minutos.

---

## Respaldo y restauración

### Exportar
En el sidebar, haz clic en **📥 Exportar respaldo (JSON)**. Se descargará un archivo `.json` con todos los registros actuales.

### Importar
Haz clic en **📤 Importar respaldo** y selecciona el archivo `.json`. Los registros se agregarán a los existentes en Firestore (no se borran los actuales).

### Migrar desde la versión local
1. Abre `calendario-permisos-2026.html` en tu navegador.
2. Exporta el respaldo JSON.
3. Abre la app en https://finanzas-slep-petorca.github.io/calendario-permisos/
4. Importa ese archivo JSON.

---

## Mantenimiento técnico

### Redesplegar el sitio
Cualquier cambio en los archivos del repositorio se despliega automáticamente al hacer push a `main`:
```bash
git add .
git commit -m "Descripción del cambio"
git push
```

### Ver logs y errores
- **Firebase Console** → tu proyecto → **Authentication** para ver usuarios que han iniciado sesión.
- **Firebase Console** → **Firestore Database** → **Datos** para ver los registros en tiempo real.
- **Firebase Console** → **Uso** para monitorear lecturas/escrituras.

### Activar/configurar GitHub Pages (primera vez)
1. Ve al repositorio en GitHub.
2. **Settings** → **Pages** → **Source**: Deploy from branch → rama `main`, carpeta `(root)` → **Save**.
3. Espera 1-2 minutos y verifica en la URL pública.

---

## Arquitectura

```
Usuario (navegador)
    │
    ▼
GitHub Pages (sirve index.html estático)
    │
    ├──► Firebase Authentication
    │        Email Link (passwordless)
    │        Validación contra whitelist en cliente + reglas Firestore
    │
    └──► Cloud Firestore
             Colección: events
             Sincronización en tiempo real via onSnapshot
             Persistencia offline via IndexedDB
```

Ver detalle en `docs/arquitectura.md`.

---

## Soporte

**Wilson Rojas Abarca**
wilson.rojas@sleppetorca.gob.cl
Subdepartamento de Finanzas — SLEP Petorca
