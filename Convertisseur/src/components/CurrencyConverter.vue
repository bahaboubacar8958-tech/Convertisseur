<script setup>
import { onMounted, onUnmounted, ref, watch } from 'vue'
import { devises, taux } from '../data/taux'

const touches = ['7', '8', '9', '4', '5', '6', '1', '2', '3', 'C', '0', '.']
const montant = ref('0')
const deviseSource = ref('USD')
const deviseCible = ref('EUR')
const resultat = ref('')
const messageErreur = ref('')
const tauxActuels = ref({ ...taux })

// Convertit toujours en passant par l'euro, qui est la devise pivot.
function calculerConversion() {
  const valeur = Number.parseFloat(montant.value)

  if (!Number.isFinite(valeur) || valeur === 0) {
    resultat.value = ''
    return
  }

  const montantEnEuros = valeur / tauxActuels.value[deviseSource.value]
  resultat.value = (montantEnEuros * tauxActuels.value[deviseCible.value]).toFixed(2)
}

function appuyer(touche) {
  if (touche === 'C') {
    montant.value = '0'
    resultat.value = ''
    messageErreur.value = ''
    return
  }

  messageErreur.value = ''

  if (touche === '.' && montant.value.includes('.')) {
    return
  }

  const nouveauMontant =
    touche === '.'
      ? montant.value === '0' ? '0.' : `${montant.value}.`
      : montant.value === '0' ? touche : `${montant.value}${touche}`

  if (nouveauMontant.length <= 10) {
    montant.value = nouveauMontant
  }
}

function convertir() {
  if (Number.parseFloat(montant.value) === 0) {
    resultat.value = ''
    messageErreur.value = 'Veuillez saisir un montant'
    return
  }

  messageErreur.value = ''
  calculerConversion()
}

function inverser() {
  const ancienneSource = deviseSource.value
  deviseSource.value = deviseCible.value
  deviseCible.value = ancienneSource

  if (resultat.value) {
    calculerConversion()
  }
}

function gererToucheClavier(event) {
  const touche = event.key === 'Escape' ? 'C' : event.key

  if (touches.includes(touche)) {
    event.preventDefault()
    appuyer(touche)
  } else if (event.key === 'Enter') {
    event.preventDefault()
    convertir()
  }
}

// Le bonus API remplace les taux fixes uniquement si la reponse est exploitable.
async function chargerTauxReels() {
  try {
    const reponse = await fetch('https://api.frankfurter.app/latest?from=EUR')
    if (!reponse.ok) {
      throw new Error(`Erreur HTTP ${reponse.status}`)
    }

    const donnees = await reponse.json()
    const nouveauxTaux = { EUR: 1 }

    for (const devise of devises) {
      if (devise !== 'EUR' && typeof donnees.rates?.[devise] === 'number') {
        nouveauxTaux[devise] = donnees.rates[devise]
      }
    }

    if (devises.every((devise) => typeof nouveauxTaux[devise] === 'number')) {
      tauxActuels.value = nouveauxTaux
    }
  } catch (erreur) {
    console.warn('Impossible de charger les taux reels, utilisation des taux fixes.', erreur)
  }
}

// Toute saisie ou modification de devise declenche automatiquement la conversion.
watch([montant, deviseSource, deviseCible, tauxActuels], () => {
  messageErreur.value = ''
  calculerConversion()
})

onMounted(() => {
  window.addEventListener('keydown', gererToucheClavier)
  chargerTauxReels()
})

onUnmounted(() => {
  window.removeEventListener('keydown', gererToucheClavier)
})
</script>

<template>
  <main class="page">
    <section class="convertisseur" aria-labelledby="titre-convertisseur">
      <div class="zone-affichage">
        <h1 id="titre-convertisseur">Currency Converter</h1>

        <div class="ligne-devise">
          <label for="devise-source">Devise source</label>
          <select id="devise-source" v-model="deviseSource">
            <option v-for="devise in devises" :key="devise" :value="devise">
              {{ devise }}
            </option>
          </select>
        </div>

        <div class="montant" aria-label="Montant saisi">{{ montant }}</div>

        <button class="bouton-inverser" type="button" aria-label="Inverser les devises" @click="inverser">
          ⇅
        </button>

        <div class="ligne-devise">
          <label for="devise-cible">Devise cible</label>
          <select id="devise-cible" v-model="deviseCible">
            <option v-for="devise in devises" :key="devise" :value="devise">
              {{ devise }}
            </option>
          </select>
        </div>

        <div class="montant resultat" aria-live="polite">{{ resultat || '0.00' }}</div>

        <button class="bouton-convertir" type="button" @click="convertir">Convertir</button>
        <p v-if="messageErreur" class="message-erreur" role="alert">{{ messageErreur }}</p>
      </div>

      <div class="clavier" aria-label="Clavier numérique">
        <button
          v-for="touche in touches"
          :key="touche"
          class="touche"
          :class="{ 'touche-effacer': touche === 'C' }"
          type="button"
          @click="appuyer(touche)"
        >
          {{ touche }}
        </button>
      </div>
    </section>
  </main>
</template>

<style scoped>
.page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px 12px;
  box-sizing: border-box;
}

.convertisseur {
  width: min(100%, 360px);
  overflow: hidden;
  border-radius: 18px;
  background: #f7f8fa;
  box-shadow: 0 12px 30px rgb(31 68 82 / 18%);
}

.zone-affichage {
  position: relative;
  padding: 26px 24px 24px;
  color: white;
  background: #2bb8e8;
}

h1 {
  margin: 0 0 24px;
  text-align: center;
  font-size: 25px;
  font-weight: 700;
}

.ligne-devise {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 6px;
}

label {
  font-size: 14px;
  font-weight: 600;
}

select {
  min-width: 92px;
  padding: 7px 9px;
  border: 0;
  border-radius: 6px;
  color: #176b82;
  background: white;
  font: inherit;
  font-weight: 700;
}

.montant {
  min-height: 57px;
  margin: 0 0 5px;
  border-bottom: 2px solid rgb(255 255 255 / 85%);
  text-align: right;
  font-size: 48px;
  line-height: 1.2;
  overflow-wrap: anywhere;
}

.resultat {
  margin-top: 4px;
}

button {
  font: inherit;
  cursor: pointer;
}

.bouton-inverser {
  display: block;
  width: 42px;
  height: 32px;
  margin: 8px auto 10px;
  border: 1px solid rgb(255 255 255 / 70%);
  border-radius: 16px;
  color: white;
  background: rgb(255 255 255 / 14%);
  font-size: 21px;
  line-height: 1;
}

.bouton-convertir {
  display: block;
  width: 100%;
  margin-top: 18px;
  padding: 11px;
  border: 0;
  border-radius: 8px;
  color: #1684a4;
  background: white;
  font-weight: 700;
}

.message-erreur {
  margin: 9px 0 0;
  text-align: center;
  color: #fff2f2;
  font-size: 13px;
  font-weight: 700;
}

.clavier {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 4px;
  padding: 18px 20px 22px;
  background: #f7f8fa;
}

.touche {
  height: 48px;
  border: 0;
  color: #1ba6c9;
  background: transparent;
  font-size: 26px;
  font-weight: 700;
}

.touche:hover,
.touche:focus-visible {
  border-radius: 8px;
  background: #e4f6fa;
  outline: none;
}

.touche-effacer {
  color: #f0727a;
}

@media (max-width: 380px) {
  .page {
    padding: 8px;
  }

  .zone-affichage {
    padding-inline: 18px;
  }

  .montant {
    font-size: 42px;
  }
}
</style>
