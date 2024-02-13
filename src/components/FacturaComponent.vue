<template>
  <div>
    <form @submit.prevent="agregarProducto">
      <label for="nombre">Nombre del Producto:</label>
      <input type="text" id="nombre" v-model="nombreProducto" >

      <label for="cantidad">Cantidad:</label>
      <input type="number" id="cantidad" v-model.number="cantidadProducto" required>

      <label for="precio">Precio Unitario:</label>
      <input type="text" id="precio" v-model="precioProducto" required>

      <button type="submit" id="btn_click">Agregar Producto</button>
    </form>

    <h2>Detalle de la Factura</h2>

    <table>
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
          <td>{{ producto.precio }} pesos</td>
          <td>{{ producto.total }} pesos</td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3" style="text-align: right;"><strong>Total Factura:</strong></td>
          <td>{{ totalFactura }} pesos</td>
        </tr>
        <tr>
          <td colspan="4">
            <label for="observaciones">Observaciones:</label><br>
            <textarea id="observaciones" v-model="observaciones" class="observaciones"></textarea>
          </td>
        </tr>
      </tfoot>
    </table>

    <div class="total-container">
      <div class="total">
        <strong>Total Factura:</strong> {{ totalFactura }} pesos
      </div>
      <div class="precio-pagar">
        <strong>Precio a pagar:</strong> {{ totalFactura }} pesos
      </div>
    </div>

    <div style="display: flex; justify-content: space-between; margin-top: 20px;">
      <!-- Datos del remitente -->
      <div>
        <h3>Remitente</h3>
        <label for="nombreRemitente">Nombre:</label>
        <input type="text" id="nombreRemitente" v-model="nombreRemitente">

        <label for="idRemitente">Identificación:</label>
        <input type="text" id="idRemitente" v-model="idRemitente">

        <label for="telefonoRemitente">Teléfono:</label>
        <input type="text" id="telefonoRemitente" v-model="telefonoRemitente">
      </div>

      <!-- Datos del destinatario -->
      <div>
        <h3>Destinatario</h3>
        <label for="nombreDestinatario">Nombre:</label>
        <input type="text" id="nombreDestinatario" v-model="nombreDestinatario">

        <label for="idDestinatario">Identificación:</label>
        <input type="text" id="idDestinatario" v-model="idDestinatario">

        <label for="telefonoDestinatario">Teléfono:</label>
        <input type="text" id="telefonoDestinatario" v-model="telefonoDestinatario">
      </div>

      <!-- Datos de la factura -->
      <div>
        <h3>Datos de la Factura</h3>
        <label for="numeroFactura">Número de Factura:</label>
        <input type="text" id="numeroFactura" v-model="numeroFactura" readonly>

        <label for="fechaFactura">Fecha de Factura:</label>
        <input type="date" id="fechaFactura" v-model="fechaFactura" readonly>

        <label for="vencimientoFactura">Vencimiento de Factura:</label>
        <input type="date" id="vencimientoFactura" v-model="vencimientoFactura" readonly>
      </div>
    </div>

    <iframe ref="pdfPreview" style="display: none; width: 40%; height: 400px;"></iframe>
    <button @click="previewPDF">Vista previa de la factura</button>
    <button @click="downloadPDF">Descargar factura</button>
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
      telefonoDestinatario: ''
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
      this.numeroFactura++;
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
      const filename = 'FACTURA.pdf';
      doc.save(filename);
      window.open(doc.output('bloburl'));
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
      const data = [
        ['Nombre', 'Cantidad', 'Precio', 'Total'],
        ...this.productos.map(producto => [
          producto.nombre,
          producto.cantidad,
          `${producto.precio.toLocaleString('es-ES')} pesos`,
          `${producto.total.toLocaleString('es-ES')} pesos`,
        ])
      ];

      const tableStartY = 60;

      doc.autoTable({
        startY: tableStartY,
        head: [data[0]],
        body: data.slice(1), 
        theme: 'grid'
      });

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
      doc.setFontSize(11);
      doc.text(this.observaciones, observacionesXPosition + 5, observacionesYPosition + 10, { maxWidth: observacionesWidth - 10 });
      const totalXPosition = 165;
      doc.text('Total:', totalXPosition, observacionesYPosition + observacionesHeight + 5);
      doc.text(`${this.totalFactura.toLocaleString('es-ES')} pesos`, totalXPosition, observacionesYPosition + observacionesHeight + 10);
      doc.text('Gracias por su compra', 110, observacionesYPosition + observacionesHeight + 20, { align: 'center', fontSize: 12 });

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
}
</script>

<style scoped>
.observaciones {
  background-color: aqua;
}

.total-container {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.total {
  background-color: #f2f2f2;
  padding: 10px;
}

.precio-pagar {
  background-color: #f2f2f2;
  padding: 10px;
}
</style>
