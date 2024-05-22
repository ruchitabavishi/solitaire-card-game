<template>
  <div class="root">
    <div class="top-area">
        <div class="left-area">
          <div class="deck-container">
            <div class="card closed-card" @click="onDeckClick"></div>
            <div v-if="currentOpenCardInDec" class="card bg-white" draggable="true" @dragstart="handleDragCurrentOpenDeckCard()">
              <div class=" card-details" :class="`${currentOpenCardInDec.color}`">
                <div> {{ currentOpenCardInDec.suitSymbol}}</div>
                <div> {{currentOpenCardInDec.rank}} </div>
              </div>
            </div>
          </div>
        </div>
        <div class="right-area">
          <div v-for="card in pileCards" :key=card.suitSymbol class="card bg-white" @dragover.prevent :id="`${card.suit}`"
           @drop="handleDropForPile($event, target)">
              <div v-if="card.rank" class="card-details" :class="`${card.cardColor}`">
                <div> {{ card.suitSymbol}}</div>
                <div> {{card.rank}} </div>
              </div>
              <div v-else class="card-details" :class="`${card.cardColor}`" >{{card.suitSymbol}}</div>
          </div>
        </div>

    </div>

    <div class="bottom-container">
      <div v-for="row in totalRow" :key='row' class="pile-row">
        <div v-if="getPileDec(row).length == 0" class="card">

        </div>
            <div v-for="(card,i) in getPileDec(row)" :key='i' class="card "
              :id="`row-${row}`"
             :class="card.isOpen ? 'bg-white non-show-card': 'closed-card non-show-card'" 
             @drop="handleDropForRow($event, target)" @dragover.prevent
             @dragstart="handleDragStart(card, row)" draggable="true">
             <div v-if="card.isOpen" class="card-details" :class="`${card.color}`">
                <div> {{ card.suitSymbol}}</div>
                <div> {{card.rank}} </div>
              </div>
          </div>
      </div>
    </div>

  </div>  
</template>

