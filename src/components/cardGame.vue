<template>
    <div>
     <div class="top-area">
     <div class="container">
       <div class="left-area">
         <div id="deck" class="deck"></div>
         <div id="top-cards" class="top-cards"></div>
       </div>
       <div class="right-area">
         <div class="pile"><span id="spade-cards">♠</span></div>
         <div class="pile"><span id="heart-cards">♥</span></div>
         <div class="pile"><span id="diamond-cards">♦</span></div>
         <div class="pile"><span id="club-cards">♣</span></div>
       </div>
     </div>
   </div>
   
   <div class="container">
     <div class="bottom-area">
       <div class="row"></div>
       <div class="row"></div>
       <div class="row"></div>
       <div class="row"></div>
       <div class="row"></div>
       <div class="row"></div>
       <div class="row"></div>
     </div>
   </div>
   
   
    </div>
   </template>
   
   <script>
   export default {
     name: 'HelloWorld',
     methods:{
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
   
         let frontInnerHTML = `<div class="inner-info card-rank">${rank}</div>`;
         frontInnerHTML += `<div class="inner-info card-suit-small">${suitSymbol}</div>`;
         frontInnerHTML += `<div class="inner-info card-suit-big">${suitSymbol}</div>`;
   
         return {
             rank,
             higher,
             inner: frontInnerHTML,
             suit,
             color: cardColor,
             below: null
         }
       },
   
       createDeck() {
   
         const cardRanking = ['A', '2', '3', '4', '5',
             '6', '7', '8', '9', '10', 'J', 'Q', 'K'];
   
         const cardSuits = ['spade', 'heart', 'diamond', 'club'];
   
         const deck = [];
   
         for (let i = 0; i < cardSuits.length; i++) {
             for (let j = 0; j < cardRanking.length; j++) {
   
                 const higherRank = cardRanking[j + 1] ? cardRanking[j + 1] : null;
                 const newCard = this.createCard(cardRanking[j], cardSuits[i], higherRank);
                 deck.push(newCard);
   
             }
   
         }
         console.log(deck,"this is complete deck")
         return deck;
   
       },
       shuffleCards(deck) {
         return deck.sort(() => Math.random() - 0.5);
       },
   
       putCardsInDeck(cards) {
   
         const deck = document.getElementById('deck');
   
         cards.forEach(() => {
             deck.innerHTML += `<div class="card back"></div>`;
         });
   
         const cardsInDeck = deck.querySelectorAll('.card');
   
         cards.forEach((card, index) => {
             cardsInDeck[index].cardInfo = card;
         });
   
       },
   
       turnCard() {
   
         const isLastCard = this === this.parentNode.lastElementChild;
         const isCardVisible = this.style.visibility !== 'hidden';
         const isCardFacingDown = this.classList.contains('back');
   
         if (isLastCard && isCardVisible) {
   
             if (isCardFacingDown) {
             
                 this.classList.remove('back');
                 this.classList.add(`${this.cardInfo.suit}`);
                 this.innerHTML = this.cardInfo.inner;
             }
   
             this.setAttribute('draggable', true);
         }
   
       },
   
   turnCardBack(card) {
   
   card.classList.remove(`${card.cardInfo.suit}`);
   card.classList.add('back');
   card.innerHTML = '';
   card.setAttribute('draggable', false);
   
   },
   
   distributeCards() {
   
   const piles = document.querySelectorAll('.row');
   
   piles.forEach((pile, index) => {
       
       for (let i = 0; i < index + 1; i++) {
   
           const deck = document.getElementById('deck');
           const topCard = deck.lastElementChild;
   
           if (i === index) {
               this.turnCard.apply(topCard);
           }
   
           deck.removeChild(topCard);
           pile.append(topCard);
       }
   
   });
   
   },
   
   
   showTopCard(topCards, deck, deckTopCard) {
   
   if (deckTopCard.cardInfo) {
   
       deck.removeChild(deckTopCard);
       topCards.append(deckTopCard);
       
       if (topCards.childElementCount > 1) {
           topCards.childNodes[0].setAttribute('draggable', false);
       }
   
       if (topCards.childElementCount > 2) {
           topCards.childNodes[1].setAttribute('draggable', false);
       }
   
       if (topCards.childElementCount > 3) {
   
           topCards.childNodes[2].setAttribute('draggable', false);
   
           const firstCard = topCards.firstElementChild;
   
           this.turnCardBack(firstCard);
           firstCard.style.visibility = 'hidden';
   
           topCards.removeChild(firstCard);
           deck.prepend(firstCard);
       }
   
   }
   
   },
   
   returnCardsToDeck(topCards, deck) {
   
   const cardsToDeck = [];
   
   topCards.childNodes.forEach(card => {
       this.turnCardBack(card);
       card.style.visibility = 'hidden';
       cardsToDeck.push(card);
   });
   
   cardsToDeck.forEach(card => {
       deck.prepend(card);
   });
   
   topCards.innerHTML = '';
   
   const cardsInDeck = deck.querySelectorAll('.card');
   
   cardsInDeck.forEach(card => {
       card.style.visibility = 'visible';
   });
   
   
   },
   
   checkDeckCards() {
   
   const topCards = document.getElementById('top-cards');
   const deck = document.getElementById('deck');
   const deckTopCard = deck.lastElementChild;
   
   if (!deckTopCard) {
       this.returnCardsToDeck(topCards, deck);
   } else if (deckTopCard.style.visibility === 'hidden') {
       this.returnCardsToDeck(topCards, deck);
   } else {
       this.showTopCard(topCards, deck, deckTopCard);
   }
   
   },
   
   
   changeCardPosition(cardToDrop, dropSpot, checkCondition) {
   
   if (checkCondition) {
   
       const cardAbove = cardToDrop.previousSibling;
   
       cardToDrop.parentNode.removeChild(cardToDrop);
       dropSpot.append(cardToDrop);
   
       if (cardToDrop.cardInfo.below) {
           cardToDrop.cardInfo.below.forEach(card => dropSpot.append(card));
       }
   
       return cardAbove;
   }
   
   },
   
   dropInRow(cardToDrop, dropSpot) {
   
   const cardToDropInfo = cardToDrop.cardInfo;
   const lastCardInRow = dropSpot.lastElementChild;
   
   if (lastCardInRow) {
   
       const lastCardInfo = lastCardInRow.cardInfo;
   
       if (lastCardInfo && lastCardInRow.draggable) {
   
           const checkRank = lastCardInfo.rank === cardToDropInfo.higher;
           const checkColor = lastCardInfo.color !== cardToDropInfo.color;
   
           return this.changeCardPosition(cardToDrop, dropSpot, (checkRank && checkColor));
       }
   
   } else {
   
       const checkRank = cardToDrop.cardInfo.rank === 'K';
   
       return this.changeCardPosition(cardToDrop, dropSpot, checkRank);
   
   }
   },
   
   dropInPile(cardToDrop, dropSpot) {
   
   const cardToDropInfo = cardToDrop.cardInfo;
   const lastCardInPile = dropSpot.lastElementChild;
   
   if (!cardToDropInfo.below) {
   
       if (lastCardInPile) {
   
           const lastCardInfo = lastCardInPile.cardInfo;
   
           if (lastCardInfo) {
   
               const checkRank = lastCardInfo.higher === cardToDropInfo.rank;
               const checkSuit = lastCardInfo.suit === cardToDropInfo.suit;
   
               if (checkRank && checkSuit) {
                   lastCardInPile.setAttribute('draggable', false);
               }
   
               return this.changeCardPosition(cardToDrop, dropSpot, (checkRank && checkSuit));
   
           } else {
   
               const checkRank = cardToDropInfo.rank === 'A';
               const checkSuit = lastCardInPile.id.split('-')[0] === cardToDropInfo.suit;
   
               return this.changeCardPosition(cardToDrop, dropSpot, (checkRank && checkSuit));
           }
   
       }
   }
   },
   
   dropCard(event, cardToDrop) {
   
   const tgt = event.target;
   
   const findSpot = function (element, htmlClass) {
   
       const classList = element.classList;
   
       if (classList) {
   
           if (classList.contains(htmlClass)) {
               return element;
   
           } else {
               return findSpot(element.parentNode, htmlClass);
   
           } 
   
       }
   
       return;
   
   }
   
   const dropSpot = findSpot(tgt, 'row') || findSpot(tgt, 'pile');
   
   let cardAbove;
   
   if (dropSpot) {
   
       if (dropSpot.classList.contains('row')) {
   
           cardAbove = this.dropInRow(cardToDrop, dropSpot);
   
       } else if (dropSpot.classList.contains('pile')) {
   
           cardAbove = this.dropInPile(cardToDrop, dropSpot);
   
       }
   
   }
   
   if (cardAbove) {
       this.turnCard.apply(cardAbove);
   }
   
   },
   
   checkGameOver() {
   
   const piles = document.querySelectorAll('.pile');
   
   let allPilesComplete =  true;
   
   piles.forEach(pile => {
       allPilesComplete = allPilesComplete && (pile.childElementCount - 1) === 13;
   });
   
   if (allPilesComplete) {
       alert('YOU WIN!');
   }
   },
   
   resetSelections(card) {
   
   card.cardInfo.below = null;
   card = null;
   
   return card;
   },
   
   dragRowCards(card) {
   
   const cardParent = card.parentNode;
   const isLastCard = card === cardParent.lastElementChild;
   
   if (!isLastCard) {
   
       card.cardInfo.below = [];
   
       for (let i = cardParent.childElementCount - 1; i >= 0; i--) {
   
           if (cardParent.childNodes[i] === card) {
               return;
           }
   
           card.cardInfo.below.unshift(cardParent.childNodes[i]);
       }
   }
   }
   
     },
     data() {
       return {
           globalDeck: [],
           shuffledDeck: []
       }
     },
     mounted() {
       this.globalDeck = this.createDeck();
       this.shuffledDeck = this.shuffleCards(this.globalDeck);
       this.putCardsInDeck(this.shuffledDeck);
       this.distributeCards();
     }
   
   
   
   
   // const deck = document.getElementById('deck');
   // deck.addEventListener('click', checkDeckCards);
   
   // const cards = document.getElementsByClassName('card');
   // for (let i = 0; i < cards.length; i++) {
   // cards[i].addEventListener('click', turnCard);
   // }
   
   // let selectedCard = null;
   
   // document.addEventListener("dragstart", event => {
   // if (event.target.draggable) {
   //     selectedCard = event.target;
   //     dragRowCards(selectedCard);
   // }
   // });
   
   // document.addEventListener("dragover", event => {
   // event.preventDefault();
   // }, false);
   
   // document.addEventListener("drop", event => {
   // event.preventDefault();
   // dropCard(event, selectedCard);
   // selectedCard = resetSelections(selectedCard);
   // checkGameOver();
   // });
     }
   
   </script>
   
   <style scoped>
   @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500&family=Poppins:wght@300;400;500;600&display=swap');
   
   html,
   body {
     width: 100%;
     margin: 0;
     padding: 0;
     background-color: #f7f7f7;
     background-color: #147246;
     font: 400 1.2em sans-serif;
     line-height: 1.2;
     user-select: none;
   }
   
   .container {
     width: fit-content;
     height: 100%;
     margin: 0 auto;
   }
   
   
   .card {
     padding: 0;
     margin: 2vmin;
     width: 7.6vmin;
     height: 11vmin;
     box-shadow: inset 0px 0px 1vmin 0.2vmin #c3c6c6;
     border-radius: 0.8vmin; 
     background-color: #f5f9fa;
     font-family: Roboto;
   }
   
   .card-rank {
     width: 48%;
     float: left;
     margin-left: 0.4vmin;
     margin-right: 0.3vmin;
     letter-spacing: -0.8vmin;
     font-size: 4vmin;
     font-weight: 500;
   }
   
   .card-suit-small {
     width: 40%;
     float: left;
     font-size: 4vmin;
   }
   
   .card-suit-big {
     width: 100%;
     height: inherit;
     line-height: 0.6;
     font-size: 8vmin;
     text-align: center;
   }
   
   @-moz-document url-prefix() {
     .card-rank {
       margin-right: 0.3vmin;
       font-size: 4vmin;
     }
     
     .card-suit-small {
       line-height: 1.6;
       font-size: 3vmin;
     }
     
     .card-suit-big {
       line-height: 0.8;
       font-size: 7vmin;
     }
   }
   
   
   .card.diamond .inner-info,
   .card.heart .inner-info {
     background: linear-gradient(to right, #c21f02, #ff2802, #c21f02, #ff2802, #c21f02);
     -webkit-background-clip: text;
     color: transparent;
   }
   
   .card.club .inner-info,
   .card.spade .inner-info {
     background: linear-gradient(to right, #242222, #534f51, #242222, #534f51, #242222);
     -webkit-background-clip: text;
     color: transparent;
   }
   
   .back {
     box-shadow: inset 0px 0px 0.2vmin 0.2vmin #0a2f41;
     background: linear-gradient(315deg, #242222, #534f51, #242222, #534f51, #242222);
     background: linear-gradient(315deg, #0a3142, #104c66, #12506b, #1b7ca5, #2195ca);
   }
   
   .top-area {
     padding-top: 2vmin;
     width: 100vw;
     height: 16vmin;
     background-color: #1b4934;
   }
   
   .left-area,
   .right-area {
     display: flex;
     place-content: center;
     float: left;
   }
   
   .left-area {
     width: 41vmin;
   }
   
   .right-area {
     width: 57vmin;
   }
   
   .deck {
     margin: 2vmin;
     width: 7.6vmin;
     height: 11vmin;
     box-shadow: inset 0px 0px 0vmin 0.5vmin #c3c6c644;
     border-radius: 0.8vmin;
     background-color: #c3c6c60c;
     position: relative;
   }
   
   .deck::before {
     content: "⟲";
     width: 100%;
     height: inherit;
     line-height: 1.6;
     font-size: 7vmin;
     text-align: center;
     color: #c3c6c644;
     position: absolute;
     cursor: pointer;
   }
   
   .deck .card {
     position: absolute;
     transform: translateX(-2vmin) translateY(-2vmin);
     cursor: pointer;
   }
   
   .top-cards {
     width: 55%;
     display: flex;
   }
   
   .top-cards .card {
     margin-right: -5vmin;
   }
   
   
   .pile {
     padding: 0;
     margin: 2vmin;
     width: 7.6vmin;
     height: 11vmin;
     box-shadow: inset 0px 0px 0vmin 0.5vmin #c3c6c644;
     border-radius: 0.8vmin;
     background-color: #c3c6c60c;
     position: relative;
   }
   
   .pile span {
     width: 100%;
     height: inherit;
     line-height: 1.4;
     font-size: 8vmin;
     text-align: center;
     color: #c3c6c644;
     position: absolute;
   }
   
   
   .pile .card {
     position: absolute;
     transform: translateX(-2vmin) translateY(-2vmin);
   }
   
   .bottom-area {
     display: flex;
     place-content: center;
     width: 100vmin;
     min-height: 100%;
     height: fit-content;
     padding-top: 4vmin;
     padding-bottom: 10vmin;
   }
   
   .row {
     display: grid;
     margin-right: 1%;
     width: 12%;
     height: 100%;
     position: relative;
   }
   
   .row::before {
     content: "";
     width: 7.6vmin;
     height: 11vmin;
     box-shadow: inset 0px 0px 0vmin 0.5vmin #c3c6c644;
     border-radius: 0.8vmin;
     background-color: #c3c6c60c;
     transform: translateX(2vmin) translateY(2vmin);
     position: absolute;
   }
   
   .row .card {
     position: relative;
   }
   
   .row .card:not(:first-child) {
     margin-top: -8vmin;
   }
   
   footer {
     position: fixed;
     bottom: 0;
     width: 100%;
     height: fit-content;
     margin-top: 25vmin;
     padding: 1vmin 0;
     background-color: #1e1f26;
     font-size: 1.6vmin;
   }
   
   footer p {
     margin: 1vmin;
     text-align: center;
     color: #F5F5F5;
     font-family: Poppins;
   }
   
   footer a:link {
     color: #b3a290;
   }
   footer a:visited {
     color: #91d4d0;
   }
   footer a:hover {
     color: #ff3300;
   }
   footer a:active {
     color: #ff3300;
   }
   
   .name-dark {
     color: #91d4d0;
     text-decoration: none;
     background: linear-gradient(to right, #91d4d0 50%, #ff3300 100%);
     -webkit-background-clip: text;
     -webkit-text-fill-color: transparent;
   }
   </style>
   