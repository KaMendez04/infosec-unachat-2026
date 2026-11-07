# Bitácora · Laboratorio 8: Client Secret de Auth0 en Supabase Vault

Repositorio: fork `KaMendez04/infosec-unachat-2026`, rama `lab8-vault`.
Ningún valor sensible (Client Secret, contraseña del rol, `VAULT_DATABASE_URL`) aparece en este archivo, en Git ni en las capturas.

| Hora | Etapa | Qué se hizo | Evidencia |
|---|---|---|---|
| 2026-11-06 | 1. Preparación | Fork del repo del curso, rama `lab8-vault`, `.env` ya ignorado por `.gitignore`, Node v24.18.0, `npm ci` y 22 pruebas unitarias en verde. | — |
| 2026-11-06 | 2. Vault | Se creó la aplicación UNA Chat en Auth0 (Regular Web Application, Client Secret (Post)) y se guardó su Client Secret en Supabase Vault con el nombre `una_chat_auth0_client_secret`; el valor queda enmascarado. | [01-vault-secreto.png](evidencias/01-vault-secreto.png) |
| 2026-11-06 | 3. Rol y permisos | Se creó el rol `una_chat_backend` (contraseña fuera del repo) y se aplicó `una_chat_vault_access.sql`. `check_vault_privileges.sql`: solo `backend_execute` es true; anon, authenticated y public, false. | [02-permisos.png](evidencias/02-permisos.png), [02b-secreto-por-nombre.png](evidencias/02b-secreto-por-nombre.png) |
| 2026-11-06 | 4. Bloqueo y lectura | `anon` no puede ejecutar la función: error 42501 (permission denied for schema una_chat_private). | [03-bloqueo-anon-funcion.png](evidencias/03-bloqueo-anon-funcion.png) |
| 2026-11-06 | 4. Bloqueo y lectura | `authenticated` tampoco puede ejecutar la función: mismo error 42501. | [04-bloqueo-authenticated.png](evidencias/04-bloqueo-authenticated.png) |
| 2026-11-06 | 4. Bloqueo y lectura | `anon` no puede leer `vault.decrypted_secrets` directamente: 42501 (permission denied for schema vault). Después se volvió al rol del panel con `reset role`. | [05-bloqueo-anon-vault.png](evidencias/05-bloqueo-anon-vault.png), [05b-reset-role.png](evidencias/05b-reset-role.png) |
| 2026-11-06 | 4. Bloqueo y lectura | Con el rol `una_chat_backend` prestado, la función devuelve un secreto de 64 caracteres (`secret_length = 64`), sin mostrar el valor; el bloque termina devolviendo el permiso con `revoke`. | [06-secret-length.png](evidencias/06-secret-length.png) |
| 2026-11-06 | 4. Bloqueo y lectura | `dashboard_is_member` sigue en true después del `revoke`: es la membresía que PostgreSQL 16 otorga automáticamente (grantor `supabase_admin`) a quien crea un rol con CREATEROLE, y el panel no puede revocarla. Esa membresía tiene `inherit_option` y `set_option` en false: el panel no hereda permisos (`usage = false`), no puede tomar el rol (`set = false`) y `set role una_chat_backend` da 42501. | [07-dashboard-is-member.png](evidencias/07-dashboard-is-member.png), [07a-membresia-automatica.png](evidencias/07a-membresia-automatica.png), [07b-member-true-por-membresia-automatica.png](evidencias/07b-member-true-por-membresia-automatica.png), [07c-set-role-denegado.png](evidencias/07c-set-role-denegado.png) |
| | 5. Auth0 | | `evidencias/08`, `09` |
| | 6. Conexión con Vault | | `evidencias/10-render-environment.png` |
| | 7. Deploy y login | | `evidencias/11` a `13` |
| | 8. Retiro de la variable | | `evidencias/14`, `15` |
| | 9. Rotación | | `evidencias/16` a `19` |

## Pendiente
- Confirmar la fecha de entrega con el profesor: el enunciado dice clase el 06/11/2026 y entrega el "13 de octubre de 2026".
- Comentar con el profesor que en Supabase `dashboard_is_member` no puede quedar en false (membresía automática de `supabase_admin`); se documentó con `usage` y `set` en false.
