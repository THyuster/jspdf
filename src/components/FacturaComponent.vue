<template>
  <div>
    <!-- Formulario para agregar productos -->
    <form @submit.prevent="agregarProducto" class="form">
      <label for="nombre" class="label">Nombre del Producto:</label>
      <input type="text" id="nombre" v-model="nombreProducto" class="input">

      <label for="cantidad" class="label">Cantidad:</label>
      <input type="number" id="cantidad" v-model.number="cantidadProducto" required class="input">
      
      <label for="precio" class="label">Precio Unitario:</label>
      <input type="text" id="precio" v-model="precioProducto" required class="input">

      <button type="submit" id="btn_click" class="btn">Agregar Producto</button>
    </form>

    <!-- Detalle de la factura -->
    <h2 class="section-title">Detalle de la Factura</h2>

    <table class="table">
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Cantidad</th>
          <th>Precio Unitario</th>
          <th>Total</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(producto, index) in productos" :key="index">
          <td>{{ producto.nombre }}</td>
          <td>{{ producto.cantidad }}</td>
          <td>{{ producto.precio.toLocaleString('es-ES') }} pesos</td>
          <td>{{ producto.total.toLocaleString('es-ES') }} pesos</td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3" style="text-align: right;"><strong>Total Factura:</strong></td>
          <td>{{ totalFactura.toLocaleString('es-ES') }} pesos</td>
        </tr>
        <tr>
          <td colspan="4">
            <label for="observaciones" class="label">Observaciones:</label><br>
            <textarea id="observaciones" v-model="observaciones" class="textarea"></textarea>
          </td>
        </tr>
      </tfoot>
    </table>

    <!-- Mostrar total factura -->
    <div class="total-container">
      <div class="total">
        <strong>Total Factura</strong>
      </div>
      <div class="precio-pagar">
        <strong>Precio a pagar:</strong> {{ totalFactura.toLocaleString('es-ES') }} pesos
      </div>
    </div>

    <!-- Información del remitente y destinatario, y datos de la factura -->
    <div style="display: flex; justify-content: space-between; margin-top: 20px;">
      <!-- Datos del remitente -->
      <div>
        <h3 class="section-title">Remitente</h3>
        <label for="nombreRemitente" class="label">Nombre:</label>
        <input type="text" id="nombreRemitente" v-model="nombreRemitente" class="input">

        <label for="idRemitente" class="label">Identificación:</label>
        <input type="text" id="idRemitente" v-model="idRemitente" class="input">

        <label for="telefonoRemitente" class="label">Teléfono:</label>
        <input type="text" id="telefonoRemitente" v-model="telefonoRemitente" class="input">
      </div>

      <!-- Datos del destinatario -->
      <div>
        <h3 class="section-title">Destinatario</h3>
        <label for="nombreDestinatario" class="label">Nombre:</label>
        <input type="text" id="nombreDestinatario" v-model="nombreDestinatario" class="input">

        <label for="idDestinatario" class="label">Identificación:</label>
        <input type="text" id="idDestinatario" v-model="idDestinatario" class="input">

        <label for="telefonoDestinatario" class="label">Teléfono:</label>
        <input type="text" id="telefonoDestinatario" v-model="telefonoDestinatario" class="input">
      </div>

      <!-- Datos de la factura -->
      <div>
        <h3 class="section-title">Datos de la Factura</h3>
        <label for="numeroFactura" class="label">Número de Factura:</label>
        <input type="text" id="numeroFactura" v-model="numeroFactura" readonly class="input">

        <label for="fechaFactura" class="label">Fecha de Factura:</label>
        <input type="date" id="fechaFactura" v-model="fechaFactura" readonly class="input">

        <label for="vencimientoFactura" class="label">Vencimiento de Factura:</label>
        <input type="date" id="vencimientoFactura" v-model="vencimientoFactura" readonly class="input">
      </div>
    </div>

    <!-- Botones para vista previa y descarga de factura -->
    <iframe ref="pdfPreview" style="display: none; width: 40%; height: 400px;"></iframe>
    <button @click="previewPDF" class="btn">Vista previa de la factura</button>
    <button @click="downloadPDF" class="btn">Descargar factura</button>
  </div>
</template>

<script>
import jsPDF from 'jspdf';
import 'jspdf-autotable';

