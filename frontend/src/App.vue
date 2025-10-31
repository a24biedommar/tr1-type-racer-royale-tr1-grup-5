<template>
  <router-view />
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
      <!--Div on mostrem el temps restant de la partida-->
        <div id="tempsRestant">
          <h1>Temps Restant:{{tempsRestant}}s</h1>
        </div>
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

   </div>

</template>

<script setup>
//imports
  import { ref } from 'vue';
  // Importem el nostre nou component de joc
  import GameEngine from './components/Game/GameEngine.vue'; 

//variables
  const vista = ref('lobby');
  const isConnected = ref(false);
  const jugador = ref({name: '', rol: '', state: 'notReady'})
  const jugadors = ref([]);

  //-- VARIABLES DEL GAME --
  const esEspectador = ref(false); //Aquesta variable controla si el jugador està jugant o si es espectador
  const tempsRestant = ref(0);//El temps que es mostrarà per pantalla
  let timerInstance = null;//Variable per guardar l'interval
  
  socket.on("JocIniciat", (dadesJoc) => {
      console.log("Rebut 'JocIniciat' amb: ", dadesJoc);
      comencarElJoc(dadesJoc)
  });


//funcions
  function tryConn(){
    var objJugador = {...jugador.value, id: 0, err: 0, frases: 0, rol:''};
    jugadors.value.push(objJugador);
    
    // 2. Fes servir .find() (retorna objecte) en lloc de .filter() (retorna array)
    const jugadorTrobat = jugadors.value.find((jug) => jug.name === objJugador.name);
    if (jugadorTrobat) {
      // Assignem l'objecte trobat (amb id, etc.) al nostre ref
      jugador.value = jugadorTrobat; 
    }
    
    isConnected.value = true;
  }

  /*---- FUNCIONS QUE UTILIZAREM AL GAME ----*/
  function comencarElJoc(dadesJoc){
    const llistaJugadors = dadesJoc.jugadores;
    const tempsDePartida = dadesJoc.temps;

    console.log("Rebut 'JocIniciat' amb la llista:", llistaJugadors );
    console.log("Durada de la partida: ", tempsDePartida, "s");

    vista.value='game';

    const jugadorActual = llistaJugadors.find(p => p.id === jugador.value.id);

    if(jugadorActual && jugadorActual.rol === 'jugador'){
      esEspectador.value = false;
      console.log("Rol assignat: JUGADOR");
    }else{
      esEspectador.value = true;
      console.log("Rol assignat: ESPECTADOR");
    }

    iniciarComptador(tempsDePartida);
  }

  function iniciarComptador(tempsInici){
    tempsRestant.value = tempsInici; 
    timerInstance = setInterval(() =>{
      if(tempsRestant.value > 0){
        tempsRestant.value--;
      } else{
        acabarPartida()
      }
    }, 1000);
  }

  function acabarPartida(){
    clearInterval(timerInstance);
    timerInstance = null;

    console.log("Temps Acabat! Mostrem els resultats")

    vista.value = 'endGame';

    if (socket) {
      socket.emit('partidaAcabada');
    }
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
