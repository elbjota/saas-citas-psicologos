# SaaS de citas para psicólogos independientes

**Agenda, pacientes, cobros y reportes, cuidando la privacidad de datos de salud mental · 2026**

Es una aplicación web por suscripción (SaaS) que usan varios consultorios a la vez, cada uno con sus datos separados:
- Cada psicóloga tiene una **página pública que ella misma edita**.
- Sus pacientes **reservan cita sin crear una cuenta**.
- Ella maneja su agenda, pacientes, pagos con comprobante, mensajes de WhatsApp y reportes desde el celular.

![Next.js](https://img.shields.io/badge/Next.js-16-000000?logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17%20%2B%20RLS-4169E1?logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-autoalojado-3FCF8E?logo=supabase&logoColor=white)
![Playwright](https://img.shields.io/badge/E2E-436%20Playwright-2EAD33?logo=playwright&logoColor=white)
![Estado](https://img.shields.io/badge/estado-ensayo%20de%20producci%C3%B3n%20OK-1F3D33)

📄 **[Caso de estudio completo (PDF)](docs/caso-de-estudio.pdf)**

![Perfil público](img/01-perfil-portada.png)

---

## El problema

Muchas psicólogas independientes llevan su consulta con WhatsApp, cuadernos y memoria:
- Las citas y los cambios de horario se conversan por chat.
- Los comprobantes de pago llegan sueltos.
- No queda claro quién pagó y quién debe.
- Los datos de los pacientes no tienen ninguna protección.

## La solución

| Para | Qué hace |
|---|---|
| **El paciente** | Ve la página de la profesional (enfoque, temas, servicios, formación, ubicación y preguntas frecuentes) y reserva en 4 pasos **sin crear cuenta** |
| **La psicóloga** | Acepta o rechaza solicitudes y ve su agenda del día y de la semana. También puede programar la siguiente cita, pasar una cita a virtual o proponer otro horario con un enlace que el paciente usa una sola vez |
| **Los cobros** | Pago en efectivo o con comprobante, que ella aprueba o rechaza. Ve qué falta cobrar y qué comprobantes faltan revisar |
| **La comunicación** | Mensajes de WhatsApp ya escritos, con plantillas que ella puede cambiar, enviados desde su propio número |
| **Los números** | Reportes por periodo: ingresos, citas atendidas, inasistencias y cobros por método de pago |
| **La plataforma** | Un panel de administrador general para ver consultorios y planes, revisar el historial de acciones y suspender el acceso. Estas reglas las aplica la propia base de datos |

## Capturas

| Perfil en el celular | Reservar sin cuenta | Agenda en el celular |
|---|---|---|
| ![](img/03-perfil-movil.png) | ![](img/04-reservar.png) | ![](img/18-agenda-movil.png) |

![Agenda del día](img/10-agenda-dia.png)

| Pacientes | Pagos |
|---|---|
| ![Pacientes](img/12-pacientes.png) | ![Pagos](img/13-pagos.png) |
| **Reportes** | **Plantillas de WhatsApp** |
| ![Reportes](img/14-reportes.png) | ![Mensajes](img/17-mensajes-whatsapp.png) |

## Cómo está armado

![Arquitectura](img/arquitectura.png)

**Decisiones principales:**
- **Datos separados por consultorio, con doble control:** la aplicación filtra los datos de cada consultorio y, además, la base de datos PostgreSQL lo vuelve a revisar con RLS (reglas dentro de la base de datos que hacen que cada consultorio vea solo lo suyo). Lo comprobé con 17 pruebas en una base de datos real.
- **Guardar solo lo necesario (minimización de datos):** no se guardan historia clínica, notas ni diagnósticos. Solo lo que hace falta para agendar y cobrar. Va en la línea de la Ley 29733 de protección de datos personales.
- **Comprobantes privados:** al subir un comprobante se revisa que el archivo sea de verdad del tipo permitido y no otro archivo disfrazado. Para verlo se usan enlaces que caducan en pocos minutos.
- **Servidor propio:** todo corre en contenedores Docker (paquetes que llevan cada programa listo para funcionar) en un servidor propio. Caddy es la única puerta de entrada desde internet y pone HTTPS automático (conexión cifrada).

## Seguridad: un fallo real que encontré y corregí

- **El fallo:** en una revisión de seguridad encontré que una función de la base de datos estaba mal configurada y podía permitir que un usuario se diera permisos de administrador (esto se llama escalada de privilegios).
- **La corrección:** lo corregí y agregué una prueba automática para que no vuelva a pasar.

Además:
- Quité a la aplicación los permisos de la base de datos que no necesitaba. Lo comprobé con **21 de 21 pruebas de ataque a permisos** que pasan.
- Los registros del sistema (logs) ocultan datos personales y contraseñas. Hay 33 pruebas que lo revisan.
- Medí que el límite de intentos de inicio de sesión era muy alto y lo dejé documentado para reforzarlo.
- Dejé un análisis de riesgos de seguridad documentado, una clasificación de qué datos son sensibles, borrado programado de datos viejos y pruebas automáticas de accesibilidad.

## Calidad

- **3116 pruebas automáticas con Vitest** (revisan funciones por separado y en conjunto) en el último registro.
- **436 pruebas E2E con Playwright:** un robot que usa la app como una persona y recorre lo que hacen la psicóloga y el paciente.
- **87 migraciones** (cada cambio a la base de datos queda guardado por versiones) y **177 commits**.

## Estado

- ✅ **Primera versión completa y probada** en un **ensayo de producción en mi computadora**: los mismos contenedores que irán al servidor (VPS, un servidor alquilado en internet), con HTTPS, copias de seguridad (backups) y su restauración.
- ⏳ **Siguiente paso:**
  - subirlo al VPS, con dominio y certificados reales;
  - correos automáticos (confirmaciones y avisos);
  - definir la política de copias de seguridad antes de recibir datos reales.
- 🚫 **Lo dejé fuera a propósito en esta versión:** pagos en línea, recordatorios automáticos y que el paciente gestione sus propias citas.

## Mi rol

Es un proyecto propio de principio a fin:
- Definí qué necesitaba el producto junto con una psicóloga piloto.
- Diseñé cómo se guardan los datos, cómo se conectan las partes y cómo se protege la privacidad.
- Desarrollé la aplicación con apoyo de asistentes de IA.
- Preparé las pruebas: automáticas, de ataque a permisos y E2E.
- Preparé el servidor propio con Docker.

---

> 🔒 **Repositorio de muestra:** no se publican el código fuente, los datos reales ni las claves del servidor.
>
> Las capturas son de una copia local con la cuenta de demostración «Ps. Mariana Salazar Rivas». La persona, los pacientes y los teléfonos son ficticios; la foto es de Unsplash (Christina @ wocintechchat).

**Bryan Jeremy Espinoza Herrera** · BJOTA Systems · Perú
