# 👨‍💻 Jorge Luis Ponte Torrealba
**Ingeniero de Sistemas | Especialista en Odoo ERP, Automatización (n8n / Python) & Agentes de IA**

Bienvenido a mi portafolio técnico. Aquí documento arquitecturas reales de automatización empresarial, integración de Inteligencia Artificial con bases de datos vectoriales (RAG) y desarrollo sobre el motor interno de **Odoo ERP** desplegadas en entornos de producción y pruebas sobre **Linux (VPS) y Docker**.

---

## 🛠️ Tech Stack Principal

* **ERP & Backend:** Odoo ERP (Inventario, Facturación, Ventas, Finanzas, Odoo Studio), Python (Odoo ORM), API JSON-RPC, reportes QWeb PDF.
* **Inteligencia Artificial & Automatización:** n8n, LangChain, Bases de Datos Vectoriales (**Qdrant** + OpenAI Embeddings), **OpenRouter** (`gpt-4o-mini`), transcripción de voz (OpenAI Audio).
* **Omnicanalidad & Infraestructura:** Integración por API REST / Webhooks con **Chatwoot**, JavaScript (transformación de payloads), servidores Linux (VPS) y contenedores Docker.
* **Ciberseguridad & Datos:** Autenticación de Dos Factores (2FA), Control de Acceso Basado en Roles (RBAC), auditoría contable y trazabilidad por números de serie.

---

## 🚀 Proyectos Destacados de Automatización e IA

### 1. Agente de IA Omnicanal para Ventas con RAG, Voz y Búsqueda Web Anti-Alucinación
**El Reto:** Automatizar la atención al cliente 24/7 respondiendo consultas de inventario, precios y especificaciones técnicas de equipos electrónicos sin inventar datos (alucinaciones).

**La Solución Técnica:**
* Recepción de mensajes desde **Chatwoot** vía Webhook en **n8n**, con detección automática de notas de voz y transcripción al español mediante **OpenAI Audio**.
* Arquitectura **RAG en Qdrant** (`text-embedding-3-small`) para traducir nombres informales o errores ortográficos del cliente a códigos exactos (`default_code`) del catálogo.
* Conexión en tiempo real a **Odoo ERP** para validar stock (`qty_available > 0`) y costos, sumada a una herramienta conectada a **OpenRouter (`gpt-4o-mini:online`)** que verifica fichas técnicas oficiales de fabricantes en la web.
* Escalamiento inteligente a asesores humanos en Chatwoot (apertura de conversación, nota privada, etiquetado `atencion-humana` y asignación automática) cuando se detecta intención de compra.

<img width="1908" height="944" alt="image" src="https://github.com/user-attachments/assets/724a790b-3c5a-460e-9679-160fb7bfb67e" />



---

### 2. Copiloto Interno de Ventas con Cálculo Multimoneda en Tiempo Real
**El Reto:** Acelerar el tiempo de respuesta del equipo humano de ventas al consultar disponibilidad de inventario, listas de precios especiales y tasas de cambio oficiales.

**La Solución Técnica:**
* Asistente conversacional interno para el equipo comercial que procesa consultas por texto o nota de voz.
* Extracción automática de la tasa cambiaria oficial (`res.currency`) directamente desde **Odoo ERP** antes de cada respuesta.
* Consulta combinada en **Qdrant** y **Odoo** (incluyendo validación de listas de precios especiales en `product.pricelist.item`).
* Post-procesamiento en **JavaScript** que calcula y formatea instantáneamente 6 esquemas de precios (detal, mayorista, divisas y moneda local) y los entrega en **Chatwoot**.

<img width="1911" height="942" alt="image" src="https://github.com/user-attachments/assets/0e0a36bd-6ff6-49a0-9242-64517dc8624e" />


---

### 3. Integración Automatizada de Ventas Online (E-Commerce a Odoo ERP)
**El Reto:** Eliminar el registro manual de pedidos web en el ERP y evitar la duplicación de fichas de clientes en la base de datos.

**La Solución Técnica:**
* Flujo automatizado en **n8n** que recibe órdenes de compra mediante Webhook y verifica en **Odoo** si el cliente ya existe por teléfono o nombre (`res.partner`), creándolo únicamente si es un registro nuevo.
* Creación secuencial de la cotización (`sale.order`) e inserción dinámica de las líneas de productos solicitados (`sale.order.line`).
* Ejecución de llamadas directas a la **API JSON-RPC de Odoo** (`execute_kw`) para asignar etiquetas personalizadas y ejecutar el método `action_confirm`, dejando la orden de venta confirmada sin intervención humana.

<img width="1902" height="943" alt="image" src="https://github.com/user-attachments/assets/28edee2d-2a48-4438-9d13-4b6d98a1cef4" />

---

### 4. Sistema Automatizado de Alertas de Reabastecimiento Crítico (Odoo ORM + Python)
**El Reto:** Prevenir quiebres de inventario en productos de alta rotación identificando automáticamente aquellos que alcanzan su stock mínimo o quedan agotados.

