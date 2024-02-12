<template>
  <div>
    <h1>Generador de Factura</h1>
    <factura-component></factura-component>
  </div>
</template>

<script>
import FacturaComponent from './components/FacturaComponent.vue';

export default {
  components: {
    FacturaComponent
  }
};
</script>

<style>
h{
  position: relative;
  text-align: center;
}
</style>
