# SPECS — Panel de Administración de AgentHub

## 1. Producto

**AgentHub** es una plataforma SaaS donde las empresas alquilan agentes de IA preconfigurados con *skills* (navegar por la web, leer documentos, gestionar calendarios, enviar correos) para tareas de negocio específicas.

Este documento describe el **panel de administración interno**. Lo usa el **administrador de AgentHub** (equipo interno) para revisar métricas, usuarios, agentes, skills, contratos y errores de ejecución. Es un **prototipo**: todos los datos están hardcodeados y no hay conexión a API ni backend.

## 2. Stack y restricciones

- **HTML** semántico: `header`, `nav`, `main`, `section`, `table`, `dialog`, `button`.
- **Tailwind CSS vía CDN** (Play CDN v3, configurado con `darkMode: "class"`). Sin archivos CSS propios, sin bloques `<style>` y sin atributos `style` en línea.
- **JavaScript vanilla** en un `<script>` al final de `index.html`. Sin frameworks, sin jQuery y sin herramientas de build.
- Un único archivo `index.html`. Las secciones se muestran y ocultan según el hash de la URL (`#dashboard`, `#usuarios`, etc.).
- Los datos viven en arrays de JavaScript (una sola fuente de verdad) y las tablas y listados se generan a partir de ellos. Así los nombres son los mismos en todas las secciones.
- Idioma de la interfaz: español. Moneda: USD.
- Debe ser usable en escritorio (≥ 1024 px) y tablet (≥ 768 px).

## 3. Datos de ejemplo (coherentes entre secciones)

| Agente | Propietario (cliente) | Estado | Skills |
|---|---|---|---|
| Atlas | Acme Retail | Activo | Navegación web, Lectura de documentos |
| Nexo | Nova Logística | Activo | Gestión de calendario, Envío de correos |
| Lumen | Banco Austral | Fallando | Lectura de documentos, Envío de correos |
| Orión | Clínica Sur | Inactivo | Gestión de calendario |
| Vega | Acme Retail | Fallando | Navegación web, Lectura de documentos, Envío de correos |

- Precios por contrato: tarifa base del agente USD 200 + cada skill (Navegación web 40, Lectura de documentos 30, Gestión de calendario 25, Envío de correos 20).
- Un contrato por agente (5 contratos). Total bruto USD 1.280; descuentos USD 85 (Atlas: cupón −30; Vega: descuento −55); **ingresos totales USD 1.195**.
- Las métricas del Dashboard se derivan de estos datos: 2 agentes activos (Atlas, Nexo) y 2 fallando (Lumen, Vega).
- El contador de cada skill coincide con los agentes que la tienen (Navegación 2, Lectura 3, Calendario 2, Correos 3).

## 4. Layout global

1. **Barra lateral (`nav`)** fija a la izquierda, de ancho `w-56` (`lg:w-64`), con el logo "AgentHub" y seis enlaces con ícono: Dashboard, Usuarios, Agentes, Skills, Contrataciones y Log de errores.
2. El enlace de la sección activa se distingue con fondo de acento (indigo), texto en negrita y `aria-current="page"`. Al cambiar de sección se actualiza el indicador.
3. **Barra superior (`header`)** con el título de la sección actual a la izquierda y, a la derecha, el **toggle de modo oscuro/claro** (botón con ícono de sol o luna) y el avatar del administrador.
4. El toggle agrega o quita la clase `dark` en `<html>`; todas las superficies usan variantes `dark:`. El modo se mantiene al cambiar de sección (y se guarda en `localStorage` para que sobreviva a una recarga).
5. Contenido principal en `main`, con un `section` por vista; solo la sección activa es visible.

## 5. Especificaciones por sección

### 5.1 Dashboard
1. Cuatro **tarjetas de métricas** en una cuadrícula responsive (`grid-cols-2`, `xl:grid-cols-4`): Ingresos totales (USD 1.195), Pérdidas por descuentos y cupones (USD 85), Agentes activos (2) y Agentes fallando (2). Cada una tiene un ícono, una etiqueta y un valor grande.
2. Cada tarjeta tiene un color de acento distinto según el tipo de métrica (verde para ingresos, ámbar para pérdidas, indigo para activos, rojo para fallando), aplicado al fondo del ícono, y una sombra sutil (`shadow-sm`).
3. Debajo de las tarjetas, un `div` de ancho completo y alto fijo (`h-64`), con borde discontinuo y la etiqueta centrada "Gráfico de actividad semanal (próximamente)", que marca la posición del gráfico.

### 5.2 Gestión de usuarios
1. Una `table` con las columnas Nombre, Email, Plan, Estado y Acciones, con 5 usuarios hardcodeados (uno por cliente y uno sin agentes).
2. El estado se muestra como **badge**: Activo (verde), Suspendido (rojo) y Pendiente (ámbar).
3. Cada fila tiene un **dropdown de acciones** (botón `⋮`) con "Ver detalle" y "Eliminar".
4. "Ver detalle" abre el **modal** con el registro completo: nombre, email, empresa, plan, estado, fecha de registro y agentes que posee.
5. "Eliminar" quita la fila de la tabla después de pedir confirmación con `confirm()`.

