# SISE — Sistema Integral de Supervisión Educativa

## Contenido
- `index.html`: aplicación web responsiva, CRUD, dashboard, alertas, documentos y exportación CSV.
- `schema.sql`: tablas PostgreSQL, índices, RLS, roles, acceso por escuela y almacenamiento privado.
- `config.example.js`: plantilla de conexión.

## Instalación
1. Crear un proyecto nuevo en Supabase. En **SQL Editor**, ejecutar `schema.sql`.
2. En **Authentication > Users**, crear manualmente el primer usuario con correo y contraseña.
3. Volver a SQL Editor y ejecutar, cambiando el correo:
   ```sql
   insert into public.profiles (id,full_name,role)
   select id,'Administración SISE','admin' from auth.users where email='TU_CORREO@EJEMPLO.COM';
   ```
4. Copiar `config.example.js` como `config.js`. Completar URL y **clave pública** del proyecto. No incluir `service_role` ni claves secretas.
5. Servir la carpeta por HTTPS (Netlify, Vercel o alojamiento estático) o localmente con `python -m http.server 8000`, abrir `http://localhost:8000`.
6. Ingresar con la cuenta creada. El admin crea escuelas. Otros usuarios deben crearse en Auth, luego en `profiles` y asignarse mediante `school_access`.

## Asignar una escuela a otro usuario
```sql
insert into public.profiles (id,full_name,role)
select id,'Inspectora','inspector' from auth.users where email='INSPECTORA@EJEMPLO.COM';
insert into public.school_access (school_id,user_id)
select s.id,p.id from public.schools s cross join public.profiles p
where s.code='EP 7' and p.full_name='Inspectora';
```
El rol `reader` puede consultar, pero no editar. `admin` tiene acceso global. `inspector` accede únicamente a escuelas asignadas.

## Funciones de esta versión
- Alta y edición de escuelas (admin), informes de visita, incidentes, indicadores, paros y acuerdos.
- Eliminación de registros de trabajo, salvo escuelas; filtros por escuela y exportación CSV.
- Legajo institucional y panel general; alertas de asistencia, alfabetización, matemática, acuerdos vencidos, situaciones abiertas y continuidad pedagógica.
- Subida de PDF/Word/imágenes a bucket privado y apertura con URL temporal.

## Limitaciones y resguardos
- **No procesa automáticamente el contenido de PDF/Word** ni interpreta documentos con IA. Los informes se registran mediante campos estructurados y los archivos quedan adjuntos.
- Los umbrales de alerta son **ilustrativos y modificables en `recommendations()`**. No son normativa educativa, ni se debe inferir calidad escolar a partir de ellos.
- No se incluyó un módulo de identificación de estudiantes, docentes involucrados o datos de salud. Evitar escribir datos sensibles en campos de texto libre. Para uso institucional real: validar base legal, clasificación documental, consentimiento/autorizaciones aplicables, retención, cifrado, auditoría y políticas de acceso con el organismo competente.
- La tabla `documents` no tiene interfaz de edición; permite carga, consulta y eliminación.
- Para una implementación de producción agregar auditoría inmutable, recuperación ante desastres, paginación, pruebas automatizadas, MFA y una política de gestión de usuarios. La interfaz no administra cuentas ni asignaciones; se gestionan en Supabase.
- El CDN de supabase-js requiere conexión a Internet. Para despliegue productivo conviene fijar versión exacta y empaquetar dependencias.
- El dashboard promedia los últimos porcentajes de asistencia por escuela sin ponderación por matrícula. Evitar comparaciones entre períodos distintos.
