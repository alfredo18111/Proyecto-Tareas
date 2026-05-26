<script setup>
import TableroKanban from './components/TableroKanban.vue';
</script>

<template>
 <div class="app-container">
 <main class="content">
  <TableroKanban/>
 </main>
 </div>
</template>

<style>
/*Reset global para quitar margenes raros del navegador */
*{
  box-sizing: border-box;
}
body{
  margin: 0;
  padding: 0;
  background-color: #121212; /*fondo oscuro elegante*/
  color: white;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
.app-container {
  width: 100%p;
  min-height: 100vh;
  display: flex;
}
.content {
  flex-grow: 1;
  padding: 40px;
  display: flex;
  justify-content: center; /*Centra el tablero en horizontal*/
  align-items: flex-start; /*Hace que empiece desde arriba de la pantalla*/

}

</style>