### 5.3 Gestión de agentes
1. Un listado de 5 **tarjetas de agente** que muestran nombre, propietario y un badge de estado: Activo (verde), Inactivo (gris) y Fallando (rojo).
2. Cada tarjeta tiene la **lista de skills colapsable**, cerrada por defecto. Un botón "Skills (n)" con un chevron la despliega con una transición de altura (`grid-rows-[0fr]` → `grid-rows-[1fr]`, 300 ms); el chevron rota 180°. Otro clic la colapsa. El botón refleja el estado en `aria-expanded`.
3. Cada agente tiene un **dropdown** con "Configurar" y "Eliminar".
4. "Configurar" abre el modal con el **prompt de sistema** del agente en un `<textarea>` editable y un botón "Guardar" que actualiza el prompt en memoria.

### 5.4 Skills
1. Un párrafo explicativo al inicio: qué es una skill en AgentHub (una capacidad que se adjunta a un agente para que pueda realizar un tipo de tarea).
2. Un catálogo en cuadrícula de 4 **tarjetas de skill** con nombre, descripción breve, precio y un indicador "n agentes la tienen habilitada" que coincide con los datos de la sección Agentes.
3. Cada skill tiene un **dropdown** con "Ver detalle" (modal con la descripción, el precio y la lista de agentes que la usan) y "Eliminar".

### 5.5 Contrataciones de agentes
1. Una `table` con las columnas Cliente, Agente, Skills contratadas, Inicio, Fin, Estado (Vigente / Finalizado) e Importe pagado, con 5 contratos.
2. Las skills contratadas se muestran como chips pequeños dentro de la celda.
3. Cada fila tiene un **dropdown** con "Ver detalle", que abre el modal con el desglose: tarifa base, cada skill con su precio, subtotal, descuento o cupón, y total pagado.

### 5.6 Log de errores
1. Una `table` con 6 errores: Timestamp, Agente, Tipo, Descripción, Estado y Acciones, ordenada del más reciente al más antiguo.
2. El tipo de error se muestra como badge con código de color por gravedad: Autenticación (rojo, crítico), Timeout (ámbar), Límite de tasa (naranja) y Parseo (azul). Una leyenda sobre la tabla explica los colores.
3. Cada entrada tiene un **dropdown** con "Ver detalle" (modal con la traza completa en un bloque `<pre>` monoespaciado) y "Marcar como resuelto" (cambia el estado de "Abierto" a "Resuelto" y atenúa la fila).

## 6. Inventario de componentes reutilizables

| Componente | Dónde aparece | Comportamiento |
|---|---|---|
| Sidebar / enlace de navegación | Global | Indicador de sección activa (`aria-current`) |
| Toggle de modo oscuro | Header | Cambia la clase `dark` en `<html>` y la persiste |
| Tarjeta de métrica | Dashboard | Ícono, etiqueta y valor; acento por tipo |
| Badge | Usuarios, Agentes, Contratos, Errores | Color según estado o tipo |
| Dropdown de acciones (`⋮`) | Todas las secciones de listado | Uno abierto a la vez; se cierra al hacer clic fuera o con Escape |
| Modal (`<dialog>`) | Usuarios, Agentes, Skills, Contratos, Errores | Contenido dinámico; se cierra con el botón ✕, clic en el backdrop o Escape |
| Lista de skills colapsable | Agentes | Cerrada por defecto; transición de altura |
| Chip de skill | Agentes, Contratos | Etiqueta pequeña con el nombre de la skill |

## 7. Criterios de aceptación

1. `SPECS.md` está commiteado en un commit anterior al primer archivo HTML.
2. La barra lateral muestra las seis secciones; al hacer clic en cada enlace se ve solo esa sección y el enlace queda marcado como activo.
3. El toggle de modo oscuro cambia todo el panel (sidebar, header, tarjetas, tablas, modal) y el modo elegido se mantiene al navegar entre secciones.
4. El Dashboard muestra 4 tarjetas de métricas con ícono, etiqueta y valor, y el área del gráfico de ancho completo debajo.
5. La tabla de usuarios tiene al menos 5 filas con badge de estado.
6. Todas las filas o tarjetas de los listados tienen un dropdown `⋮` que se abre al hacer clic y se cierra al hacer clic otra vez o fuera de su área.
7. "Ver detalle" abre el modal con datos completos en Usuarios, Skills, Contrataciones y Log de errores; "Configurar" abre el modal con un `<textarea>` editable en Agentes.
8. Todos los modales se cierran con el botón de cierre y con clic en el backdrop.
9. Las listas de skills de los agentes están colapsadas por defecto y se expanden y colapsan con una transición visible.
10. Los nombres de agentes y clientes coinciden en Agentes, Contrataciones y Log de errores, y los contadores de skills y las métricas del Dashboard cuadran con los listados.
11. "Marcar como resuelto" cambia el estado del error a "Resuelto".
12. No hay atributos `style`, bloques `<style>` ni archivos CSS propios; todo el estilo es de Tailwind.
13. El panel es usable a 768 px de ancho (las tablas se desplazan horizontalmente si no caben).
