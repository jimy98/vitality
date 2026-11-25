# Vitality — Plataforma Web de Gestión Integral para Consultorios Nutricionales

Sistema web full-stack diseñado para centralizar la gestión operativa de consultorios nutricionales, con **segregación estricta de roles** mediante JWT:
- **Profesional (Lic. en Nutrición):** CRUD completo de pacientes, historias clínicas, evolución antropométrica, calculadora nutricional clínica y diseño/aprobación de planes alimenticios semanales.
- **Paciente:** Portal de **solo lectura** para consultar sus planes aprobados, balance de macronutrientes, exportación a PDF y seguimiento de evolución antropométrica.

---

## Capturas

![Login](docs/capturas/Captura_de_pantalla_Login.png)
![Pacientes](docs/capturas/Captura_de_pantalla_Pacientes.png)
![Panel Principal](docs/capturas/Captura_de_pantalla_Panel_Principal.png)
![Captura PDF](docs/capturas/Captura_de_pantalla_PDF_Plan.png)
![Planes Alimenticios](docs/capturas/Captura_de_pantalla_Planes_Alimenticios.png)

## Funcionalidades Principales

- **Autenticación y roles con JWT + bcryptjs** (soporte multi-profesional: cada nutricionista gestiona su propio consultorio aislado).
- **Gestión de Pacientes e Historia Clínica:** antecedentes médicos, restricciones alimentarias y generación de acceso para el paciente.
- **Seguimiento Antropométrico:** registro histórico de peso, altura, IMC automático, % de grasa y masa muscular, con gráficos interactivos y **deltas acumulados de progreso**.
- **Calculadora Nutricional Clínica:** cálculo de IMC, TMB (*Mifflin-St Jeor* y *Harris-Benedict*), GET y distribución de macronutrientes con creación directa de planes.
- **Editor de Planes Alimenticios Semanales:**
  - Comidas organizadas por día (`Lunes`–`Domingo`) y momento (`Desayuno` a `Colación`).
  - Control de estado (`Borrador`, `Aprobado`, `Archivado`), **duplicación de planes** y **copia de días**.
  - **Plantillas rápidas de comidas** precargables en 1 clic.
  - **Alertas preventivas** de desvío calórico (>±10%) y cruce con restricciones alimentarias.
  - **Exportación a PDF** con diseño visual de lámina semanal y gráfico de torta de macronutrientes.
  - Aviso rápido al paciente por **WhatsApp Web**.
- **Personalización visual:** 3 modos de apariencia (**Verde**, **Blanco** y **Negro / Dark Mode**) con persistencia en `localStorage`.
- **Documentación interactiva de la API:** Swagger / OpenAPI en `/api-docs`.

---

## Arquitectura y Stack Tecnológico

| Capa | Tecnologías |
| :--- | :--- |
| **Frontend** | React 18, Vite 6, React Router v6, Lucide React, CSS Variables |
| **Backend** | Node.js (ESM), Express 4, JWT (`jsonwebtoken`), `bcryptjs`, `swagger-ui-express` |
| **Base de Datos** | MySQL (`mysql2/promise`) — Esquema relacional de 5 tablas (`Profesionales`, `Pacientes`, `Registros_Antropometricos`, `Planes_Alimenticios`, `Detalles_Comidas`) |

---

## Base de Datos Relacional (MySQL — 5 Tablas)

El esquema definido en [`database/schema.sql`](database/schema.sql) utiliza motor `InnoDB` (`utf8mb4_unicode_ci`) con integridad referencial estricta (`ON DELETE CASCADE ON UPDATE CASCADE`) e índices sobre las claves foráneas más consultadas:

| Tabla | Clave Primaria (PK) | Claves Foráneas (FK) | Descripción y Relaciones |
| :--- | :--- | :--- | :--- |
| **`Profesionales`** | `id_profesional` | — | Almacena los nutricionistas matriculados (`matricula` y `correo_electronico` únicos, contraseña con hash `bcryptjs`). Se relaciona **1:N** con `Pacientes` y **1:N** con `Planes_Alimenticios`. |
| **`Pacientes`** | `id_paciente` | `id_profesional` → `Profesionales(id_profesional)` | Guarda datos personales, credenciales de acceso al portal, nivel de actividad, antecedentes médicos y restricciones alimentarias. Pertenece a un único profesional (**N:1**) y tiene **1:N** registros antropométricos y **1:N** planes alimenticios. |
| **`Registros_Antropometricos`** | `id_registro` | `id_paciente` → `Pacientes(id_paciente)` | Historial de controles por fecha (`peso_kg`, `altura_cm`, `porcentaje_grasa`, `masa_muscular_kg`). Permite calcular dinámicamente el IMC y graficar la evolución de cada paciente (**N:1**). |
| **`Planes_Alimenticios`** | `id_plan` | `id_paciente` → `Pacientes(id_paciente)`<br>`id_profesional` → `Profesionales(id_profesional)` | Cabecera del plan nutricional (`fecha_creacion`, `objetivo_calorico`, `estado`: `Borrador` \| `Aprobado` \| `Archivado`, `observaciones`). Vincula paciente y profesional, y agrupa **1:N** comidas en `Detalles_Comidas`. El portal del paciente solo expone planes `Aprobado`. |
| **`Detalles_Comidas`** | `id_detalle` | `id_plan` → `Planes_Alimenticios(id_plan)` | Comidas asignadas por `dia_semana` (`Lunes`–`Domingo`) y `momento_dia` (`Desayuno`–`Colacion`), con descripción de alimentos y gramos de proteína, carbohidratos y grasas (**N:1** con `Planes_Alimenticios`). |

> **Pruebas funcionales:** El detalle de casos verificados (autenticación, rechazo `403`, cálculos IMC/TMB/GET, duplicación, copia de días y alertas de restricciones) se encuentra en [`docs/pruebas-funcionales.md`](docs/pruebas-funcionales.md).

---

## Instalación y Puesta en Marcha Local

### Requisitos previos
- **Node.js** 18 o superior
- **MySQL** 8.0 (o XAMPP / MariaDB) corriendo en `localhost:3306`

### 1. Clonar el repositorio
```bash
git clone https://github.com/jimy98/vitality.git
cd vitality
```

### 2. Configurar el Backend y la Base de Datos
```bash
cd backend
cp .env.example .env
```
Editá `backend/.env` con tus credenciales de MySQL (`DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`) y luego ejecutá:
```bash
npm install
npm run init-db
npm start
```
- API disponible en: `http://localhost:4000`
- Documentación Swagger: `http://localhost:4000/api-docs`

### 3. Configurar y levantar el Frontend
En otra terminal:
```bash
cd frontend
npm install
npm run dev
```
- Aplicación web disponible en: `http://localhost:5173`

### Credenciales de demostración (cargadas por `npm run init-db`)
- **Profesional:** `nutricionista@vitality.com` / `vitality123`
- **Paciente:** `lucas@vitality.com` / `paciente123`

---

## Equipo de Desarrollo

| Integrante | Usuario de GitHub | Rol |
| :--- | :--- | :--- |
| **Mercado Cáceres José Gustavo** | [@jimy98](https://github.com/jimy98) | Desarrollo Full-Stack & Arquitectura |
| **Ruiz Santino Santiago** | [@Santino07-byte](https://github.com/Santino07-byte) | Frontend UI/UX & Documentación Visual |
| **Ignacio Agustín Mercado** | [@NachoEmme7](https://github.com/NachoEmme7) | Base de Datos, Pruebas & Documentación API |
