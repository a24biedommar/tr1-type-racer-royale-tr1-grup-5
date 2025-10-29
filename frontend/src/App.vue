<template>
  <router-view />
  <!-- vista de lobby -->
  <div v-if="vista === 'lobby'">
    <div v-if="isConnected === false">
      <input type="text" v-model="jugador" placeholder="Introdueix nom"/>
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
          
        </div>
      <!--Div on mostrem el temps restant de la partida-->
        <div id="tempsRestant">

        </div>
      <!--Div on llistem els usuaris de la partida i els accerts i errors d'aquests-->
        <div id="llistaUsers">

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
  //import GameEngine from './components/Game/GameEngine.vue';

//variables
  const vista = ref('lobby');
  const isConnected = ref(false);
  const isMajority = ref(jugadors.value.filter(player => player.state === 'ready').length >= Math.round(jugadors.value.length/2))
  const isAdmin = ref((jugador.value.rol === 'admin'))
  const jugador = ref({name: '', rol: '', state: 'notReady'}) //rol: 'ready' | 'notReady'
  const jugadors = ref([]);

  var socket = null;
  
//funcions
  /*la idea es que intenti fer la connexio per socket i si funciona, especificar
    la variable jugador com a objecte sencer i jugadors en general també amb obj*/
  function tryConn(){
    //socket intenta connectar amb el server i rep el llistat actual de jugadors
    var objJugador = {...jugador, id: 0, err: 0, frases: 0, rol:''};
    jugadors.value.push(objJugador);
    jugador.value = jugadors.value.filter((jug) => jug.name === jugador.value.name);
    isConnected.value = true;
    //sino alert
  }

  function setAdmin(id){
    /*socket intenta canviar rol admin actiu per jugador i jugador especific per admin
    i sobreescriu al servidor intercanviant l'id d'ambdos jugadors*/
    console.log(id)
  }

  function deletePlayer(id){
    /*socket intenta eliminar jugador forçant-li desconnection i s'actualitza la llista de
    jugadors al servidor*/
    console.log(id)
  }

  function startGame(){
    /*compte enrrere de 5 segons, si es compleix sense interrupcions, envia comença el joc */
  }

  /*---- FUNCIONS I VARIABLES QUE UTILIZAREM AL GAME ----*/
  //1.VARIABLES
  const esEspectador = ref(false); //Aquesta variable controla si el jugador està jugant o si es espectador

  //2.FUNCIONS
  /*Äquesta funció s'executa quan el servidor emet 'JocIniciat' 
  Rep la llista de jugadors que si que jugaran la partida*/
  function comencarElJoc(llistaJugadors){
    console.log("Rebut 'JocIniciat' amb la llista:", llistaJugadors );

    //Canviem la vista general de 'lobby' a 'game'
    vista.value='game';

    //Mirem el rol del jugador que ens envia el servidor
    const rolJugador = llistaJugadors.find(p => p.rol ===jugador.value.rol);

    //Un cop tenim el rol del jugador gestionem si és espectador o si és jugador
    if(rolJugador === 'jugador'){
      esEspectador.value = false;
      console.log("Rol assignat: JUGADOR");
    }else{
      esEspectador.value = true;
      console.log("Rol assignat: ESPECTADOR");
    }
    //TODO: DECLARAR EL SOCKET.ON('JOCINICIAT', LLISTAJUGADORS)
  }
  //TODO: FALTA TRUCAR A LA VARIABLE TEMPS I GESTIONAR EL COOLDOWN DEL TEMPS, QUAN EL TEMPS ARRIBA A 0 CANVIA DE VISTA
  //TODO: FALTA QUE EL USUARI PUGI JUGAR
  //TODO: FALTA LLISTAR ELS JUGADORS I ELS ERRORS O ACCERTS QUE FAIGIN
  
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