**La Solución Técnica:**
* Acción programada desarrollada en **Python** utilizando el **ORM de Odoo** sobre el modelo de reglas de abastecimiento (`stock.warehouse.orderpoint`).
* Filtrado y ordenamiento por nivel de urgencia, construcción dinámica de una tabla resumen en HTML/CSS y renderizado automático de un reporte adjunto en **PDF mediante QWeb** (`_render_qweb_pdf` y `ir.attachment` en Base64) enviado por correo electrónico a la gerencia.

```python
# Odoo Server Action / Cron Job (Python ORM + QWeb PDF)
# Modelo: Regla de inventario mínimo (stock.warehouse.orderpoint)
# Objetivo: Detectar productos críticos (Agotados o Stock <= Mínimo) y enviar alerta HTML + PDF por correo

reglas_all = env['stock.warehouse.orderpoint'].search([])
criticos = reglas_all.filtered(
    lambda r: r.product_id.active and r.product_id.type != 'service' and (
        r.product_min_qty > 0 and r.qty_on_hand <= r.product_min_qty
    )
).sorted(key=lambda r: (r.qty_on_hand > 0, -(r.product_max_qty - r.qty_on_hand)))

if criticos:
    filas_html = ""
    for r in criticos:
        por_pedir = r.product_max_qty - r.qty_on_hand
        estado = "AGOTADO" if r.qty_on_hand <= 0 else "STOCK BAJO"
        color_estado = "#dc2626" if r.qty_on_hand <= 0 else "#d97706"
        filas_html += f"""
        <tr>
            <td style="padding:8px; border:1px solid #ddd; text-align:center; color:{color_estado}; font-weight:bold;">{estado}</td>
            <td style="padding:8px; border:1px solid #ddd;"><b>{r.product_id.display_name}</b></td>
            <td style="padding:8px; border:1px solid #ddd; text-align:center;">{r.location_id.display_name}</td>
            <td style="padding:8px; border:1px solid #ddd; text-align:center; color:#dc2626;"><b>{r.qty_on_hand:g}</b></td>
            <td style="padding:8px; border:1px solid #ddd; text-align:center;">{r.product_min_qty:g}</td>
            <td style="padding:8px; border:1px solid #ddd; text-align:center;">{r.product_max_qty:g}</td>
            <td style="padding:8px; border:1px solid #ddd; text-align:center; background-color:#fef3c7;"><b>{por_pedir:g} {r.product_uom_name}</b></td>
        </tr>
        """

    cuerpo_correo = f"""
    <div style="font-family: Arial, sans-serif; color: #333;">
        <h2 style="color: #1e3a8a;">📦 Reporte Semanal de Reabastecimiento - ERP Odoo</h2>
        <p>A continuación se detallan los <b>{len(criticos)} productos</b> cuyo inventario actual se encuentra agotado o por debajo del stock mínimo calculado por rotación:</p>
        <table style="border-collapse: collapse; width: 100%; font-size: 13px;">
            <thead>
                <tr style="background-color: #1e3a8a; color: white;">
                    <th style="padding:8px; border:1px solid #ddd; color:white;">Estado</th>
                    <th style="padding:8px; border:1px solid #ddd; text-align:left; color:white;">Producto</th>
                    <th style="padding:8px; border:1px solid #ddd; color:white;">Ubicación</th>
                    <th style="padding:8px; border:1px solid #ddd; color:white;">Stock Actual</th>
                    <th style="padding:8px; border:1px solid #ddd; color:white;">Mínimo</th>
                    <th style="padding:8px; border:1px solid #ddd; color:white;">Máximo</th>
                    <th style="padding:8px; border:1px solid #ddd; color:white;">Cantidad a Pedir</th>
                </tr>
            </thead>
            <tbody>
                {filas_html}
            </tbody>
        </table>
        <p style="margin-top:15px; font-size:12px; color:#666;">Generado automáticamente por Odoo ERP - Control de Inventario.</p>
    </div>
    """

    reporte_xml_id = 'studio_customization.reporte_reabastecimiento_custom'
    pdf_content, _ = env['ir.actions.report']._render_qweb_pdf(reporte_xml_id, res_ids=criticos.ids)
    adjunto = env['ir.attachment'].create({
        'name': 'Reporte_Reabastecimiento_Critico.pdf',
        'type': 'binary',
        'datas': b64encode(pdf_content),
        'res_model': 'mail.mail',
        'mimetype': 'application/pdf',
    })

    valores_correo = {
        'subject': f'🚨 Alerta de Stock Crítico Semanal ({len(criticos)} productos por reponer)',
        'body_html': cuerpo_correo,
        'email_to': 'gerencia@empresa.com, compras@empresa.com',
        'auto_delete': False,
        'attachment_ids': [(6, 0, [adjunto.id])],
    }
    correo = env['mail.mail'].create(valores_correo)
    correo.send()
