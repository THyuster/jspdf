<template>
  <div>
    <form @submit.prevent="agregarProducto">
      <label for="nombre">Nombre del Producto:</label>
      <input type="text" id="nombre" v-model="nombreProducto" >

      <label for="cantidad">Cantidad:</label>
      <input type="number" id="cantidad" v-model.number="cantidadProducto" required>

      <label for="precio">Precio Unitario:</label>
      <input type="number" id="precio" v-model.number="precioProducto" required>

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
          <td>{{ producto.precio }}</td>
          <td>{{ producto.total }}</td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3" style="text-align: right;"><strong>Total Factura:</strong></td>
          <td>{{ totalFactura }}</td>
        </tr>
        <tr>
          <td colspan="4">
            <label for="observaciones">Observaciones:</label><br>
            <textarea id="observaciones" v-model="observaciones" class="observaciones"></textarea>
          </td>
        </tr>
      </tfoot>
    </table>

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
      precioProducto: 0,
      productos: [],
      observaciones: '',
      numeroFactura: 1
    };
  },
  computed: {
    totalFactura() {
      return this.productos.reduce((total, producto) => total + producto.total, 0);
    }
  },
  methods: {
    agregarProducto() {
      const total = this.cantidadProducto * this.precioProducto;
      this.productos.push({
        nombre: this.nombreProducto,
        cantidad: this.cantidadProducto,
        precio: this.precioProducto,
        total: total
      });
      this.numeroFactura++;
      this.nombreProducto = '';
      this.cantidadProducto = 0;
      this.precioProducto = 0;
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
      const pageSize = { width: 216 , height: 279  };
      const doc = new jsPDF({ format: [pageSize.width, pageSize.height] });

      // Agregar imagen, texto y línea de encabezado
      const img = new Image();
      img.src = require('@/assets/jj.jpg'); 
      doc.addImage(img, 'PNG', 8, 5, 25, 25);
      doc.text('Yuster', 30, 17);
      doc.text('Nit: 147852369', 30, 22);
      doc.line(10, 30, 140, 30);

      // Agregar título y dirección
      doc.setFontSize(10);
      doc.text('YUSCORP', 10, 35);
      doc.text('Dirección: Calle 123, Ciudad', 10, 39);
      doc.text('Teléfono: 123-456-789', 10, 43);

      //fecha y hora
      const numeroFactura = `N° Factura: ${this.numeroFactura}`;
      const now = new Date();
      const fechaHoraActual = `${now.toLocaleDateString()} ${now.toLocaleTimeString()}`;
      const maxWidth = doc.internal.pageSize.width;
      const textWidth = doc.getStringUnitWidth(numeroFactura) * doc.internal.getFontSize();
      const xPosition = maxWidth - textWidth - -10; 

      // fecha y hora posicion
      doc.text(numeroFactura, xPosition, 38);
      doc.text(fechaHoraActual, xPosition, 43);

      //tabla de datos de factura
      const data = [
        ['Nombre', 'Cantidad', 'Precio', 'Total'],
        ...this.productos.map(producto => [
          producto.nombre,
          producto.cantidad,
          producto.precio.toLocaleString('es-ES', { style: 'currency', currency: 'COP' }),
          producto.total.toLocaleString('es-ES', { style: 'currency', currency: 'COP' }),
        ])
      ];

      const tableStartY = 50;

      doc.autoTable({
        startY: tableStartY,
        head: [data[0]],
        body: data.slice(1), 
        theme: 'grid'
      });

      // Agregar líneas en todas las páginas
      const agregarLineasEnTodasLasPaginas = () => {
        doc.setLineWidth(1);
        doc.line(0, 0, pageSize.width, 0); // Línea superior
        doc.line(0, pageSize.height, pageSize.width, pageSize.height); // Línea inferior
        doc.line(0, 0, 0, pageSize.height); // Línea izquierda
        doc.line(pageSize.width, 0, pageSize.width, pageSize.height); // Línea derecha
      };

      if (doc.autoTable.previous.finalY > pageSize.height - 10) {
        doc.addPage();
        agregarLineasEnTodasLasPaginas();
      }

      if (doc.autoTable.previous.finalY > pageSize.height - 10) {
        doc.addPage();
        agregarLineasEnTodasLasPaginas();
      }
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
      doc.text(this.totalFactura.toLocaleString('es-ES', { style: 'currency', currency: 'COP' }), totalXPosition, observacionesYPosition + observacionesHeight + 10);
      doc.text('Gracias por su compra', 110, observacionesYPosition + observacionesHeight + 20, { align: 'center', fontSize: 12 });

      return doc;
    }
  }
}
</script>

<style scoped>
.observaciones {
  background-color: aqua;
}
</style>
