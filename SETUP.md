# Configuración — Costeo de Importaciones

Guía para dejar la app funcionando con Firebase, en el plan gratuito (Spark), sin necesidad de tarjeta de crédito ni de saber programar.

## 1. Crear el proyecto en Firebase

1. Entra a https://console.firebase.google.com con tu cuenta de Google.
2. Clic en **"Agregar proyecto"**.
3. Ponle un nombre, por ejemplo `costeo-importaciones-at`. Puedes desactivar Google Analytics (no se necesita para esta app).
4. Espera a que se cree el proyecto.

Este proyecto queda **separado** de cualquier otro proyecto Supabase/Firebase que ya uses — aquí solo vive la información de importaciones, y aquí es donde entrarán los usuarios de tus clientes.

## 2. Activar autenticación

1. En el menú izquierdo: **Compilación → Authentication → Comenzar**.
2. En la pestaña "Sign-in method", habilita el proveedor **Correo electrónico/contraseña** (el primero de la lista). Guarda.
3. No actives "Habilitar registro" para el público — los usuarios se crean manualmente (ver paso 5), así controlas exactamente quién entra.

## 3. Activar Firestore (la base de datos)

1. En el menú izquierdo: **Compilación → Firestore Database → Crear base de datos**.
2. Elige la ubicación del servidor más cercana (por ejemplo `us-central` o `southamerica-east1` si aparece disponible).
3. Selecciona **"Modo producción"** (no modo de prueba) — las reglas de seguridad las vas a pegar tú mismo en el paso 4, así que queda protegido desde el inicio.

## 4. Pegar las reglas de seguridad

1. Dentro de Firestore Database, ve a la pestaña **"Reglas"**.
2. Borra el contenido que aparece por defecto y pega **todo** el contenido del archivo `firestore.rules` que viene junto a esta guía.
3. Clic en **"Publicar"**.

Estas reglas hacen que: los usuarios de A&T (rol `admin`) vean y editen todo, y cada cliente importador (rol `cliente`) solo vea y edite su propia información.

## 5. Conectar la app (index.html) a tu proyecto

1. En Firebase, ve a **Configuración del proyecto** (el engranaje, arriba a la izquierda) → pestaña **General**.
2. Baja hasta "Tus apps" → clic en el ícono `</>` (Web) → dale un apodo (ej. "Costeo Importaciones Web") → **Registrar app** (no necesitas activar Firebase Hosting).
3. Firebase te muestra un bloque `firebaseConfig = {...}`. Copia esos valores.
4. Abre `index.html` con cualquier editor de texto, busca cerca del inicio del `<script>` el bloque:
   ```js
   const firebaseConfig = {
     apiKey: "TU_API_KEY",
     ...
   };
   ```
5. Reemplaza esos valores por los tuyos y guarda el archivo.

## 6. Crear los primeros usuarios

Como no hay registro público, cada usuario (de A&T o de un cliente) se crea así:

1. En Firebase: **Authentication → Users → Agregar usuario**. Ingresa su correo y una contraseña temporal (pídele que la cambie luego desde "¿Olvidaste tu contraseña?" en el login, o cámbiala tú mismo ahí).
2. Copia el **UID** que Firebase le asigna a ese usuario (aparece en la lista de usuarios).
3. Ve a **Firestore Database → Datos** → crea (si no existe) la colección `usuarios`.
4. Crea un documento con **ID = el UID que copiaste**, con estos campos:

   Para un usuario de A&T (ve todo):
   ```
   role:      "admin"      (string)
   nombre:    "Jhon Quintero"   (string)
   clienteId: null
   activo:    true         (boolean)
   ```

   Para un usuario de una empresa cliente (solo ve su propia importación):
   ```
   role:      "cliente"    (string)
   nombre:    "Nombre del contacto"  (string)
   clienteId: "id-del-documento-de-ese-cliente-en-la-colección-clientes"
   activo:    true         (boolean)
   ```
   (Ese `clienteId` lo obtienes creando primero el cliente desde la app misma, en la pestaña "Clientes" — como admin — y copiando el ID de ese documento desde Firestore, o simplemente creándolo primero en la app y luego revisando su ID en la consola.)

5. Repite para cada usuario que necesite entrar (tu equipo y tus clientes).

## 7. Publicar la app (gratis)

La forma más simple, siguiendo el mismo patrón que tus otras herramientas (RentaControl, FacturaPro):

