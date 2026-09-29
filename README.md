# Reintento automático de instancia ARM (A1.Flex) en Oracle Cloud

Este script corre en **GitHub Actions** (gratis) e intenta crear tu instancia
ARM cada ~10 minutos hasta que Oracle tenga capacidad disponible. Cuando lo
logra, te manda una notificación push a tu celular.

## Paso 1: Crea un repositorio en GitHub

1. Ve a https://github.com/new
2. Puede ser **privado** (recomendado, porque vas a guardar datos sensibles
   como "Secrets", que GitHub cifra y nadie puede ver, ni tú después de
   guardarlos).
3. Sube estos dos archivos manteniendo la misma estructura de carpetas:
   ```
   .github/workflows/retry.yml
   README.md
   ```

## Paso 2: Consigue los datos que necesitas de Oracle Cloud

En la consola de Oracle Cloud:

| Dato | Dónde encontrarlo |
|---|---|
| `OCI_TENANCY_OCID` | Perfil (ícono arriba a la derecha) → Tenancy: xxxx → copia el OCID |
| `OCI_USER_OCID` | Perfil → My profile → copia el OCID de tu usuario |
| `OCI_REGION` | El identificador de tu región, ej. `sa-bogota-1` |
| `OCI_COMPARTMENT_OCID` | Identity → Compartments → el compartment donde creas recursos (o usa el mismo que el tenancy OCID si usas el compartment raíz) |
| `OCI_AD` | Compute → Instances → Create Instance → en "Placement" verás el nombre completo, algo como `xxxx:SA-BOGOTA-1-AD-1` |
| `OCI_SUBNET_OCID` | Networking → Virtual Cloud Networks → tu VCN → Subnets → copia el OCID |
| `OCI_IMAGE_OCID` | Al elegir la imagen (ej. Ubuntu, Oracle Linux) en el asistente, o con `oci compute image list` |
| `OCI_OCPUS` | Cuántos OCPUs quieres, ej. `1` o `4` |
| `OCI_MEMORY_GB` | Cuánta RAM quieres, ej. `6` o `24` |

### Generar la API Key (para `OCI_PRIVATE_KEY` y `OCI_FINGERPRINT`)

1. En la consola: Perfil → My profile → "API keys" (en el menú de la izquierda)
2. Clic en "Add API Key" → "Generate API Key Pair"
3. Descarga la **clave privada** (archivo `.pem`) — este es tu `OCI_PRIVATE_KEY`
4. Oracle te mostrará el **fingerprint** — cópialo, es tu `OCI_FINGERPRINT`

### Tu clave SSH pública (`SSH_PUBLIC_KEY`)

Si no tienes una, genérala en tu PC con:
```
ssh-keygen -t rsa -b 4096
```
y copia el contenido del archivo `id_rsa.pub` (no el privado).

## Paso 3: Configura las notificaciones (ntfy.sh, gratis, sin registro)

1. Instala la app **ntfy** en tu celular (Android/iPhone) o simplemente
   entra a https://ntfy.sh desde el navegador.
2. Inventa un nombre de "topic" único y secreto, por ejemplo
   `oracle-arm-juan-8823kd` (que nadie más adivine).
3. En la app, suscríbete a ese mismo nombre de topic.
4. Ese nombre es tu `NTFY_TOPIC`.

## Paso 4: Guarda todo como "Secrets" en GitHub

En tu repositorio: **Settings → Secrets and variables → Actions → New repository secret**

Crea un secret por cada uno de estos nombres, con su valor correspondiente:

```
OCI_TENANCY_OCID
OCI_USER_OCID
OCI_FINGERPRINT
OCI_PRIVATE_KEY       (pega el contenido completo del archivo .pem)
OCI_REGION
OCI_COMPARTMENT_OCID
OCI_AD
OCI_SUBNET_OCID
OCI_IMAGE_OCID
OCI_OCPUS
OCI_MEMORY_GB
SSH_PUBLIC_KEY
NTFY_TOPIC
```

## Paso 5: Actívalo

1. Ve a la pestaña **Actions** de tu repositorio.
2. Deberías ver el workflow "Reintentar crear instancia ARM (A1.Flex)".
3. Puedes correrlo manualmente con el botón "Run workflow" para probar que
   todo esté bien configurado, o simplemente esperar a que corra solo cada
   10 minutos.

## Cosas importantes que debes saber

- **GitHub apaga automáticamente los workflows programados (cron) si el
  repositorio no tiene actividad (commits) por 60 días.** Si lo dejas
  corriendo mucho tiempo, entra de vez en cuando y haz cualquier commit
  pequeño para reactivarlo.
- GitHub no garantiza el minuto exacto del cron; en horas de mucho tráfico
  puede demorarse unos minutos extra en ejecutar.
- El script no crea una segunda instancia si ya tienes una corriendo
  (verifica esto antes de intentar), así que es seguro dejarlo corriendo
  indefinidamente.
- Todo esto usa la **API pública oficial de Oracle**, no es nada malicioso
  ni contra las reglas de Oracle — es una práctica muy común para el
  Always Free tier.
