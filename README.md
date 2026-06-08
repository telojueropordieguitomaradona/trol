# 🐾 Veterinaria Patitas Sanas — DB Docs

> Sistema de Gestión Clínica Veterinaria · v1.0 · Única sucursal: Col. Emiliano Zapata, Los Reyes La Paz

---

## 1. Problemática y Requerimientos

La clínica del **Dr. Alejandro Ortega López** operaba de forma manual (libretas y hojas de cálculo), causando pérdida de historiales, desorganización de la agenda y riesgos en emergencias. La solución es un **esquema relacional normalizado** que digitaliza tutores, mascotas, citas, historial clínico, vacunación/desparasitación y pagos.

**Requerimientos clave**

| Módulo | Puntos principales |
|---|---|
| **Acceso** | Perfiles `veterinario` y `recepcionista`; una sola sucursal; catálogo de servicios |
| **Tutores** | Nombre, dirección, correo, teléfono(s), fecha de registro; relación 1:N con mascotas |
| **Mascotas** | ID único, nombre, especie (`CHECK`: reptil/felino/canino/ave), raza, sexo, edad, carnet |
| **Citas** | Fecha, hora, motivo, monto, veterinario asignado; admite walk-in |
| **Agenda** | Tabla ternaria: `cita` + `recepcionista` + `tutor` |
| **Servicios** | 1:1 con cita; tipo (`CHECK`: consulta/cirugía/rayos x/ultrasonido/cremación) + motivo |
| **Vacunación / Desparasitación** | Historial por mascota con fechas de aplicación y próxima dosis |
| **Pagos** | Monto, fecha, método (efectivo/tarjeta/transferencia); campo `id_uk_transaccion` para facturación futura |

---

## 2. Diseño Conceptual

### Entidades

| Tabla | Rol |
|---|---|
| `empleado` | Supertipo ISA; atributos compartidos: `id_empleado`, `nombre`, `telefono`, `rfc` |
| `veterinario` | Subtipo de `empleado`; añade `cedula_profesional` y `especialidad` |
| `recepcionista` | Subtipo de `empleado`; PK propia `id_caja` + FK `id_empleado` (UNIQUE NOT NULL) |
| `tutor` | Responsable legal de las mascotas |
| `mascota` | Paciente central del sistema |
| `tiene` | Asociativa N:M entre `tutor` y `mascota` |
| `cita` | Evento programado; FK a `veterinario` |
| `agenda` | Asociativa ternaria: `cita` + `recepcionista` + `tutor` |
| `servicio` | Acto médico (1:1 con `cita`, N:1 con `mascota`) |
| `vacuna` | Débil por existencia; depende de `mascota` |
| `desparasitacion` | Débil por existencia; depende de `mascota` |

### Relaciones y cardinalidades

| Relación | Tipo | Cardinalidad |
|---|---|---|
| `empleado` → `veterinario` / `recepcionista` | ISA | (0,1):(1,1) |
| `tutor` — `mascota` (vía `tiene`) | N:M | (0,N):(1,1) |
| `veterinario` → `cita` | 1:N | (1,1):(0,N) |
| `cita` → `servicio` | 1:1 | (1,1):(0,1) |
| `mascota` → `servicio` / `vacuna` / `desparasitacion` | 1:N | (1,1):(0,N) |
| `cita` + `recepcionista` + `tutor` → `agenda` | Ternaria N:M:M | (0,N):(0,N):(0,N) |

### Jerarquía ISA

```
empleado  (supertipo)
   ├── veterinario   — PK/FK: id_empleado; atributos: cedula_profesional, especialidad
   └── recepcionista — PK: id_caja; FK UNIQUE NOT NULL: id_empleado
```

Especialización **disjunta** y **parcial**. Todas las FK usan `DEFERRABLE INITIALLY IMMEDIATE` para permitir inserciones ordenadas dentro de una transacción.

### Entidades débiles

- **`vacuna` / `desparasitacion`** → dependencia de *existencia* respecto a `mascota` (FK `id_mascota` no nula).
- **`veterinario` / `recepcionista`** → dependencia de *identidad* respecto a `empleado` (PK heredada).
- **`servicio`** → dependencia de existencia respecto a `cita` (`id_cita` UNIQUE FK).
- **`tiene` / `agenda`** → tablas asociativas puras sin atributos propios.

### Diagrama EER

![EER](https://raw.githubusercontent.com/ctlu-l/pagina-web/main/EER.drawio.png)

---

## 3. Modelo Relacional

### Estrategia de transformación

| Patrón | Decisión |
|---|---|
| **Jerarquía ISA** | Tabla por subtipo; FK con `DEFERRABLE INITIALLY IMMEDIATE` |
| **N:M binaria** | Tabla `tiene` con PK compuesta (`id_cliente`, `id_mascota`) |
| **N:M ternaria** | Tabla `agenda` con PK de tres columnas |
| **1:1** | `id_cita` en `servicio` con restricción `UNIQUE` |
| **Entidades débiles** | FK obligatoria + `ALTER TABLE … ADD FOREIGN KEY … DEFERRABLE` |

### Diagrama Relacional

![Relacional](https://raw.githubusercontent.com/ctlu-l/pagina-web/main/relacional.png)

---

## 4. Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Frontend | React (componentes modulares) |
| Backend | Node.js |
| Base de datos | Supabase (PostgreSQL gestionado, API REST, Row Level Security) |
| Despliegue | GitHub Pages + Vercel |

---

🔗 **[Repositorio GitHub](https://github.com/gabrielhuav/DB-Coursework-2026-2)** · **[Demo en vivo](https://patitas-sanas.vercel.app/)**
