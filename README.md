# Delivery Gestor

Plataforma para administrar motorizados, asignar deliveries, supervisar su ejecucion y controlar los movimientos financieros de la operacion.

## Propuesta

La arquitectura inicial esta documentada en [arquitectura_plataforma_delivery.md](arquitectura_plataforma_delivery.md). El stack propuesto es TypeScript, React Native con Expo, NestJS, Prisma, PostgreSQL/PostGIS y Supabase; Next.js se incorpora para el panel web cuando corresponda.

## Estado

Monorepo inicial con NestJS, Expo, Next.js, Prisma 7 y Supabase CLI. La implementacion del MVP seguira en commits separados.

## Desarrollo local

Requiere Node.js 22 o superior (menor que 25), pnpm 12 y Git. Android Studio es necesario para usar el emulador Android.

Para la base local, instala WSL2 desde PowerShell como administrador con `wsl --install`, reinicia Windows y abre Docker Desktop. Luego, desde la raiz del repositorio:

```powershell
pnpm install
Copy-Item apps/api/.env.example apps/api/.env
pnpm db:start
pnpm --filter @delivery/api exec prisma migrate dev --name init
```

Comandos de desarrollo: `pnpm dev:api`, `pnpm dev:mobile` y `pnpm dev:admin`. Para detener Supabase local: `pnpm db:stop`.