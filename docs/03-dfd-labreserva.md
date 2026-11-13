
# Diagrama de Flujo de Datos (DFD) — LabReserva

## Parte A: Modelado de amenazas

El siguiente diagrama representa los componentes principales de LabReserva, los flujos de información y las fronteras de confianza identificadas durante el análisis inicial.

El DFD fue elaborado a partir del README y la exploración de la aplicación, sin inspeccionar el código fuente.

## Diagrama

```mermaid
flowchart TB
    E1["E1: Estudiante"]
    E2["E2: Profesor"]
    E3["E3: Administrador"]

    subgraph CLIENTE["Cliente / Navegador"]
        UI["Interfaz web"]
    end

    subgraph SERVIDOR["Servidor LabReserva"]
        P1["P1: Autenticación"]
        P2["P2: Gestión de usuarios"]
        P3["P3: Laboratorios"]
        P4["P4: Comentarios"]
        P5["P5: Reservas"]
        P6["P6: Archivos y manuales"]

        subgraph DATOS["Almacenamiento y recursos internos"]
            D1[("D1: SQLite")]
            D2[("D2: Manuales")]
            D3[("D3: Configuración y secretos")]
        end
    end

    E1 --> UI
    E2 --> UI
    E3 --> UI

    UI <-->|"FC1: Solicitudes HTTP / Respuestas"| P1
    UI <-->|"FC1: Operaciones y consultas"| P2
    UI <-->|"FC1: Búsqueda y resultados"| P3
    UI <-->|"FC1: Publicar y consultar"| P4
    UI <-->|"FC1: Gestión de reservas"| P5
    UI <-->|"FC1: Solicitud de manuales"| P6

    P1 <-->|"FC2: Usuarios y credenciales"| D1
    P2 <-->|"FC2: Usuarios y roles"| D1
    P3 <-->|"FC2: Laboratorios"| D1
    P4 <-->|"FC2: Comentarios"| D1
    P5 <-->|"FC2: Reservas"| D1
    P6 -->|"FC2: Lectura de documentos"| D2

    D3 -->|"FC3: Parámetros de configuración"| P1
```

## Elementos representados

| Identificador | Tipo | Descripción |
|---|---|---|
| E1–E3 | Entidades externas | Estudiantes, profesores y administradores. |
| P1–P6 | Procesos | Funcionalidades lógicas de LabReserva. |
| D1 | Almacén de datos | Base de datos SQLite con usuarios, laboratorios, comentarios y reservas. |
| D2 | Almacén de archivos | Manuales disponibles en el servidor. |
| D3 | Configuración | Variables y secretos utilizados internamente. |
| FC1 | Frontera de confianza | Separación entre navegador y servidor. |
| FC2 | Frontera lógica | Acceso de los procesos a datos y archivos. |
| FC3 | Frontera de confianza | Acceso a configuración y secretos internos. |

## Consideraciones de seguridad

- Las solicitudes provenientes del navegador deben considerarse no confiables.
- La autenticación no sustituye la autorización sobre cada recurso.
- Las operaciones sobre SQLite deben validar los permisos correspondientes.
- Las solicitudes de archivos deben limitarse a los documentos permitidos.
- La configuración y los secretos no deben exponerse a los usuarios.

Este modelo es preliminar y se utilizará para analizar las hipótesis STRIDE documentadas en `01-modelado-amenazas.md`.
