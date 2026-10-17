# SaaS de citas para psicólogos independientes

**Agenda, pacientes, cobros y reportes con privacidad para datos de salud mental · 2026**

En este SaaS multi-consultorio:
- Cada psicóloga tiene una **página pública editable**.
- Sus pacientes **reservan sin crear cuenta**.
- Ella gestiona agenda, pacientes, pagos con comprobante, mensajes de WhatsApp y reportes desde el celular.

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

Las psicólogas independientes gestionan su consulta con WhatsApp, cuadernos y memoria:
- Las citas y reprogramaciones se conversan por chat.
- Los comprobantes de pago llegan sueltos.
- No hay un registro claro de quién debe qué.
- Los datos de los pacientes quedan sin ninguna protección.

## La solución

| Para | Qué hace |
|---|---|
| **El paciente** | Página de la profesional (enfoque, temas, servicios, formación, ubicación, preguntas frecuentes) y reserva en 4 pasos **sin cuenta** |
| **La psicóloga** | Confirma o rechaza solicitudes y ve su agenda del día y de la semana. También: «programar siguiente cita», «pasar a virtual» y reprogramación con enlace de un solo uso para el paciente |
| **Los cobros** | Efectivo o comprobante, con aprobación o rechazo; pendientes de cobro y comprobantes por revisar |
| **La comunicación** | Mensajes de WhatsApp listos (`wa.me`) con plantillas editables, enviados desde su propio número |
| **Los números** | Reportes por periodo: ingresos, atendidas, inasistencias y cobros por método |
| **La plataforma** | Super Admin con consultorios, planes, auditoría y suspensión de acceso, aplicados por la base de datos |

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

## Arquitectura

![Arquitectura](img/arquitectura.png)

**Decisiones clave:**
- **Aislamiento en dos capas:** la aplicación y PostgreSQL con RLS, más 8 claves foráneas compuestas con `organization_id`. Se verificó con 17 pruebas contra una base real.
- **Minimización:** no se guardan historia clínica, notas ni diagnósticos; solo lo necesario para agendar y cobrar.
- **Archivos privados:** los comprobantes se validan por su firma binaria (SVG prohibido) y se sirven con URLs firmadas de 5 a 10 minutos.
- **Supabase autoalojado** en Docker Compose detrás de Caddy, el único servicio que expone puertos.

## Seguridad: un fallo real encontrado y corregido

Durante las pruebas adversariales descubrí un problema:
- **El fallo:** una función `SECURITY DEFINER` sin `pg_temp` en su `search_path` permitía **saltarse RLS y escalar a Super Admin**.
- **La corrección:** se arregló y ahora una prueba de regresión falla si cualquier función queda sin esa protección.

Además:
- Privilegios recortados (`TRUNCATE`, `TRIGGER`, `REFERENCES`, `MAINTAIN`) y **21/21 pruebas adversariales**; rol `app_user` y PostgREST cerrado.
- Logs con datos personales y secretos redactados (33 pruebas).
- Medición real del límite de autenticación, documentada como riesgo a reforzar.
- Modelo de amenazas basado en OWASP, clasificación de datos, retención programada y pruebas de accesibilidad con axe-core.

## Calidad

- **3116 pruebas Vitest** (unitarias e integración) en el último registro.
- **436 pruebas E2E con Playwright** en 58 specs, que recorren los flujos reales de la psicóloga y del paciente.
- **87 migraciones** versionadas y **177 commits**.

## Estado

- ✅ **MVP completo y probado** en un **ensayo de producción en local**: los mismos contenedores que irán al VPS, con HTTPS, backups y restauración.
- ⏳ **Siguiente paso:**
  - despliegue en el VPS, dominio y certificados reales;
  - correo transaccional;
  - la política de backups antes de recibir datos reales.
- 🚫 **Fuera del MVP a propósito:** pasarela de pago, recordatorios automáticos y autogestión del paciente.

## Mi rol

Proyecto propio de principio a fin:
- Definí el producto con una psicóloga piloto.
- Diseñé el modelo de datos, la arquitectura y la estrategia de privacidad.
- Desarrollé la aplicación con apoyo de asistentes de IA.
- Escribí las pruebas: unitarias, de integración, adversariales y E2E.
- Preparé la infraestructura autoalojada.

---

> 🔒 **Repositorio vitrina:** no se publican el código fuente, los datos reales ni los secretos de infraestructura.
>
> Las capturas son de una copia local con la cuenta de demostración «Ps. Mariana Salazar Rivas». La persona, los pacientes y los teléfonos son ficticios; la foto es de Unsplash (Christina @ wocintechchat).

**Bryan Jeremy Espinoza Herrera** · BJOTA Systems · Perú
