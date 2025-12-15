# Manual de Desarrollador - Worldbuilding Lab Backend

## Índice

1. [Descripción General](#1-descripción-general)
2. [Requisitos Previos](#2-requisitos-previos)
3. [Configuración del Proyecto](#3-configuración-del-proyecto)
4. [Levantar el Proyecto](#4-levantar-el-proyecto)
5. [Comandos Útiles](#5-comandos-útiles)
6. [Estructura del Proyecto](#6-estructura-del-proyecto)
7. [Módulos Principales](#7-módulos-principales)
8. [Arquitectura del Sistema](#8-arquitectura-del-sistema)
9. [Base de Datos](#9-base-de-datos)
10. [Sistema de Roles](#10-sistema-de-roles)
11. [Manejo de Errores](#11-manejo-de-errores)
12. [CI/CD](#12-cicd)

---

## 1. Descripción General

**Austral Worldbuilding Lab** es una plataforma de worldbuilding colaborativo en tiempo real. El backend es una API REST construida con NestJS que gestiona:

- **Organizaciones** y **Proyectos** de worldbuilding
- **Mandalas**: representaciones visuales de personajes/contextos 
- **Generación de contenido con IA** (Gemini)
- **Colaboración**: invitaciones y roles por proyecto/organización

### Stack Principal

| Tecnología | Uso |
|------------|-----|
| NestJS 11 | Framework backend |
| PostgreSQL | Base de datos principal |
| Prisma | ORM |
| Redis | Cache y colas (BullMQ) |
| Firebase | Autenticación y Firestore |
| Azure Blob Storage | Archivos |
| Google Gemini | IA generativa |

---

## 2. Requisitos Previos

Antes de empezar, asegurate de tener instalado:

- **Node.js** v22 o superior
- **npm** (viene con Node)
- **Docker** y **Docker Compose**
- **Git**

Verificar instalación:
```bash
node --version    # v22.x.x
npm --version     # 10.x.x
docker --version  # Docker version 24.x.x o superior
```

---

## 3. Configuración del Proyecto

### 3.1 Clonar el repositorio

```bash
git clone https://github.com/austral-worldbuilding-lab/back.git
cd back
```

### 3.2 Instalar dependencias

```bash
npm install
```

Esto también configura Husky para los git hooks automáticamente.

### 3.3 Configurar variables de entorno

Crear archivo `.env` en la root del proyecto. Esto lo podemos hacer copiando el .env.example y poniendo los valores de las variables.

```bash
cp .env.example .env
```

---

## 4. Levantar el Proyecto

### 4.1 Ambiente de desarrollo

**Paso 1: Levantar servicios con Docker**

```bash
docker-compose up -d
```

Esto levanta:
- El servidor en puerto `3000`
- PostgreSQL en puerto `5432`
- Redis en puerto `6379`
- Redis UI en puerto `5540`

**Paso 2: Ejecutar migraciones**

```bash
npx prisma generate
npx prisma migrate dev
```

**Paso 3: Ejecutar seed (primera vez)**

```bash
npm run prisma:seed
```

Esto crea los roles por defecto (dueño, facilitador, worldbuilder, lector).

**Paso 4: Iniciar el servidor**

Si queremos iniciar el servidor corriendo directamente el node (sin usar docker), pero ya executamos el `docker compose up`, se puede hacer:

```bash
docker stop awbl-app  # Solo si ya hice el docker compose up
npm run start:dev
```

El servidor estará disponible en `http://localhost:3000`.

La documentación Swagger está en `http://localhost:3000/api/docs`.

### 4.2 Solo levantar (si ya está configurado)

```bash
docker-compose up -d      # DB y Redis
npm run start:dev         # Backend
```

### 4.3 Ambiente de producción

```bash
npm run build
npm run start             # Incluye migraciones automáticas
```

---

## 5. Comandos Útiles

### Desarrollo
```bash
npm run start:dev          # Servidor en modo watch
npm run start:debug        # Servidor con debugger
```

### Base de datos
```bash
npx prisma studio          # UI para ver/editar datos
npx prisma migrate dev     # Crear/aplicar migraciones en dev
npx prisma migrate deploy  # Aplicar migraciones en prod
npx prisma generate        # Regenerar cliente Prisma
npm run prisma:seed        # Ejecutar seed
```

### Testing
```bash
npm run test               # Tests unitarios
npm run test:watch         # Tests en modo watch
npm run test:cov           # Tests con cobertura
npm run test:e2e           # Tests end-to-end
```

### Código
```bash
npm run lint               # Lint + fix
npm run lint:check         # Solo verificar lint
npm run format             # Formatear con Prettier
```

### Build
```bash
npm run build              # Compilar a dist/
```

### Seguridad
```bash
npm run scan-secrets       # Escanear secretos en archivos stageados (requiere Docker)
```
> **Nota**: Este comando corre `gitleaks` automáticamente mediante un **pre-commit hook** en Husky. Si detecta algun secreto, rechaza el commit.

### Git (saltear hooks si es necesario)
```bash
git push --no-verify       # Push sin pre-push hook
git commit --no-verify     # Commit sin pre-commit hook
```

---

## 6. Estructura del Proyecto

```
back/
├── src/
│   ├── main.ts                 # Entry point
│   ├── app.module.ts           # Módulo principal
│   ├── common/                 # Código compartido
│   │   ├── exceptions/         # Excepciones custom
│   │   ├── guards/             # Guards de autorización
│   │   ├── pipes/              # Validación
│   │   └── services/           # Logger, Cache
│   ├── config/                 # Configuraciones
│   └── modules/                # Módulos de negocio
│       ├── ai/                 # Generación con IA
│       ├── auth/               # Autenticación Firebase
│       ├── files/              # Gestión de archivos
│       ├── mandala/            # Mandalas
│       ├── organization/       # Organizaciones
│       ├── project/            # Proyectos
│       ├── queue/              # Procesamiento async para enciclopedia y soluciones
│       ├── solution/           # Soluciones
│       └── user/               # Usuarios
├── prisma/
│   ├── schema.prisma           # Esquema de DB
│   ├── migrations/             # Migraciones
│   └── seed.ts                 # Datos iniciales
├── docs/                       # Documentación
├── test/                       # Tests e2e
└── docker-compose.yml          # Servicios locales
```

---

## 7. Módulos Principales

### `auth/`
Autenticación con Firebase. Verifica tokens JWT en cada request.

### `organization/`
Gestiona organizaciones. Una organización agrupa proyectos y usuarios.

### `project/`
Gestiona proyectos de worldbuilding (También llamados mundos). Incluye configuración de dimensiones, escalas, tags, y jerarquía de proyectos (timeline).

### `mandala/`
Gestiona mandalas. Tipos: CHARACTER, CONTEXT, OVERLAP, OVERLAP_SUMMARY.

### `ai/`
Integración con Google Gemini. Se usa el `Strategy Pattern` para poder implementar otras integraciones con AIs. Genera: postits, preguntas, enciclopedias, soluciones, provocaciones, imágenes.

### `queue/`
Procesamiento asíncrono con BullMQ. Usa workers "on-demand" que se apagan cuando no hay trabajo (ahorra requests a Redis).

### `files/`
Upload y gestión de archivos en Azure Blob Storage.

### `solution/`
Gestiona soluciones generadas por IA. Incluye la posibilidad de generaraction items e imágenes concretas de la solución.

### `invitation/` y `organization-invitation/`
Sistema de invitaciones para agregar usuarios a proyectos/organizaciones.

---

## 8. Arquitectura del Sistema

La documentación detallada sobre la arquitectura interna, incluyendo el flujo de generación de soluciones con IA, el uso de Redis y el sistema de Workers On-Demand, se ha movido a su propio documento:

👉 **[Ver Documentación de Arquitectura](/docs/ARCHITECTURE.md)**

---

## 9. Base de Datos

### Entidades principales

- **User**: usuarios del sistema
- **Organization**: agrupa proyectos
- **Project**: proyecto de worldbuilding (tiene configuración, dimensiones, escalas)
- **Mandala**: representación visual dentro de un proyecto
- **Role**: roles de permisos (dueño, facilitador, worldbuilder, lector)
- **UserProjectRole**: relación usuario-proyecto-rol
- **UserOrganizationRole**: relación usuario-organización-rol
- **Invitation**: invitaciones pendientes
- **Solution**: soluciones generadas
- **Provocation**: preguntas provocadoras

### Diagrama

![Diagrama de Base de Datos](/docs/images/WBL%20Local%20Docker%20DB.svg)

Ver el schema completo en `prisma/schema.prisma`.

---

## 10. Sistema de Roles

Los roles se usan tanto a nivel de **organización** como de **proyecto**.

| Rol | Nivel | ¿Qué puede hacer? |
|-----|-------|-------------------|
| `dueño` | 1 | Todo: crear, editar, eliminar proyectos, gestionar usuarios |
| `facilitador` | 2 | Crear proyectos, gestionar usuarios, generar contenido con IA |
| `worldbuilder` | 3 | Crear contenido (tags, provocaciones), generar con IA, subir archivos |
| `lector` | 4 | Solo ver contenido |

> El nivel menor = más privilegios.

Los endpoints usan guards que verifican el rol:
```typescript
@RequireProjectRoles('dueño', 'facilitador')  // Solo estos roles pueden acceder
```

---

## 11. Manejo de Errores

El proyecto usa excepciones personalizadas. Importar desde `@common/exceptions/custom-exceptions`:

| Excepción | Código | Cuándo usar |
|-----------|--------|-------------|
| `ResourceNotFoundException` | 404 | Recurso no encontrado |
| `BusinessLogicException` | 422 | Regla de negocio violada |
| `ExternalServiceException` | 502 | Error en Firebase/Azure/Gemini |
| `StateConflictException` | 409 | Estado inválido para la operación |
| `ValidationException` | 400 | Error de validación |
| `ForbiddenException` | 403 | Sin permisos |

Ejemplo:
```typescript
if (!project) {
  throw new ResourceNotFoundException('Project', projectId);
}
```

---

## 12. CI/CD

El proyecto usa **GitHub Actions** para automatizar la integración y el despliegue. Cada vez que se hace push a cualquier rama, se corre el pipeline de CI que instala dependencias, genera el cliente Prisma, corre el linter y ejecuta los tests. Si el push es a `dev` o `main`, además se construye la imagen Docker, se sube a GitHub Container Registry (ghcr.io), y se notifica a Azure Web App mediante un webhook para que haga pull de la nueva imagen y reinicie el servicio.

| Rama | Ambiente | Qué pasa |
|------|----------|----------|
| `dev` | Desarrollo | Deploy automático a ambiente dev |
| `main` | Producción | Deploy automático a ambiente prod |
| Otras | - | Solo corre CI (build, lint, tests) |

Los archivos de configuración están en `.github/workflows/`.

---

## Referencias

- [Documentación NestJS](https://docs.nestjs.com/)
- [Prisma Docs](https://www.prisma.io/docs/)
- [BullMQ](https://docs.bullmq.io/)

---

*Última actualización: Diciembre 2025*
