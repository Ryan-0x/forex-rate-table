<script setup>
import { ref, onMounted } from "vue";

const ranks = [
  "A",
  "2",
  "3",
  "4",
  "5",
  "6",
  "7",
  "8",
  "9",
  "10",
  "J",
  "Q",
  "K",
];
const suits = ["♦", "♣", "♥", "♠"];

const cards = ref([]);
const isShuffle = ref(false);

const players = 4;

const playerCards = ref({});

const results = ref({});
const winner = ref("");
function displayInitialDeck() {
  let id = 1;
  cards.value = [];

  for (let s = 0; s < suits.length; s++) {
    for (let r = 0; r < ranks.length; r++) {
      let card = {
        id: id,
        rank: ranks[r],
        suit: suits[s],
      };
      cards.value.push(card);
      id++;
    }
  }

  isShuffle.value = false;
  playerCards.value = {};
  for (let i = 1; i <= players; i++) {
    playerCards.value[`Player${i}`] = [];
  }
  winner.value = null;
  results.value = {};
}

function shuffle() {
  for (let i = cards.value.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [cards.value[i], cards.value[j]] = [cards.value[j], cards.value[i]];
  }

  isShuffle.value = true;
}

function distributeCards() {
  cards.value.forEach((card, index) => {
    playerCards.value[`Player${(index % players) + 1}`].push(card);
  });

  cards.value = [];
}

function calculateBestScoring() {
  const rankOrder = {
    2: 2,
    3: 3,
    4: 4,
    5: 5,
    6: 6,
    7: 7,
    8: 8,
    9: 9,
    10: 10,
    J: 11,
    Q: 12,
    K: 13,
    A: 14,
  };

  const suitOrder = { "♣": 1, "♦": 2, "♥": 3, "♠": 4 };

  let highestRankCount = 0;
  let winningRank = null;
  let winningPlayer = null;

  const bestScoring = {};

  Object.keys(playerCards.value).forEach((player) => {
    const rankCount = {};

    playerCards.value[player].forEach((card) => {
      const rank = card.rank;
      if (!rankCount[rank]) {
        rankCount[rank] = 0;
      }
      rankCount[rank]++;
    });

    const playerMaxRankCount = Math.max(...Object.values(rankCount));
    const playerHighestRank = Object.keys(rankCount).reduce((a, b) =>
      rankOrder[a] > rankOrder[b] ? a : b
    );

    bestScoring[player] = {
      highestRank: playerHighestRank,
      rankCount: playerMaxRankCount,
      highestSuit: playerCards.value[player].find(
        (card) => card.rank === playerHighestRank
      ).suit,
    };

    if (
      playerMaxRankCount > highestRankCount ||
      (playerMaxRankCount === highestRankCount &&
        rankOrder[playerHighestRank] > rankOrder[winningRank])
    ) {
      highestRankCount = playerMaxRankCount;
      winningRank = playerHighestRank;
      winningPlayer = player;
    } else if (
      playerMaxRankCount === highestRankCount &&
      rankOrder[playerHighestRank] === rankOrder[winningRank]
    ) {
      const playerHighestSuit = playerCards.value[player].find(
        (card) => card.rank === playerHighestRank
      ).suit;
      const currentWinningHighestSuit = playerCards.value[winningPlayer].find(
        (card) => card.rank === winningRank
      ).suit;

      if (suitOrder[playerHighestSuit] > suitOrder[currentWinningHighestSuit]) {
        winningPlayer = player;
      }
    }
  });
  results.value = bestScoring;
  winner.value = winningPlayer;
  return { bestScoring, winningPlayer };
}

onMounted(() => {
  displayInitialDeck();
  for (let i = 1; i <= players; i++) {
    playerCards.value[`Player${i}`] = [];
  }
});
</script>

<template>
  <header></header>
  <!-- {{ cards }} -->
  <div id="app">
    <div>
      <button
        class="font-bold py-2 px-4 rounded outline mx-2 my-2 text-red-500"
        v-if="isShuffle"
        @click="displayInitialDeck"
      >
        Reset
      </button>
      <button
        v-if="cards.length > 0"
        class="font-bold py-2 px-4 rounded outline mx-2 my-2"
        @click="shuffle"
      >
        Shuffle
      </button>
      <button
        v-if="cards.length > 0 && isShuffle"
        class="font-bold py-2 px-4 rounded outline mx-2 my-2 text-green-400"
        @click="distributeCards"
      >
        Distribute
      </button>

      <button
        v-if="cards.length == 0 && winner == null"
        class="font-bold py-2 px-4 rounded outline mx-2 my-2 text-green-400"
        @click="calculateBestScoring"
      >
        Calculate
      </button>
    </div>

    <TransitionGroup
      tag="div"
      class="ml-[30px] pt-[30px] transition-transform duration-[1s] top-0"
    >
      <div
        v-for="card in cards"
        :key="card.id"
        class="w-[50px] h-[80px] float-left mx-[5px] my-[5px] rounded-sm bg-slate-300"
      >
        {{ card.suit }}{{ card.rank }}
      </div>
    </TransitionGroup>
    <div
      class="flex"
      v-if="cards.length == 0"
      v-for="(cards, player) in playerCards"
      :key="player"
    >
      <h3>
        {{ player }}
        {{ winner ? (winner == player ? "-- Winner" : "-- Loser") : "" }}
      </h3>
      <ul>
        <div
          v-for="card in cards"
          :key="card.id"
          class="w-[50px] h-[80px] float-left mx-[5px] my-[5px] rounded-sm bg-slate-300"
        >
          {{ card.suit }}{{ card.rank }}
        </div>
        <p>
          {{ results[`${player}`] }}
        </p>
      </ul>
    </div>
  </div>
</template>

<style scoped>
html,
body,
#app {
  height: 100%;
  background: white;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
