<script setup>
import Holoculator from './components/Holoculator.vue';
</script>

<template>
  <main>
    <Holoculator />
  </main>
</template>

<style>
/* Reset de standaard browser-marges voor een naadloze look */
html, body {
  margin: 0;
  padding: 0;
  overflow: hidden; /* Zorgt dat er geen scrollbalken verschijnen */
}

/* DE ACHTERGROND STIJLEN STAAN NU HIER */
#app {
  width: 100vw; /* Volledige breedte van het scherm */
  height: 100vh; /* Volledige hoogte van het scherm */
  
  /* Donkere, futuristische achtergrondkleur */
  background-color: #0c0a1a; 
  
  /* Creëert een grid met CSS gradients */
  background-image: 
    linear-gradient(rgba(56, 189, 248, 0.1) 1px, transparent 1px),
    linear-gradient(90deg, rgba(56, 189, 248, 0.1) 1px, transparent 1px);
  background-size: 50px 50px;
  
  /* Animatie die de grid laat bewegen */
  animation: moveGrid 10s linear infinite;
  
  position: relative; /* Nodig voor het ::before pseudo-element */
  overflow: hidden;
}

/* Het gloed-effect is nu ook op #app geplaatst */
#app::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 700px;
  height: 700px;
  background: radial-gradient(circle, rgba(168, 85, 247, 0.15), transparent 70%);
  transform: translate(-50%, -50%);
  filter: blur(50px);
  z-index: 0; /* Zorgt dat de gloed achter de content blijft */
}

/* De <main> tag is nu simpeler en alleen voor de layout */
main {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

@keyframes moveGrid {
  from {
    background-position: 0 0;
  }
  to {
    background-position: 50px 50px;
  }
}
</style>