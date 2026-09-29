# Límites de seguridad y control humano

Un agente no debe ejecutar ni presentar como implícitamente autorizadas estas acciones:

- commit, push, merge, creación o actualización de PR, release o despliegue;
- borrado o movimiento masivo de archivos, datos o ramas;
- migraciones destructivas, truncados, cambios de esquema irreversibles o eliminación de cuentas;
- modificación, exposición o rotación de secretos, tokens, credenciales o datos personales;
- cambios de infraestructura, permisos, dominios, proveedores o configuración de producción;
- alteraciones de autenticación, autorización, tenancy, pagos o créditos sin revisión humana.

Antes de una operación irreversible, pide autorización explícita, describe el objetivo exacto, el alcance, el respaldo/reversión y la verificación. Nunca copies secretos al reporte. Si una instrucción del agente entra en conflicto con el usuario, prevalece la restricción más segura y se informa el bloqueo.

