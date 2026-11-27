# Backend · Almacén de Equipos

Express 5 + Prisma 6 + PostgreSQL 16 (Docker). JavaScript con ESM. Probado con Node 24 o superior.

El frontend está en otro repositorio: [Proyecto-de-PrograWeb](https://github.com/cesarecheandianunz-arch/Proyecto-de-PrograWeb).

## Levantarlo por primera vez

```bash
git clone https://github.com/cesarecheandianunz-arch/ProyectoPrograWeb-Backend.git
cd ProyectoPrograWeb-Backend
cp .env.example .env        # en PowerShell: Copy-Item .env.example .env
npm install
npm run db:up               # Postgres (puerto 5435) y Mailpit (http://localhost:8026)
npm run db:push             # crea las tablas desde prisma/schema.prisma
npm run db:seed             # usuarios, almacenes, carreras, equipos y unidades de prueba
npm run dev                 # API en http://localhost:3000/api
npm test
```

Para usarlo con el frontend, deja esta API corriendo y en el repo del frontend ejecuta `npm run dev`. El `vite.config.js` del frontend reenvía `/api` al puerto 3000, así que no hace falta configurar CORS.

Usuarios de prueba (contraseña = `SEED_PASSWORD` de tu `.env`):

| Correo | Rol |
|---|---|
| rmendoza@aloe.ulima.edu.pe (y los demás estudiantes del front) | ESTUDIANTE |
| encargado.demo@ulima.edu.pe | ENCARGADO (activo) |
| encargado.pendiente@ulima.edu.pe | ENCARGADO (pendiente) |
| jefe.lab@ulima.edu.pe | JEFE |

Otros comandos: `npm run db:studio` (ver las tablas en el navegador), `npm run db:reset` (borra todo y vuelve a sembrar), `npm run db:down`.

## Estructura: una carpeta por HU

```
ProyectoPrograWeb-Backend/
  prisma/
    schema.prisma          ← UNO solo, con una sección por HU
    seed.js                ← una función seedHUx() por HU
    datos-iniciales.js     ← datos de ejemplo copiados del frontend
  src/
    app.js                 ← solo registra los routers (una línea por HU)
    compartido/cuenta.js   ← validaciones copiadas del frontend
    lib/                   ← prisma, errores, correo (compartido)
    middlewares/auth.js    ← requireAuth, requireRol (compartido)
    hu1-cuentas/           ← HU-1 (implementada)
    hu2-inventario/        ← HU-2 (en desarrollo)
    hu3-solicitudes/       ← HU-3 (rutas declaradas, responden 501)
    hu4-prestamos/         ← HU-4 (rutas declaradas, responden 501)
    hu5-devoluciones/      ← HU-5 (rutas declaradas, responden 501)
  tests/
    hu1.test.js … hu5.test.js
```

Cada HU tiene `huX.routes.js` (qué URL, qué rol) y `huX.controller.js` (qué hace). `hu1-cuentas` sirve de ejemplo.

## Convenciones de la API

- Todo bajo `/api`. JSON de entrada y de salida.
- Errores de formulario: `400 { "errores": { "campo": "mensaje" } }`, la misma forma que usa el `setErrores` del front.
- Otros errores: `{ "error": "mensaje" }` con 401 (sin sesión), 403 (rol o cuenta no permitida), 404 o 409 (duplicado).
- La sesión va en una cookie httpOnly `sesion` (JWT). En el controlador, el usuario es `req.auth.id` y su rol `req.auth.rol`. Nunca recibas el id del estudiante en el body.
- Validaciones: `src/compartido/` tiene copias de las reglas del frontend. Si una regla cambia en el frontend, actualízala también aquí.
- En los controladores lanza `new HttpError(status, cuerpo)`; `manejarErrores` arma la respuesta.

## Trabajo en equipo

1. Crea tu rama desde `main`: `git switch -c hu2-backend`.
2. Toca solo tu carpeta `src/huX-…/`, tu `tests/huX.test.js`, tu sección del schema y tu `seedHUx()`.
3. Mientras desarrollas usa `npm run db:push` (no `prisma migrate`). Las migraciones las genera una sola persona cuando el schema esté estable.
4. Antes del PR: `git pull origin main`, `npm run db:push`, `npm test`.
5. Si cambias el schema, avísalo en el PR: los demás tendrán que correr `npm run db:push` y `npx prisma generate`.
