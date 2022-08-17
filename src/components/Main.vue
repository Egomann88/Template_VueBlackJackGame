<template>
  <section class="border-2 border-black px-12 py-20">
    <div class="sm:flex">
      <!-- dealer -->
      <div class="mt-4 sm:m-4">
        <h2>Dealer: <span v-if="!canHit">{{ dealerPts }}</span><span v-else>{{ dealerPts - dealerHiddenValue }}</span>
        </h2>
        <div class="flex">
          <img :src="dealerHiddenCardImgSrc" alt="" height="175" width="125" class="p-1" />
          <img v-for="i in dealerCardImgSrc.length" :key="i" :src="dealerCardImgSrc[i - 1]" alt="" height="175"
            width="125" class="p-1" />
          <!-- cards -->
        </div>
      </div>

      <!-- Player -->
      <div class="mt-4 sm:m-4">
        <h2>Player: {{ playerPts }}</h2>
        <div class="flex">
          <img v-for="i in playerCardImgSrc.length" :key="i" :src="playerCardImgSrc[i - 1]" alt="" height="175"
            width="125" class="p-1">
          <!-- cards -->
        </div>
      </div>
    </div>
    <!-- result -->
    <div class="mt-4">
      <button @click="hit();" :disabled="!canHit" :class="!canHit ? 'disabled' : ''"
        class="btn w-full sm:w-1/4 m-2">Hit</button>
      <button @click="double();" :disabled="true" :class="true ? 'disabled' : ''"
        class="btn w-full sm:w-1/4 m-2">Double</button>
      <button @click="stay()" class="btn w-full sm:w-1/4 m-2">Stay</button>
      <h3>{{ result }}</h3>
    </div>
  </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  data() {
    let dealerPts: number = 0,
      playerPts: number = 0,
      dealerAceCount: number = 0,
      playerAceCount: number = 0,
      dealerHiddenValue: number = 0,
      playerCardsOnField: number = 0;
    let cardImgTopSrc: string = "/src/assets/cards/",
      dealerHiddenCardImgSrc: string = "/src/assets/cards/BACK.png",
      result: string = "Ongoing";
    let dealerCardImgSrc: string[] = [],
      playerCardImgSrc: string[] = [],
      deck: string[] = [];

    let hidden: any = "";
    let canHit: boolean = true;

    return {
      dealerPts,
      playerPts,
      dealerAceCount,
      playerAceCount,
      dealerHiddenValue,
      playerCardsOnField,
      cardImgTopSrc,
      dealerHiddenCardImgSrc,
      result,
      dealerCardImgSrc,
      playerCardImgSrc,
      deck,
      hidden,
      canHit,
    }
  },
  setup() { },
  mounted() {
    this.buildDeck();
    this.shuffleDeck();
    this.startGame();
  },
  methods: {
    /**
     * filles the player's deck with all cards
     */
    buildDeck() {
      const cardValues = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"];
      const categories = ["C", "D", "H", "S"];

      for (let i = 0; i < categories.length; i++) {
        for (let j = 0; j < cardValues.length; j++) {
          this.deck.push(cardValues[j] + "-" + categories[i]); // A-C, A-D,...
        }
      }
    },

    /**
     * Shuffles all card in the players deck randomly
     */
    shuffleDeck() {
      for (let i = 0; i < this.deck.length; i++) {
        let j = Math.round(Math.random() * this.deck.length);
        let temp = this.deck[i];
        this.deck[i] = this.deck[j];
        this.deck[j] = temp;
      }
    },

    startGame() {
      let i: number = 0;
      let card: string | undefined = "";

      this.hidden = this.deck.pop();
      // reckon with the hidden value right from the start
      this.dealerHiddenValue += this.getValue(this.hidden);
      this.dealerHiddenValue += this.checkAce(this.hidden);
      this.dealerPts += this.getValue(this.hidden);
      this.dealerAceCount += this.checkAce(this.hidden);

      while (this.dealerPts < 17) {
        //<img>
        card = this.deck.pop(); // cut first value

        if (card == undefined) {
          console.error("card is undefined", card);
          return 0;
        }

        this.dealerCardImgSrc[i] = this.cardImgTopSrc + card + ".png";  // add cardSrc
        this.dealerPts += this.getValue(card);  // adds value of card to dealers points
        this.dealerAceCount = this.checkAce(card);  // check if aces exists
        i++;
      }

      console.log(this.hidden);
      console.log(this.dealerPts);

      for (i = this.playerCardsOnField; i < 2; i++) { // player begins with two cards
        card = this.deck.pop(); // cut first value

        if (card == undefined) {
          console.error("card is undefined", card);
          return 0;
        }

        this.playerCardImgSrc[i] = this.cardImgTopSrc + card + ".png";  // add cardSrc
        this.playerPts += this.getValue(card);  // adds value of card to dealers points
        this.playerAceCount = this.checkAce(card);  // check if aces exists
        this.playerCardsOnField = i + 1;
      }
    },

    getValue(card: string) {
      let data = card.split("-");
      let value = data[0];

      switch (value) {
        case "A":
          return 11;
        case "J":
        case "Q":
        case "K":
          return 10;
      }

      return parseInt(value);
    },

    checkAce(card: string) {
      if (card[0] == "A") {
        return 1;
      }
      return 0;
    },

    reduceAce(playerPts: number, playerAceCount: number) {
      while (playerPts > 21 && playerAceCount > 0) {
        playerPts -= 10;
        playerAceCount -= 1;
      }

      return playerPts;
    },

    hit() {
      if (!this.canHit) {
        alert('Sie können nicht mehr weiterspielen.');
        return 0;
      }

      let card: string | undefined = this.deck.pop(); // cut first value

      if (card == undefined) {
        console.error("card is undefined", card);
        return 0;
      }

      this.playerCardImgSrc[this.playerCardsOnField++] = this.cardImgTopSrc + card + ".png";  // add cardSrc
      this.playerPts += this.getValue(card);  // adds value of card to dealers points
      this.playerAceCount = this.checkAce(card);  // check if aces exists

      if (this.reduceAce(this.playerPts, this.playerAceCount) > 21) {
        this.canHit = false;
      }
    },

    double() {
      if (!this.canHit) {
        alert('Sie können nicht mehr weiterspielen.');
        return 0;
      }

      let card: string | undefined = this.deck.pop(); // cut first value

      if (card == undefined) {
        console.error("card is undefined", card);
        return 0;
      }

      this.playerCardImgSrc[this.playerCardsOnField++] = this.cardImgTopSrc + card + ".png";  // add cardSrc
      this.playerPts += this.getValue(card);  // adds value of card to dealers points
      this.playerAceCount = this.checkAce(card);  // check if aces exists

      if (this.reduceAce(this.playerPts, this.playerAceCount) > 21) {
        this.canHit = false;
      }
    },

    stay() {
      this.dealerPts = this.reduceAce(this.dealerPts, this.dealerAceCount);
      this.playerPts = this.reduceAce(this.playerPts, this.playerAceCount);

      this.canHit = false;

      this.dealerHiddenCardImgSrc = this.cardImgTopSrc + this.hidden + ".png"; // hidden img
      // console.log("stay");

      let msg: string = "";
      if (this.playerPts > 21) {
        msg = "You Lose!";
      } else if (this.dealerPts > 21) {
        msg = "You Win!";
      } else if (this.playerPts == this.dealerPts) {
        msg = "Tie!";
      } else if (this.playerPts > this.dealerPts) {
        msg = "You Win!";
      } else if (this.playerPts < this.dealerPts) {
        msg = "You Lose!";
      } else {
        console.error("Unknown Error");
      }

      this.result = msg;
    },
  },
});
</script>

<style scoped>
</style>