export default {
  data() {
    return {
      nombreProducto: '',
      cantidadProducto: 0,
      precioProducto: '',
      productos: [],
      observaciones: '',
      numeroFactura: 1,
      fechaFactura: '',
      vencimientoFactura: '',
      nombreRemitente: '',
      idRemitente: '',
      telefonoRemitente: '',
      nombreDestinatario: '',
      idDestinatario: '',
      telefonoDestinatario: '',
      maxProductosPorPagina: 20 // Define el máximo de productos por página
    };
  },
  computed: {
    totalFactura() {
      return this.productos.reduce((total, producto) => total + producto.total, 0);
    }
  },
  methods: {
    agregarProducto() {
      const precioNumerico = parseFloat(this.precioProducto.replace(/,/g, ''));

      // Verificar si el precio es un número válido
      if (isNaN(precioNumerico)) {
        alert('Por favor, ingrese un precio válido.');
        return;
      }
      
      const total = this.cantidadProducto * precioNumerico;
      this.productos.push({
        nombre: this.nombreProducto,
        cantidad: this.cantidadProducto,
        precio: precioNumerico,
        total: total
      });
      this.nombreProducto = '';
      this.cantidadProducto = 0;
      this.precioProducto = '';
    },
    previewPDF() {
      const doc = this.generatePDF();
      const pdfDataUri = doc.output('datauristring');
      this.$refs.pdfPreview.src = pdfDataUri;
      this.$refs.pdfPreview.style.display = 'block';
    },
    downloadPDF() {
      const doc = this.generatePDF();
      doc.save("Factura.pdf");
      window.open(doc.output('bloburl'));
      this.numeroFactura++;
    },
    generatePDF() {
      const pageSize = { width: 216, height: 279 };
      const doc = new jsPDF({ format: [pageSize.width, pageSize.height] });

      // Encabezado - Título y datos de la empresa
      const img = new Image();
      img.src = require('@/assets/jj.jpg'); 
      doc.addImage(img, 'PNG', 180, 5, 25, 25);
      doc.setFontSize(10);
      doc.setFont('helvetica', 'bold');
      doc.text('YUSCORP', 164, 17);
      doc.text('Nit: 147852369', 158, 20);
      doc.setFontSize(20);
      doc.line(10, 28, 205, 28);
      doc.setFont('helvetica', 'bold');
      doc.text('Factura', 12, 20);

      // Datos del remitente
      doc.setFontSize(12);
      doc.setFont('bold');
      doc.text('Remitente', 10, 35);
      doc.text(`Nombre: ${this.nombreRemitente}`, 10, 40);
      doc.text(`Identificación: ${this.idRemitente}`, 10, 45);
      doc.text(`Teléfono: ${this.telefonoRemitente}`, 10, 50);

      // Datos del destinatario
      doc.text('Destinatario', 80, 35);
      doc.text(`Nombre: ${this.nombreDestinatario}`, 80, 40);
      doc.text(`Identificación: ${this.idDestinatario}`, 80, 45);
      doc.text(`Teléfono: ${this.telefonoDestinatario}`, 80, 50);

      // Datos de la factura
      doc.text('Datos de la Factura', 150, 35);
      doc.text(`N° Factura: ${this.numeroFactura}`, 150, 40);
      doc.text(`Fecha: ${this.fechaFactura}`, 150, 45);
      doc.text(`Vencimiento: ${this.vencimientoFactura}`, 150, 50);

      // Línea de separación
      doc.setLineWidth(0.5);
      doc.line(10, 55, pageSize.width - 10, 55);

      // Detalle de la factura - Tabla de productos
      let startY = 60;
      let remainingData = [
        ['Nombre', 'Cantidad', 'Precio', 'Total'],
        ...this.productos.map(producto => [
          producto.nombre,
          producto.cantidad,
          `${producto.precio.toLocaleString('es-ES')} pesos`,
          `${producto.total.toLocaleString('es-ES')} pesos`,
        ])
      ];

      while (remainingData.length > 0) {
        const availableSpace = pageSize.height - startY; 
        const maxRows = Math.floor(availableSpace / 20); 
        const rowsForPage = remainingData.slice(0, maxRows);

        doc.autoTable({
          startY: startY,
          head: [remainingData[0]],
          body: rowsForPage,
          theme: 'grid'
        });

        remainingData = remainingData.slice(maxRows);

        if (remainingData.length > 0) {
          doc.addPage();
          startY = 10;
        }
      }

      // Observaciones
      const observacionesXPosition = 10;
      const observacionesYPosition = doc.autoTable.previous.finalY + 10;
      const observacionesWidth = pageSize.width - 20;
      const observacionesHeight = 30;

      doc.setFillColor(255, 255, 255);
      doc.rect(observacionesXPosition, observacionesYPosition, observacionesWidth, observacionesHeight, 'F');
      doc.setTextColor(0, 0, 0);
      doc.setFontSize(12);
      doc.text('Observaciones:', observacionesXPosition + 5, observacionesYPosition + 5);
      doc.setFontSize(14);
      doc.text(this.observaciones, observacionesXPosition + 5, observacionesYPosition + 10, { maxWidth: observacionesWidth - 10 });

      // Total y agradecimiento
      const totalXPosition = 150;
      const priceXPosition = totalXPosition + 14;
      const totalYPosition = pageSize.height - 32; 
      const thanksTextYPosition = totalYPosition + 10;
      doc.text('Total:', totalXPosition, totalYPosition);
      doc.text(`${this.totalFactura.toLocaleString('es-ES')} pesos`, priceXPosition, totalYPosition);
      doc.line(10, totalYPosition + 3, pageSize.width - 10, totalYPosition + 3);
      doc.text('Gracias por su compra', 110, thanksTextYPosition, { align: 'center', fontSize: 12 });

      return doc;
    }
  },
  mounted() {
    const currentDate = new Date();
    this.fechaFactura = currentDate.toISOString().substr(0, 10);

    const dueDate = new Date(currentDate);
    dueDate.setDate(dueDate.getDate() + 30);
    this.vencimientoFactura = dueDate.toISOString().substr(0, 10);
  }
};
</script>

<style scoped>
.label {
  display: block;
  margin-bottom: 5px;
}

.input {
  width: 100%;
  padding: 5px;
  margin-bottom: 10px;
}

.btn {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  margin-right: 10px;
}

.table {
  width: 100%;
  border-collapse: collapse;
}

.table th, .table td {
  border: 1px solid #ddd;
  padding: 8px;
}

.section-title {
  margin-top: 20px;
  margin-bottom: 10px;
}

.total-container {
  margin-top: 20px;
}

.total {
  font-size: 20px;
}

.precio-pagar {
  font-size: 24px;
  margin-top: 10px;
}
</style>
