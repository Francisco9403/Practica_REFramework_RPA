# 🤖 RPA MercadoLibre Price Validator (Dispatcher & Performer)

Este proyecto es una automatización robótica desarrollada en **UiPath** utilizando el **Robotic Enterprise Framework (REFramework)**. El robot simula el proceso de un analista de compras: busca productos en MercadoLibre, extrae sus precios en tiempo real y valida si se ajustan a un presupuesto predefinido.

## 📋 Descripción del Proceso

El proyecto sigue el modelo de arquitectura **Dispatcher & Performer** para garantizar escalabilidad y manejo de errores:

1.  **Dispatcher (Cargador):** Lee una lista de productos y presupuestos (ej. Excel) y los carga en una Cola de Orquestador (`Cola_Precios_Retail`).
2.  **Performer (Procesador):**
    * Obtiene los ítems de la cola transacción por transacción.
    * Navega a MercadoLibre y busca el producto.
    * Utiliza **Selectores Dinámicos (Strict Selectors)** para identificar el precio independientemente de cambios menores en la UI.
    * Limpia y transforma los datos (String a Decimal).
    * Aplica **Lógica de Negocio**:
        * ✅ **Success:** Si el precio es menor o igual al presupuesto.
        * ⚠️ **Business Exception:** Si el precio excede el presupuesto (reportado en Orchestrator sin detener el bot).

## 🛠️ Tecnologías y Conceptos Clave

* **UiPath Studio** (Windows - VB.Net)
* **REFramework:** Manejo robusto de excepciones, reintentos y logging.
* **Orchestrator Queues:** Gestión de transacciones y estados.
* **Web Automation:**
    * Uso de `Strict Selectors` con atributos wildcards para manejar el diseño dinámico de MercadoLibre (clases `andes-money-amount`).
    * Estrategias de espera (`WaitForReady: Complete`) y limpieza de selectores.
* **Data Manipulation:** Limpieza de strings (moneda, puntos de mil) y conversión de tipos para validaciones matemáticas.

## ⚙️ Configuración (Setup)

Para ejecutar este robot en tu entorno local:

1.  **Pre-requisitos:**
    * UiPath Studio instalado.
    * Google Chrome con la extensión de UiPath habilitada.
    * Conexión a UiPath Orchestrator.

2.  **Configuración de Orchestrator:**
    * Crear una Cola llamada: `Cola_Precios_Retail`.
    * Asegurarse de que el robot tenga permisos en la carpeta (ej. `Shared`).

3.  **Config file:**
    * El archivo `Data/Config.xlsx` debe apuntar a la cola correcta en la hoja `Settings`.

## 🚀 Ejecución

1.  Ejecutar `Dispatcher_Cargador.xaml` para poblar la cola con ítems de prueba (Producto + Presupuesto).
2.  Ejecutar `Main.xaml`. El robot comenzará a procesar los ítems pendientes automáticamente.

## 📝 Ejemplo de Lógica

```vb
' Limpieza de datos
str_PrecioLimpio = str_PrecioBruto.Replace(".", "").Replace("$", "").Trim

' Validación
If (CDec(str_PrecioLimpio) > CDec(in_Presupuesto)) Then
    Throw New BusinessRuleException("Presupuesto excedido")
Else
    LogMessage("Compra Aprobada")
End If
