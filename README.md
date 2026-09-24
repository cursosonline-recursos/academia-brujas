# Academia de Brujas — PWA

Experiencia web instalable para Academia de Brujas. Incluye navegación adaptable, academia con ocho caminos, práctica diaria, grimorio local, publicaciones del aquelarre, perfil de progreso, inicio de sesión por enlace mágico preparado para Supabase y endpoint de webhook Hotmart.

## Estado de esta entrega

- La interfaz principal funciona en modo de muestra y guarda las páginas del grimorio/publicaciones en el almacenamiento local del navegador.
- La autenticación por correo, la lectura de permisos Premium y el checkout Hotmart se activan al configurar Supabase y el enlace de pago.
- El endpoint de Hotmart está preparado para validar `X-HOTMART-HOTTOK` y actualizar permisos; falta desplegarlo en un proyecto Supabase real y pegar sus secretos.
- Los contenidos de los cursos son estructura editorial demostrativa. Antes de cobrar por ellos hay que cargar lecciones completas, autores, términos de compra y soporte.
- El mercado de servicios, reservas y pagos a practicantes está definido en la arquitectura funcional, pero no implementado en esta primera entrega de la PWA.

La interfaz contiene experiencias de muestra; **no debe anunciarse como suscripción activada hasta conectar y verificar Hotmart/Supabase**.

## Ver localmente

Abre `index.html` en un navegador para recorrer la interfaz. Para instalar la PWA o que funcione el service worker, sírvela desde `localhost` o un dominio con HTTPS (GitHub Pages lo proporciona automáticamente).

## Publicar en GitHub Pages

1. Crea un repositorio y copia el contenido de esta carpeta como raíz del repositorio.
2. En GitHub abre **Settings → Pages** y selecciona **GitHub Actions** como origen.
3. Sube la rama `main`; el workflow `.github/workflows/deploy.yml` publicará la PWA.
4. La URL inicial tendrá formato `https://usuario.github.io/repositorio/`. Después puedes conectar un dominio propio.

La PWA es estática, pero los secretos de Hotmart y la clave `service_role` de Supabase nunca se ponen aquí.

## Conectar autenticación y compra

1. Crea un proyecto Supabase.
2. Ejecuta `supabase/migrations/202609230001_membership.sql`.
3. Completa `config.js` con `supabaseUrl`, `supabaseAnonKey` y `hotmartCheckoutUrl`. La anon/publishable key sí puede estar en el navegador; nunca pegues la service role.
4. Configura plantillas de correo y URL de retorno permitida para el dominio de la PWA en Supabase Auth.
5. Despliega la función `hotmart-webhook` y configura los secretos en el backend:

```text
HOTMART_HOTTOK=<Hottok de tu cuenta Hotmart>
HOTMART_PRODUCT_IDS=<ID de producto, separado por coma si hay varios>
HOTMART_ACCESS_DAYS=<product_id:días; opcional, p.ej. 1234567:32>
```

Supabase ya proporciona `SUPABASE_URL` y `SUPABASE_SERVICE_ROLE_KEY` en el entorno de Edge Functions. Configura ambos como secretos de servidor si tu método de despliegue no los inyecta automáticamente.

6. En Hotmart crea el webhook para el producto y selecciona eventos de compra aprobada/completada, pago recurrente aprobado, retraso/vencimiento, cancelación, reembolso y contracargo. Usa como URL la dirección de la función desplegada.
7. Activa acceso para compras confirmadas. Al crear cuenta en la PWA, usar el mismo correo de compra vincula el permiso automáticamente.
8. Revisa la carga real del webhook en el historial de Hotmart; los nombres y campos pueden variar con la versión de webhook configurada. La función maneja los campos comunes de las versiones 1/2; coteja los payloads antes de abrir ventas.

## Regla de acceso incluida

- Compra/renovación aprobada: estado `active` y vencimiento según la fecha de acceso que envíe Hotmart; si no viene, se usa `HOTMART_ACCESS_DAYS` o 32 días (configura explícitamente el valor para cada plan, sobre todo anual).
- Cancelación: se deja de renovar. Se conserva acceso hasta la fecha final informada por Hotmart; si no viene fecha, se revoca de inmediato.
- Atraso: estado `past_due`, sin acceso premium mientras el cobro se recupera.
- Reembolso/contracargo: acceso retirado.

La política de acceso tras cancelar debe coincidir con lo que informas antes de la compra y con la configuración de Hotmart.

## Seguridad

- Hotmart llama un endpoint público porque no inicia sesión en Supabase Auth; la función valida primero el Hottok configurado.
- No guardar secretos ni en `config.js`, ni en GitHub Pages, ni en el código cliente.
- Configurar redirect URLs de Supabase estrictamente para los dominios de producción y desarrollo.
- El estado de membresía se consulta con la función SQL `has_academy_access()` y RLS protege la fila vinculada a la cuenta.
- Revisa límites, políticas de privacidad, reembolsos y términos de suscripción antes del lanzamiento comercial.

## Estructura

```text
index.html / styles.css / app.js   Interfaz de la PWA
manifest.json / sw.js / icon.svg   Instalación y caché offline
config.example.js                 Valores públicos de configuración
supabase/migrations                Permisos y vínculo con usuarios
supabase/functions/hotmart-webhook Recepción de eventos Hotmart
.github/workflows/deploy.yml       Publicación automática en GitHub Pages
```
