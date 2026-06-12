# Arquitectura — Calendario de Permisos SLEP Petorca

## Diagrama de flujo general

```
┌─────────────────────────────────────────────────────────────┐
│                        USUARIO                              │
│              (Navegador, cualquier dispositivo)             │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTPS
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    GITHUB PAGES                             │
│         https://finanzas-slep-petorca.github.io/            │
│                  calendario-permisos/                       │
│                                                             │
│  Sirve archivos estáticos desde rama main:                  │
│    • index.html  (toda la lógica del cliente)               │
│    • Tailwind CSS via CDN                                   │
│    • Firebase SDK via CDN (gstatic.com)                     │
└──────────────┬──────────────────────────┬───────────────────┘
               │                          │
               ▼                          ▼
┌──────────────────────┐    ┌─────────────────────────────────┐
│  FIREBASE AUTH       │    │       CLOUD FIRESTORE           │
│                      │    │                                 │
│  Email Link          │    │  Colección: events              │
│  (passwordless)      │    │  ┌─────────────────────────┐   │
│                      │    │  │ Documento (auto-ID)      │   │
│  1. Cliente solicita │    │  │ ─────────────────────── │   │
│     envío de link    │    │  │ date: "YYYY-MM-DD"       │   │
│  2. Usuario hace     │    │  │ memberId: string         │   │
│     clic en email    │    │  │ type: string             │   │
│  3. Firebase verifica│    │  │ replacementId: string|null│  │
│     y emite token    │    │  │ notes: string            │   │
│  4. Cliente valida   │    │  │ createdBy: string (email)│   │
│     contra whitelist │    │  │ createdAt: timestamp     │   │
│                      │    │  │ lastModifiedBy: string   │   │
│  Whitelist (8 emails)│    │  │ lastModifiedAt: timestamp│   │
│  controlada en:      │    │  └─────────────────────────┘   │
│    • index.html      │    │                                 │
│    • firestore.rules │    │  Seguridad:                     │
│                      │    │  • Solo lectura/escritura       │
│                      │    │    si email autenticado         │
│                      │    │    está en whitelist            │
│                      │    │  • Validación de estructura     │
│                      │    │    en creates                   │
│                      │    │                                 │
│                      │    │  Sync:                          │
│                      │    │  • onSnapshot → tiempo real     │
│                      │    │  • IndexedDB → offline cache    │
└──────────────────────┘    └─────────────────────────────────┘
```

## Flujo de autenticación (Email Link)

```
Usuario ingresa email
        │
        ▼
¿Está en ALLOWED_EMAILS?
        │
   No ──┴──► Mensaje de error en pantalla (no se envía nada)
        │
       Sí
        │
        ▼
sendSignInLinkToEmail()
        │
        ▼
Firebase envía email con enlace mágico
        │
        ▼
Usuario hace clic en el enlace desde su correo
        │
        ▼
index.html carga con parámetros en la URL
        │
        ▼
isSignInWithEmailLink() → true
        │
        ▼
signInWithEmailLink() → token JWT emitido
        │
        ▼
onAuthStateChanged() detecta sesión activa
        │
        ▼
¿Email está en ALLOWED_EMAILS?
        │
   No ──┴──► signOut() + pantalla "Acceso denegado"
        │
       Sí
        │
        ▼
Mostrar calendario + iniciar onSnapshot()
```

## Flujo de sincronización en tiempo real

```
Cualquier pestaña/usuario       Firestore Cloud
        │                              │
        │──── onSnapshot suscrito ────►│
        │                              │
        │  Usuario A crea evento       │
        │──── addDoc() ───────────────►│
        │                              │──► Notifica a TODOS
        │◄─── snapshot actualizado ────│    los suscriptores
        │                              │
        │  Estado local se actualiza   │
        │  renderAll() se ejecuta      │
        │                              │
        │  Usuario B ve el cambio      │
        │  en menos de 2 segundos      │
```

## Modelo de datos Firestore

### Colección: `events`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `date` | string | Fecha en formato `YYYY-MM-DD` |
| `memberId` | string | ID del integrante (`wilson`, `juanpablo`, etc.) |
| `type` | string | `feriado` \| `permiso` \| `licencia` \| `compensacion` \| `otro` |
| `replacementId` | string \| null | ID del reemplazo asignado, o null |
| `notes` | string | Notas adicionales (puede estar vacío) |
| `createdBy` | string | Email del usuario que creó el registro |
| `createdAt` | timestamp | Fecha/hora de creación (serverTimestamp) |
| `lastModifiedBy` | string | Email del último usuario que modificó |
| `lastModifiedAt` | timestamp | Fecha/hora de última modificación |

### IDs de integrantes del equipo

| memberId | Nombre completo |
|----------|----------------|
| `wilson` | Wilson Rojas |
| `juanpablo` | Juan Pablo Bustamante |
| `benedicto` | Benedicto Fessia |
| `sebastian` | Sebastian Olguin |
| `fernando` | Fernando Saavedra |
| `yessenia` | Yessenia Vilches |
| `catalina` | Catalina Brito |

## Stack tecnológico

| Componente | Tecnología | Versión |
|-----------|-----------|---------|
| Frontend | HTML + JavaScript vanilla | — |
| Estilos | Tailwind CSS | CDN |
| Base de datos | Cloud Firestore | SDK v10.12.0 |
| Autenticación | Firebase Auth | SDK v10.12.0 |
| Hosting | GitHub Pages | rama `main` |
| Offline | IndexedDB (via Firebase) | — |

No se utiliza ningún framework de frontend (React, Vue, etc.) ni paso de compilación (webpack, vite, etc.). Todo el código es HTML/JS estático servido directamente por GitHub Pages.
