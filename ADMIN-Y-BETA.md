# Crear tu cuenta admin y una cuenta beta

Esta guía activa dos permisos distintos:

- **Admin:** puede entrar a toda la academia y gestionar caminos, módulos y lecciones.
- **Beta:** puede explorar todo el contenido, pero no editarlo.

## 1. Activar el panel

En Supabase abre **SQL Editor**, crea una consulta nueva y pega el contenido de `supabase/migrations/202609230002_admin_beta_content.sql`. Pulsa **Run** y confirma que aparezca `Success`.

## 2. Crear tu usuario

La app debe estar publicada y conectada a Supabase. En Academia de Brujas, pulsa **Entrar**, escribe tu correo y abre el enlace que llegue a tu correo. Esto crea tu cuenta.

## 3. Darte permiso de administradora

En Supabase, abre **SQL Editor → New query** y ejecuta esto, reemplazando el correo de ejemplo por el correo con el que entraste a la app:

```sql
insert into public.academy_roles (user_id, role)
select id, 'admin'
from auth.users
where lower(email) = lower('tu-correo@ejemplo.com')
on conflict do nothing;
```

## 4. Crear una cuenta beta con acceso completo

La persona debe entrar una vez en la app con su correo. Luego ejecuta otra consulta con ese correo:

```sql
insert into public.academy_roles (user_id, role)
select id, 'beta'
from auth.users
where lower(email) = lower('correo-beta@ejemplo.com')
on conflict do nothing;
```

## 5. Ver los permisos

Cierra sesión en la app y vuelve a entrar. Tu cuenta administradora tendrá **Administrar** en el menú. La cuenta beta tendrá acceso a las rutas, sin herramientas de edición.

Si SQL devuelve `Success` pero ninguna fila coincide, revisa que esa persona haya iniciado sesión primero y que el correo esté escrito igual al de su cuenta.

La clave `service_role`, la clave secreta de Supabase y el Hottok de Hotmart no se necesitan para asignar estos permisos; nunca los pegues en la PWA.