<script>
  const cardRanking = ['A', '2', '3', '4', '5',
              '6', '7', '8', '9', '10', 'J', 'Q', 'K'];
  const cardSuits = ['spade', 'heart', 'diamond', 'club'];

  // 24 cards in deck
  // 28 card distributed

  export default {
      data() {
        return {
          currentOpenCardInDec: null,
          remaininDecNextIndex: 0,
          totalRow: [0,1,2,3,4,5,6],
          globalDeck: [],
          shuffledDeck: [],
          pileCards: [
            {
              suitSymbol:'♠',
              suit: 'spade',
              cardColor :'black',
              rank: null,
              higher: 'A'
            },
            {
              suitSymbol:'♥',
              suit: 'heart',
              cardColor :'red',
              rank: null,
              higher: 'A'
            },
            {
              suitSymbol:'♦',
              suit: 'diamond',
              cardColor :'red',
              rank: null,
              higher: 'A'
            },
            {
              suitSymbol:'♣',
              suit: 'club',
              cardColor :'black',
              rank: null,
              higher: 'A'
            },
          ],
          remainingDeck: [],
          pileDeck: {},
          draggindCard: null,
          draggingRow: null
        }
      },
      methods: {
        handleDropForRow(event, target) {
          console.log(event, target)
          const row = event.target.id
          this.checkIfcardCanBeDroppedOnRow(row)
        },
        handleDragCurrentOpenDeckCard() {
          this.draggindCard = this.currentOpenCardInDec
          this.draggingRow = null
        },
        handleDragStart(card, row) {
          this.draggindCard = card
          this.draggingRow = row
          console.log(card,"this is dragging card")
        },
        handleDropForPile(e, target) {
          console.log(target)
          this.checkCardCanBeDropedOnPile(e.target.id)
          console.log("this is drop", e.target.id)
        },
        checkIfcardCanBeDroppedOnRow(row) {
          const rowArray = this.pileDeck[row]
          console.log(row, rowArray)
          const length = rowArray.length

          const latestCard = rowArray[length-1]
          if(this.draggindCard.color == latestCard.color) {
            console.log("cant move as both the cards are of same color")
          }
          if(this.draggindCard.higher != latestCard.rank) {
            console.log("cant move as number are not maching with the logic")
          }
          this.pileDeck[row].push(this.draggindCard)
          this.updateDraggedRow()
        },
        checkCardCanBeDropedOnPile(id) {
          const droppedCard = this.pileCards.find(card => card.suit == id)
          const index = this.pileCards.findIndex(card => card.suit == id)
          if(!droppedCard) {
            console.warn("no card found to drop")
            return
          }
          const higherOfSelectedPile = droppedCard.higher
          if(higherOfSelectedPile == this.draggindCard.rank) {
            // card can be dropped there
            console.log(this.pileCards[index],"this.pileCards[index]")
            this.pileCards[index].rank =  this.draggindCard.rank
            this.pileCards[index].higher = this.draggindCard.higher
            this.updateDraggedRow()
            this.updateRemainingDeck();
          }
        },
        updateRemainingDeck() {
          if(!this.currentOpenCardInDec) {
            return;
          }
          if(this.draggindCard.rank != this.currentOpenCardInDec.rank && this.draggindCard.suit != this.currentOpenCardInDec.suit) {
            console.warn("draggind card is not from deck so not updating anything")
            return
          }
          const draggedCardIndex = this.remainingDeck.findIndex(e => e.rank == this.draggindCard.rank && e.suit == this.draggindCard.suit)
          console.log(this.remainingDeck[draggedCardIndex])
          this.remainingDeck.splice(draggedCardIndex, 1)

          this.onDeckClick()
        },  
        updateDraggedRow() {
          if(this.draggingRow == null) {
            console.warn("no row index found to update")
            return;
          }
          this.pileDeck[`row-${this.draggingRow}`].pop()
          const index = this.pileDeck[`row-${this.draggingRow}`].length
          if(index == 0) {
            return;
          }
          this.pileDeck[`row-${this.draggingRow}`][index - 1].isOpen = true
        },
        onDeckClick() {
          this.currentOpenCardInDec = this.remainingDeck[this.remaininDecNextIndex]
          this.remaininDecNextIndex++;

          if(this.remaininDecNextIndex == (this.remainingDeck.length) ) {
            this.showDeckResetIcon = true;
            this.remaininDecNextIndex = 0
            return
          }

        },
        getPileDec(index) {
          return this.pileDeck[`row-${index}`] || []
        },
        createCard(rank, suit, higher) {
          
          let suitSymbol, cardColor;
          switch (suit) {
              case 'spade':
                  suitSymbol = '♠';
                  cardColor = 'black';
                  break;
              case 'heart':
                  suitSymbol = '♥';
                  cardColor = 'red';
                  break;
              case 'diamond':
                  suitSymbol = '♦';
                  cardColor = 'red';
                  break;
              case 'club':
                  suitSymbol = '♣';
                  cardColor = 'black';
                  break;
              default:
                  suitSymbol = '🃏';
                  cardColor = 'black';
                  break;
          }

          return {
            suitSymbol: suitSymbol,
              rank,
              higher,
              suit,
              color: cardColor,
              below: null
          }
        },
        createDeck() {
          const deck = [];
          for (let i = 0; i < cardSuits.length; i++) {
            for (let j = 0; j < cardRanking.length; j++) {  
              const higherRank = cardRanking[j + 1] ? cardRanking[j + 1] : null;
              const newCard = this.createCard(cardRanking[j], cardSuits[i], higherRank);
              deck.push(newCard);
            }
          }
          return deck;
        },
        handleDropEvent(element) {
          console.log(element,"this is drop element taget")
        },

        distributeCards() {
          const totalRow = 7; 
          this.remainingDeck = this.globalDeck;
         
          // each row will have index+1 card 
          // eg: 0th index  -  1 card and index(0th) card will be opne 
          // eg: 7th row: index - 6 and 7th (index actually) card will be open 

          for(let i=0; i< totalRow; i++) {
            this.pileDeck[`row-${i}`] = []
              for(let j=0; j<= i; j++) {
                const card = this.remainingDeck[this.remainingDeck.length - 1]
                card.isOpen = false
                
                if(j == i) {  
                  card.isOpen = true
                   
                }
                this.pileDeck[`row-${i}`].push(card)
                // remove card from remaining card deck 
                this.remainingDeck.pop()
              }
          }
          console.log(this.pileDeck,"this is pile deck")
        },
        shuffleCards(deck) {
         return deck.sort(() => Math.random() - 0.5);
       },
      },
      checkCardCanBeDropeed() {

      },
      mounted() {
        this.globalDeck = this.createDeck();
        this.shuffledDeck = this.shuffleCards(this.globalDeck)
        this.distributeCards()
      },
      created() {
      }
  }

</script>

<style scoped>
.root {
  position: relative;
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
}

.top-area {
  justify-content: space-around;
  align-items: center;
  min-height: 30%;
  background: darkgreen;
  display: flex;
}

.right-area {
  width: 45%;
  display: flex;
  justify-content: space-around;
}

.closed-card {
  background: skyblue;
}

.bg-white {
  background: white;
}


.card {
  display: flex;
  border-radius: 8px;
  width: 100px;
  height: 150px;
  border: black solid 1px;
  box-shadow: 0cm;
}
.red {
  color: red
}

.black {
  color: black
}

.card-details {
  margin:auto;
  align-content: center;
  font-size: 60px;
}
.bottom-container {
  padding-top: 60px;
  display: flex;
  height: 100%;
  width: 100%;
  justify-content: space-around;
  background: lightgreen;
}
.pile-row {
  display: flex;
  flex-direction: column;
}
.pile-row .card:not(:first-child) {
  margin-top: -100px;
}
.deck-container {
  display: flex;
}
.pile-column {
  display: flex;
}
</style>
