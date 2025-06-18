<script setup>
import { ref, reactive } from 'vue';

let joueur = ref('chat');
let gagnant = ref('');
let positionPiece = ref(0);

const tableau = reactive([
  ['', '', '', '', '', '', ''],
  ['', '', '', '', '', '', ''],
  ['', '', '', '', '', '', ''],
  ['', '', '', '', '', '', ''],
  ['', '', '', '', '', '', ''],
  ['', '', '', '', '', '', ''],
]);

function mouvement_tableau(event) {
  positionPiece.value = event.pageX - 30;
}

function click_td(ligne, colonne) {
  if (gagnant.value) return; // bloque le jeu après victoire

  let ligneLibre = undefined;
  for (let l = 5; l >= 0; l--) {
    if (tableau[l][colonne] === '') {
      ligneLibre = l;
      break;
    }
  }
  if (ligneLibre === undefined) return;

  tableau[ligneLibre][colonne] = joueur.value;

  if (verifier_gagner(joueur.value, ligneLibre, colonne)) {
    gagnant.value = joueur.value;
    return;
  }

  joueur.value = joueur.value === 'chat' ? 'souris' : 'chat';
}

function verifier_gagner(j, ligne, colonne) {
  const directions = [[0, 1], [1, 0], [1, 1], [1, -1]];
  for (const [dx, dy] of directions) {
    let count = 1;

    // Avant
    let x = colonne + dx;
    let y = ligne + dy;
    while (x >= 0 && x < 7 && y >= 0 && y < 6 && tableau[y][x] === j) {
      count++;
      x += dx;
      y += dy;
    }

    // Arrière
    x = colonne - dx;
    y = ligne - dy;
    while (x >= 0 && x < 7 && y >= 0 && y < 6 && tableau[y][x] === j) {
      count++;
      x -= dx;
      y -= dy;
    }

    if (count >= 4) return true;
  }
  return false;
}

function rejouer() {
  for (let l = 0; l < 6; l++) {
    for (let c = 0; c < 7; c++) {
      tableau[l][c] = '';
    }
  }
  joueur.value = 'chat';
  gagnant.value = '';
}
</script>

<template>
  <div id="jeu">
    <div id="joueur-chat" :class="{ joueur: true, actif: joueur === 'chat' }">
      <h2>Chat</h2>
      <img src="./assets/piece-chat.svg" />
    </div>

    <img
      id="piece-active"
      :style="'left:' + positionPiece + 'px'"
      :src="'/src/assets/piece-' + joueur + '.svg'"
    />

    <table @mousemove="mouvement_tableau">
      <tr v-for="ligne in 6" :key="ligne">
        <td v-for="colonne in 7" :key="colonne" @click="click_td(ligne - 1, colonne - 1)">
          <Transition name="trans-piece">
            <img
              v-if="tableau[ligne - 1][colonne - 1] !== ''"
              :src="'/src/assets/piece-' + tableau[ligne - 1][colonne - 1] + '.svg'"
            />
          </Transition>
        </td>
      </tr>
    </table>

    <div id="joueur-souris" :class="{ joueur: true, actif: joueur === 'souris' }">
      <h2>Souris</h2>
      <img src="./assets/piece-souris.svg" />
    </div>
  </div>

  <!-- Animation de victoire -->
  <Transition name="fade-slide">
    <div v-if="gagnant" class="popup">
      <p>{{ gagnant }} a gagné !</p>
      <button @click="rejouer">Rejouer</button>
    </div>
  </Transition>

  <!-- Bouton rejouer si pas encore gagné -->
  <div v-if="!gagnant" style="text-align:center; margin-top: 20px;">
    <button @click="rejouer">Réinitialiser</button>
  </div>
</template>

<style scoped>
#jeu {
  display: flex;
  margin-top: 100px;
  justify-content: center;
}

#piece-active {
  position: absolute;
  left: 100px;
  top: 20px;
}

.joueur {
  width: 100px;
  height: 200px;
  margin: 10px;
  margin-top: 100px;
  background-color: blue;
  color: white;
  border-radius: 10px;
  font-family: sans;
  padding: 0.5em;
  text-align: center;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.5);
}

.joueur.actif {
  background-color: #55f;
}

.joueur h2 {
  font-size: 18px;
}

table {
  padding: 25px;
  padding-top: 27px;
  padding-bottom: 23px;
  background-image: url(assets/cadre.svg);
  border-collapse: separate;
  border-spacing: 0;
}

td {
  text-align: center;
  width: 68px;
  height: 68px;
}

table img {
  position: relative;
  transition: top 0.5s ease-in;
  top: 0;
  z-index: -1;
}

#jeu tr:nth-child(1) .trans-piece-enter-from {
  top: -110px;
}
#jeu tr:nth-child(2) .trans-piece-enter-from {
  top: -180px;
}
#jeu tr:nth-child(3) .trans-piece-enter-from {
  top: -250px;
}
#jeu tr:nth-child(4) .trans-piece-enter-from {
  top: -320px;
}
#jeu tr:nth-child(5) .trans-piece-enter-from {
  top: -390px;
}
#jeu tr:nth-child(6) .trans-piece-enter-from {
  top: -460px;
}

button {
  font-size: 16px;
  padding: 10px 20px;
  background-color: #337ab7;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  box-shadow: 2px 2px 3px rgba(0, 0, 0, 0.3);
  transition: background-color 0.3s ease;
}
button:hover {
  background-color: #286090;
}

.popup {
  position: fixed;
  top: 30%;
  left: 50%;
  transform: translateX(-50%);
  background: white;
  padding: 20px;
  border: 2px solid #55f;
  border-radius: 12px;
  text-align: center;
  z-index: 10;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.4);
  animation: appear 0.4s ease;
}
.popup p {
  font-size: 20px;
  margin-bottom: 10px;
}

/* Transition pour la popup */
.fade-slide-enter-active {
  animation: appear 0.4s ease forwards;
}
.fade-slide-leave-active {
  animation: disappear 0.3s ease forwards;
}
@keyframes appear {
  0% {
    opacity: 0;
    transform: translate(-50%, -10%);
  }
  100% {
    opacity: 1;
    transform: translate(-50%, 0%);
  }
}
@keyframes disappear {
  from {
    opacity: 1;
    transform: translate(-50%, 0%);
  }
  to {
    opacity: 0;
    transform: translate(-50%, -10%);
  }
}
</style>
