# Currency Converter

Application web Vue.js 3 permettant de convertir un montant entre plusieurs devises
avec un clavier numerique integre.

## Fonctionnalites

- Conversion entre USD, EUR, TND, GBP, JPY et CAD avec l'euro comme devise pivot.
- Saisie par le clavier de l'application ou par le clavier physique.
- Inversion des devises source et cible.
- Controle du point decimal unique et limitation a 10 caracteres.
- Conversion automatique lors de la saisie ou du changement de devise.
- Chargement des taux reels via l'API Frankfurter, avec repli sur les taux fixes
  du TP si le reseau est indisponible.
- Interface responsive utilisable sur un ecran de 360 px.

## Installation

Depuis le dossier du projet :

```bash
npm install
```

## Lancer en developpement

```bash
npm run dev
```

Vite affiche ensuite l'adresse locale de l'application dans le terminal.

## Construire pour la production

```bash
npm run build
```

Le projet utilise Vue 3, Vite et du CSS ecrit sans bibliotheque d'interface.
