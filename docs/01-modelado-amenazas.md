
# Parte A — Modelado de amenazas de LabReserva

## 1. Objetivo

Identificar los componentes, flujos de datos, fronteras de confianza y posibles amenazas de seguridad de LabReserva mediante un Diagrama de Flujo de Datos (DFD) y la metodología STRIDE.

El análisis se realiza inicialmente a partir del README y de la interfaz de la aplicación, sin inspeccionar el código fuente, siguiendo las instrucciones de la actividad.

## 2. Descripción del sistema

LabReserva es una aplicación web que permite gestionar reservas de laboratorios de una institución educativa.

El sistema contempla tres roles:

- **Estudiante:** puede consultar laboratorios, publicar comentarios, crear y cancelar sus propias reservas y editar su perfil.
- **Profesor:** tiene las capacidades del estudiante y puede consultar el directorio de usuarios.
- **Administrador:** puede realizar las operaciones anteriores y funciones administrativas, como crear laboratorios, consultar todas las reservas y eliminar usuarios.

La aplicación funciona mediante una API accesible por HTTP y utiliza una base de datos SQLite en memoria.

### 2.1 Funcionalidades observadas

Durante la exploración de la aplicación con la cuenta de estudiante se identificaron:

1. Inicio de sesión con correo electrónico y contraseña.
2. Consulta del listado de laboratorios.
3. Búsqueda de laboratorios por nombre o edificio.
4. Consulta de detalles de un laboratorio.
5. Visualización y publicación de comentarios.
6. Consulta de las reservas del estudiante.
7. Cierre de sesión.

El README también documenta funciones como registro de usuarios, modificación del perfil, creación y cancelación de reservas, descarga de manuales y operaciones administrativas.

Estas funcionalidades forman parte del alcance del modelado aunque no todas aparezcan en la interfaz principal.

## 3. Diagrama de Flujo de Datos (DFD)

El DFD representa las interacciones entre los usuarios, la aplicación, el almacenamiento de información y la configuración del sistema.

Se utiliza un modelo de alto nivel para identificar los principales puntos donde pueden presentarse amenazas.

El Diagrama de Flujo de Datos se encuentra documentado en:

[Ver DFD de LabReserva](03-dfd-labreserva.md)

### 3.1 Entidades externas

| ID | Entidad | Descripción |
|---|---|---|
| E1 | Estudiante | Usuario que consulta laboratorios, publica comentarios y gestiona sus reservas. |
| E2 | Profesor | Usuario que también puede consultar el directorio de usuarios. |
| E3 | Administrador | Usuario que realiza operaciones de administración del sistema. |

### 3.2 Procesos

| ID | Proceso | Responsabilidad |
|---|---|---|
| P1 | Autenticación | Recibir credenciales, validar la identidad y emitir tokens. |
| P2 | Gestión de usuarios | Consultar y actualizar información de usuarios. |
| P3 | Gestión de laboratorios | Listar, buscar, consultar y administrar laboratorios. |
| P4 | Gestión de comentarios | Consultar y registrar comentarios asociados a laboratorios. |
| P5 | Gestión de reservas | Crear, consultar y cancelar reservas. |
| P6 | Gestión de archivos | Permitir la consulta o descarga de manuales. |

Estos procesos son una representación lógica de las responsabilidades de la API, no una afirmación sobre la implementación interna de cada módulo.

### 3.3 Almacenes de datos

| ID | Almacén | Información |
|---|---|---|
| D1 | Base de datos SQLite | Usuarios, roles, laboratorios, reservas y comentarios. |
| D2 | Archivos de manuales | Documentos asociados a laboratorios. |
| D3 | Configuración del servidor | Parámetros y secretos utilizados por la aplicación. |

La base de datos SQLite funciona en memoria, por lo que sus datos regresan al estado inicial cuando se reinicia el servidor.

La configuración se representa como una fuente interna de información sensible, diferenciada de la base de datos.

### 3.4 Flujos de datos

| ID | Origen | Destino | Datos intercambiados |
|---|---|---|---|
| F1 | Usuario | API | Credenciales para autenticación. |
| F2 | API | Usuario | Token y datos de la cuenta. |
| F3 | Usuario | API | Solicitudes de consulta, búsqueda y operaciones. |
| F4 | API | SQLite | Consultas y modificaciones de registros. |
| F5 | SQLite | API | Información de usuarios, laboratorios y reservas. |
| F6 | Usuario | API | Contenido de comentarios. |
| F7 | API | Usuario | Resultados, comentarios y reservas. |
| F8 | API | Archivos | Solicitudes de lectura de manuales. |
| F9 | Configuración | API | Parámetros y secretos del servidor. |

