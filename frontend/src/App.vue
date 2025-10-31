<template>
  <!-- vista de lobby -->
  <div v-if="vista === 'lobby'">
    <div v-if="isConnected === false">
      <input type="text" v-model="jugador.name" placeholder="Introdueix nom"/>
      <button @click="tryConn">Entra</button>
      <p>Type Racer Royale</p>
    </div>
    <div v-else>
      <p>Benvingut {{ jugador.value.name }} en aquesta partida tens el rol de {{ jugador.value.rol }} <!--El rol--></p>
      <!-- Llista pel admin-->
      <ul v-if="isAdmin">
        <li v-for="player in jugadors" :key="player.id">
          <button v-if="player.id > 0" @click="setAdmin(player.id)">
            <!--Logo estrella buit-->
          </button>
          {{ player.name }}
          <button v-if="player.id > 0" @click="deletePlayer(player.id)">
            <!--Logo creu-->
          </button>
          <div class="estat" v-bind:class="player.state === 'ready' ? 'ready' : 'notReady'"></div>
        </li>
      </ul>
      <!-- Llista per jugador / espectador -->
      <ul v-else>
        <li v-for="player in jugadors" :key="player.id">
          {{ player.name }}
          <div class="estat" v-bind:class="player.state === 'ready' ? 'ready' : 'notReady'"></div>
        </li>
      </ul>
      <!--Botons-->
      <button v-if="isAdmin" v-bind:class="isMajority ? '' : 'disabled'" @click="startGame">
        { }
      </button>
    </div>
  </div>


  <!-- vista de joc -->
  <div v-else-if="vista === 'game'">
    <div id="jugador" v-if="!esEspectador">
      
      <!-- Div on mostrem la informació de la partida (els textos)-->
        <div id="partida">
          <!--Truquem al game Engine i enviem les props que rebrà aquest component-->
          <GameEngine 
            :socket="socket"
            :jugador="jugador" 
            :esEspectador="esEspectador"
          />

        </div>
      <!--Fem Servir el component temps Restant per mostrar el temps -->
        <TempsRestant 
          :tempsInicial="tempsDePartida"
          @tempsAcabat="acabarPartida"
        />

      <!--Div on llistem els usuaris de la partida i els accerts i errors d'aquests-->
        <div id="ranquing">
          <!-- Aquí anirà el rànquing -->
        </div>
    </div>

    <div id="espectador" v-else>
        <h1>ETS ESPECTADOR, ESPERA A QUE ACABIN LA PARTIDA</h1>
    </div>    
  </div>


  <!-- vista de endgame-->
   <div v-else-if="vista === 'endGame'">
      <h1>Partida acabada!</h1>
      <!-- Aquí anirien els resultats -->
   </div>

</template>

<script setup>
//imports
  import { ref } from 'vue';
  // Importem els dos components del joc
  import GameEngine from './components/Game/GameEngine.vue'; 
  import TempsRestant from './components/Game/TempsRestant.vue';

//variables
  const vista = ref('lobby');
  const isConnected = ref(false);
  const jugador = ref({name: '', rol: '', state: 'notReady'})
  const jugadors = ref([]);
  const isAdmin = ref(true); // Canvia a 'false' per provar com a jugador
  const isMajority = ref(true); // Canvia a 'false' per desactivar el botó

  //-- VARIABLES DEL GAME --
  const esEspectador = ref(false); //Aquesta variable controla si el jugador està jugant o si es espectador
  const tempsDePartida = ref(0); //Aquesta variable guarda el temps que dura una partida
  
  //Rebem que el joc s'ha iniciat i executem la funcio comencarElJoc
  if (socket) {
    socket.on("JocIniciat", (dadesJoc) => {
        console.log("Rebut 'JocIniciat' amb: ", dadesJoc);
        comencarElJoc(dadesJoc)
    });
  }


//funcions
  function tryConn(){
    var objJugador = {...jugador.value, id: 0, err: 0, frases: 0, rol:''};
    jugadors.value.push(objJugador);
    
    const jugadorTrobat = jugadors.value.find((jug) => jug.name === objJugador.name);
    if (jugadorTrobat) {
      jugador.value = jugadorTrobat; 
    }
    
    isConnected.value = true;
  }

  /*---- FUNCIONS QUE UTILIZAREM AL GAME ----*/
  function comencarElJoc(dadesJoc){
    const llistaJugadors = dadesJoc.jugadores;
    
    //Guardem el temps d'inici de la partida en una variable
    tempsDePartida.value = dadesJoc.temps; 

    console.log("Rebut 'JocIniciat' amb la llista:", llistaJugadors );
    console.log("Durada de la partida: ", tempsDePartida.value, "s");

    //Cambiem la vista del joc
    vista.value='game';

    //Busquem quin rol té el jugadorActual
    const jugadorActual = llistaJugadors.find(p => p.id === jugador.value.id);

    //Si és jugador mostrem el joc si no mostrem el h1 del espectador
    if(jugadorActual.rol === 'jugador'){
      esEspectador.value = false;
      console.log("Rol assignat: JUGADOR");
    }else{
      esEspectador.value = true;
      console.log("Rol assignat: ESPECTADOR");
    }
  }

  //Funcio que es crida en acabar la partida
  function acabarPartida(){
    //Mostrem per consola que s'ha acabat el temps de la partida
    console.log("Temps Acabat! Mostrem els resultats")

    //Cambiem la vista a endGame
    vista.value = 'endGame';

    //Enviem al servidor amb un emit que la partida s'ha acabat
    socket.emit('partidaAcabada');

  }

  function setAdmin(id) {
    console.log("Fer admin a:", id);
  }
  function deletePlayer(id) {
    console.log("Esborrar jugador:", id);
  }
  function startGame() {
    console.log("Començar partida");
  }
</script>

<style scoped>
  .estat{
    max-width: 60%;
    width: 30px;
    height: auto;
  }
  .ready{
    background-color: greenyellow;
  }

  .notReady{
    background-color: red;
  }
</style>