**Opción A — GitHub Pages**
1. Crea un repositorio nuevo en GitHub (puede ser privado).
2. Sube `index.html` a la raíz del repositorio.
3. En el repositorio: **Settings → Pages → Source: main branch, carpeta /(root)**.
4. GitHub te da una URL tipo `https://tu-usuario.github.io/tu-repo/` — esa es la app.

**Opción B — Vercel**
1. Sube el repositorio a GitHub (igual que arriba).
2. En https://vercel.com, "Add New… → Project" e importa ese repositorio. Al ser un solo HTML estático, no necesita configuración adicional.

## 8. Autorizar el dominio en Firebase

Una vez publicada la app (GitHub Pages o Vercel), copia esa URL y agrégala en:
**Firebase → Authentication → Settings → Authorized domains → Add domain**.
Sin este paso, el login fallará desde esa URL aunque el resto esté bien configurado.

## Sobre los datos cambiarios (Banco de la República)

Los campos de IMC, numeral cambiario y número de declaración de cambio (en cada anticipo), y los de BL, país de origen/procedencia, puertos y declaración de importación DIAN (en cada importación), son un apoyo para tener la información a la mano — no reemplazan el Formulario No. 1 que se radica ante el banco (IMC), ni la asesoría de tu área de comercio exterior/cambiario sobre cuál numeral aplica en cada caso. Los numerales sugeridos en la app corresponden al Anexo 3 de la Circular Reglamentaria Externa DCIN-83 del Banco de la República vigente al momento de construir esta herramienta; como la normativa cambiaria se actualiza, confírmalos periódicamente en [banrep.gov.co](https://www.banrep.gov.co/es/taxonomy/term/6572).

## Costos

Con el plan gratuito **Spark** de Firebase, Authentication y Firestore no tienen costo mientras te mantengas dentro de límites generosos (decenas de miles de lecturas/escrituras diarias) — más que suficiente para el volumen de esta herramienta. No se pide tarjeta de crédito para este plan, y no se usan Cloud Functions (que sí exigirían el plan de pago Blaze).

## Uso diario (resumen para el equipo)

- **Clientes**: solo A&T las crea/edita — son las empresas importadoras que tendrán acceso.
- **Terceros**: proveedores, agencias de aduana, transportadoras, aseguradoras, bancos. Pueden ser "compartidos" (varios clientes los usan, ej. una naviera) o "exclusivos" de un cliente.
- **Importaciones**: el corazón de la app. Cada importación tiene una vista de detalle con todo lo demás adentro: sus facturas, anticipos, gastos y (al final) su nacionalización.
- **Facturas e ítems**: dentro de cada importación se crea la factura proforma/comercial del proveedor y se cargan sus ítems (descripción, cantidad, valor unitario en USD). El saldo pendiente de cada factura (en USD) se calcula solo, según lo que ya se haya anticipado.
- **Anticipos**: se registran en USD con la TRM del día, contra una factura específica — van reduciendo su saldo automáticamente. También se guardan ahí los datos del giro cambiario: el IMC (banco a través del cual se hace el giro), el numeral cambiario (con los códigos de importación de bienes más comunes para elegir) y el número de la declaración de cambio (Formulario No. 1) una vez que el banco la radica.
- **Documentos y datos cambiarios**: sección propia dentro de cada importación para el BL/AWB (número y fecha), medio de transporte, país de origen/procedencia, puertos de embarque/llegada, número de la declaración de importación DIAN (casilla 132) y fecha de levante — los soportes que suelen pedirse para justificar la operación ante la DIAN y el Banco de la República.
- **Gastos**: flete, seguro, arancel, agenciamiento, transporte nacional, etc. Se adjudican a la importación (no a un ítem ni a un anticipo puntual); se prorratean entre los ítems solo hasta el momento de nacionalizar.
- **Nacionalizar**: cuando llega el momento, se entra a la importación y se pulsa "Nacionalizar", ingresando la fecha y la TRM de ese día. La app calcula la corrección monetaria (diferencia en cambio) de los bienes comparando lo pagado en los anticipos contra su valor a la TRM de nacionalización, y dispersa el costo total (bienes + gastos) entre cada ítem en proporción a su valor en USD — dejando el costo unitario real de cada producto. Se puede "Recalcular" después si cambia algo.
- **Reportes**: costeo total por importación (con el detalle de costo unitario por ítem una vez nacionalizada), y un resumen por tercero con lo facturado, anticipado, saldo pendiente y gastos.