## 4. Fronteras de confianza

Las fronteras de confianza representan los puntos donde los datos pasan entre componentes con diferentes niveles de confianza.

### FC1. Cliente y servidor

El navegador o cliente HTTP envía solicitudes a la API.

Los datos recibidos, como credenciales, identificadores y comentarios, deben considerarse no confiables.

Posibles amenazas:
- Manipulación de solicitudes.
- Suplantación de identidad.
- Inyección de contenido.
- Acceso no autorizado.

### FC2. Aplicación y almacenamiento

La API realiza operaciones sobre la base de datos SQLite y los archivos del sistema.

Aunque SQLite se ejecuta dentro del entorno del servidor, el acceso a los datos debe respetar las reglas de autorización y validación.

Posibles amenazas:
- Consultas inseguras.
- Acceso a información de otros usuarios.
- Modificación de registros sin autorización.
- Lectura de archivos no permitidos.

### FC3. Aplicación y configuración

La aplicación utiliza parámetros y secretos internos para operar.

Esta información debe mantenerse fuera del alcance de usuarios no autorizados.

Posibles amenazas:
- Exposición de secretos.
- Configuración insegura.
- Uso indebido de credenciales o claves.

## 5. Identificación de amenazas mediante STRIDE

STRIDE permite analizar seis categorías de amenazas:

- **S — Spoofing:** suplantación de identidad.
- **T — Tampering:** manipulación de información.
- **R — Repudiation:** repudio de acciones.
- **I — Information Disclosure:** divulgación de información.
- **D — Denial of Service:** denegación de servicio.
- **E — Elevation of Privilege:** elevación de privilegios.

### 5.1 Hipótesis identificadas

| ID | Categoría | Componente | Hipótesis |
|---|---|---|---|
| H01 | S | P1 Autenticación | Un atacante podría obtener acceso a una cuenta mediante credenciales comprometidas o fallas de autenticación. |
| H02 | T | P5 Reservas | Un usuario podría manipular una solicitud para alterar datos de una reserva que no debería modificar. |
| H03 | R | P5 Reservas | Un usuario podría negar haber creado o cancelado una reserva si no existen registros de auditoría suficientes. |
| H04 | I | P5 Reservas / D1 SQLite | Un usuario autenticado podría consultar reservas pertenecientes a otro usuario mediante un identificador. |
| H05 | D | API | Un atacante podría afectar la disponibilidad enviando solicitudes excesivas o contenido demasiado grande. |
| H06 | E | Operaciones administrativas | Un estudiante podría intentar ejecutar funciones reservadas a administradores. |
| H07 | I | API | Los mensajes de error podrían revelar rutas internas, consultas o información sensible. |
| H08 | T | P2 Usuarios | Un usuario podría modificar campos protegidos de su perfil, como el rol, mediante una petición alterada. |
| H09 | T | P4 Comentarios | Un atacante podría introducir contenido HTML o JavaScript que posteriormente se ejecute en el navegador de otro usuario. |
| H10 | S | P4 Comentarios | Un estudiante podría intentar atribuir un comentario a otro usuario mediante la manipulación de identificadores. |

### 5.2 Riesgos principales

Las amenazas relacionadas con el control de acceso, la modificación de información y la exposición de datos merecen especial atención.

Se consideran prioritarias las hipótesis relacionadas con:

- Acceso a reservas de otros usuarios.
- Ejecución de operaciones administrativas.
- Modificación de roles o datos protegidos.
- Inyección de contenido en comentarios.
- Exposición de información sensible.

Esta priorización es preliminar. La severidad definitiva se determinará durante la Parte B de acuerdo con el impacto y la evidencia obtenida.

## 6. Conclusión de la Parte A

El modelado permitió identificar los principales componentes y flujos de información de LabReserva, así como las fronteras donde pueden presentarse amenazas de seguridad.

Se formularon diez hipótesis STRIDE relacionadas con autenticación, autorización, manipulación de información, disponibilidad y divulgación de datos.

Estas hipótesis no representan vulnerabilidades confirmadas. Su existencia se evaluará mediante pruebas controladas durante la Parte B, utilizando exclusivamente la instancia local de LabReserva.

La siguiente etapa consistirá en contrastar las hipótesis con el comportamiento real de la aplicación, identificar sus causas y documentar posibles correcciones.
