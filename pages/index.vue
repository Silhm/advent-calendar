<template>
  <div class="container">

    <!-- CALENDAR DOORS -->
    <section class="calendar-grid">
      <div class="title">
        <div class="title-lines-container">
          <p class="title-line1">Kakemono</p>
          <p class="title-line2">Calendrier de l'avent<br><span class="bigger-text">2020!</span></p>
        </div><!-- closes title-lines-container -->
        <p class="title-intro">
          Découvrez chaque jour les exposants de la Japan Addict et les surprises qu'ils ont à nous faire découvrir!</p>
      </div><!-- closes title -->

      <div v-for="day in daysCount" :class="'day'+day" @mouseover="hover">
        <div class="door" :class="openDoors.indexOf(day)>=0? 'open':''">
          <div class="front" @click="clickFront" :day="day">{{day}}</div>
          <div class="back" @click="clickBack" title="Cliquez pour en savoir plus">{{day}}</div>
        </div>
      </div>

    </section><!-- closes calendar-grid -->

    <!-- MODAL POPUP BOX -->

    <div id="modalPopup" class="modal" :class="modalDisplay? '':'hidden'">
      <!-- MODAL CONTENT -->

      <div class="modalContent">
        <span class="close" @click="modalDisplay=false">&times;</span>
        <div v-if="exposants[currentModal]">
          <h3 class="dailyDate">{{exposants[currentModal].date}}</h3>
          <h2 class="dailyTitle">{{exposants[currentModal].title}}</h2>
          <p class="dailyContent">{{exposants[currentModal].content}}</p>
          <p>
            <span class="highlight">When:</span> <span class="dailyTime">{{exposants[currentModal].time}}</span><br>
            <span class="highlight">Where:</span> <span class="dailyLocation">{{exposants[currentModal].location}}</span>
          </p>
          <p>
            <span class="highlight">Price:</span> <span class="dailyPrice">{{exposants[currentModal].price}}</span>
          </p>
          <a class="dailyLink" target="_blank" :href="exposants[currentModal].linkUrl">{{exposants[currentModal].linkText}}</a>
        </div>
      </div><!-- closes modalContent -->

    </div><!-- closes modalPopup -->

    <div id="modalAbout" class="modal" :class="aboutDisplay? '':'hidden'">
      <!-- MODAL CONTENT -->
      <div class="modalContent">
        <span class="close" @click="aboutDisplay=false">&times;</span>
        <div>
          <h2 class="dailyTitle">Calendrier de l'avent Kakemono</h2>
          <p class="aboutContent">
            Réalisé par Simon FLORENTIN <a href="https://www.simon-florentin.fr">www.simon-florentin.fr</a>
                inspiré par Anwen Williams <a href=" https://anwenwilliams.com/">anwenwilliams.com</a>
            dans le cadre de la Japan Addict Online.
          </p>
          <img src="~static/img/kakemono_logo.jpg">
        </div>
      </div><!-- closes modalContent -->

    </div><!-- closes modalPopup -->

    <cookie-consent></cookie-consent>

    <footer>
      <span @click="aboutDisplay=true">À propos</span> - <span @click="reset">Recommencer</span>
    </footer>

  </div>
</template>

<script>
  import CookieConsent from '../components/CookieConsent.vue'

  export default {
    components: {
      CookieConsent
    },
    data() {
      return {
        daysCount: 30,
        debugMode: false,
        debugDay: 15,

        todaysDate: null,
        modalDisplay: false,
        aboutDisplay: false,
        currentModal: 1,
        exposants: {},
        openDoors: [],

        useCookies: false,
        cookieDoors: null,
      };
    },
    created(){
      let json = require('../assets/exposants.json');
      this.exposants = json.exposants;

      this.useCookies = localStorage.getItem('use-cookies') === "true";
      let cookie = this.$cookie.get('doors');
      if(!cookie && this.useCookies){
        this.$cookie.set('doors','[]', { expires: '2M' });
        cookie = this.$cookie.get('doors');
      }
      this.cookieDoors = cookie;

    },
    mounted() {
      let date = new Date();
      this.todaysDate = this.debugMode? this.debugDay:date.getDate();
      this.openDoors = this.useCookies? JSON.parse(this.cookieDoors):[];

      this.$children[0].$on('consent', (accept) => {
        this.useCookies = accept
        if(!accept) {
          this.$cookie.delete('doors');
        }
      });


      window.addEventListener("keydown",function (e) {
        if (e.keyCode === 114 || (e.ctrlKey && e.keyCode === 70)) {
          e.target.classList.add("vibrate");
          window.setTimeout(function () {
            e.target.classList.remove("vibrate");
            alert("Non non non! il faut chercher tout·e seul·e ;) ");
          },500);

          e.preventDefault();
        }
      })

    },
    methods:{
      clickFront(event){
        let door = event.target;
        var numberClicked = door.getAttribute("day");
        var calendarNum = parseInt(numberClicked, 10);

        if (calendarNum <= this.todaysDate) {
          this.openDoors.push(calendarNum);
          if(this.useCookies) {
            this.$cookie.set('doors', JSON.stringify(this.openDoors), {expires: '1M'});
          }
        }
        else{
          door.classList.add("vibrate");
          let text = door.innerText;
          door.innerHTML = "<small>Pas encore</small>"

          window.setTimeout(function () {
            door.classList.remove("vibrate");
            door.innerText = text;
          },500);

        }
      },

      clickBack(event){
        let door = event.target;
        // find number of corresponding door-front (as back has no innerHTML of its own) //
        var calendarNum = door.previousElementSibling.innerHTML;
        // -1 from door-front to get correct message array index //
        this.currentModal = calendarNum - 1;

        // change modal css to display it //
        this.modalDisplay = true;
      },

      hover(event){
        let targetDay = parseInt(event.target.getAttribute("day"),10);
        if(targetDay > this.todaysDate){
          event.target.parentNode.classList.add("no-hover");
        }
      },

      reset(){
        this.openDoors = [];
      }
    }

  }
</script>

<style lang="scss">
  /* === SCSS COLOUR VARIABLES === */

  $backgroundteal: #266c8e;
  $descriptionBorder: #a83c00; // bordure de la description
  $bordercolor: #ff9752;
  $textcolor: #ff8032;
  $darkslate: #4e1d00; // bordure des chiffres
  $doorback: #f47325;
  $titlebackground: #ffffff;
  $numbers: #ffffff;
  $linkbuttonhover: #2883af;

  /* SCSS RESPONSIVE TITLE TYPE @MIXIN */

  /// Viewport sized typography with minimum and maximum values
  /// @author Eduardo Boucas (@eduardoboucas)
  ///
  /// @param {Number}   $responsive  - Viewport-based size
  /// @param {Number}   $min         - Minimum font size (px)
  /// @param {Number}   $max         - Maximum font size (px)
  ///                                  (optional)
  /// @param {Number}   $fallback    - Fallback for viewport-
  ///                                  based units (optional)
  ///
  /// @example scss - 5vw font size (with 50px fallback),
  ///                 minumum of 35px and maximum of 150px
  /// @include responsive-font(5vw, 35px, 150px, 50px);

  @mixin responsive-font($responsive, $min, $max: false, $fallback: false) {
  $responsive-unitless: $responsive / ($responsive - $responsive + 1);
  $dimension: if(unit($responsive) == 'vh', 'height', 'width');
  $min-breakpoint: $min / $responsive-unitless * 100;

    @media (max-#{$dimension}: #{$min-breakpoint}) {
      font-size: $min;
    }

      @if $max {
      $max-breakpoint: $max / $responsive-unitless * 100;

        @media (min-#{$dimension}: #{$max-breakpoint}) {
          font-size: $max;
        }
        }

        @if $fallback {
          font-size: $fallback;
        }

        font-size: $responsive;
      }

      /* =============================== */

      body {
        background: url(../static/img/bg2.jpg) no-repeat center center fixed;
        background-position-x: 75%;
        -webkit-background-size: cover;
        -moz-background-size: cover;
        -o-background-size: cover;
        background-size: cover;

        font-family: 'Montserrat', Verdana;
      }

      /* === MOBILE-FIRST GRID LAYOUT === */

      .calendar-grid {
        display: grid;
        width: 96%;
        max-width: 900px;
        margin: 1% auto;

        grid-template-columns: repeat(3, 1fr);
        grid-template-rows: auto;
        grid-gap: 15px;

        grid-template-areas:    "title    title     title"
                          "day23    day20     day12"
                          "day2     day14     day4"
                          "day5     day22     day28"
                          "day1     day26     day9"
                          "day10    day11     day29"
                          "day13    day3      day15"
                          "day6     day17     day8"
                          "day25    day7      day27"
                          "day16    day18     day24"
                          "day19    day30     day21";
      }

      /* MEDIA QUERY -- DESKTOP LAYOUT */

      @media only screen and (min-width: 520px) {

        body {
          background: url(../static/img/bg2.jpg) no-repeat center center fixed;
          -webkit-background-size: cover;
          -moz-background-size: cover;
          -o-background-size: cover;
          background-size: cover;
        }

        .calendar-grid {
          grid-template-columns: repeat(6, 1fr);
          grid-template-areas:  "title      title     title     day2      day7      day1"
                                "title      title     title     day26     day11     day12"
                                "title      title     title     day19     day9      day25"
                                "day29      day8      day30     day30     day21     day20"
                                "day17      day22     day30     day30     day5      day18"
                                "day13      day28     day4      day6      day27     day24"
                                "day3       day23     day16     day14     day10     day15";
        }
      }

      /* === INDIVIDUAL GRID ITEMS ===  */

      .title {
        grid-area: title;
        text-align: center;
        color: $descriptionBorder;
      }

      .title-lines-container {
        background: $titlebackground;
        border-radius: 30px 0;
        border: 5px solid $bordercolor;
        transform: rotate(2deg);
        margin: 1.5% 0 0 0;
      }
      .title-line1 {
        font-family: 'Oleo Script Swash Caps', Courier;
      @include responsive-font(4vw, 30px, 60px, 45px);
        margin: 0;
        padding: 10px 15px 0;
      }
      .title-line2 {
        font-family: 'BioRhyme', Courier;
      @include responsive-font(3vw, 21px, 44px, 35px);
        line-height: 1.1em;
        margin: 0;
        padding: 0 15px 15px;
      }
      .bigger-text {
      @include responsive-font(3.5vw, 26px, 50px, 40px);
      }
      .title-intro {
        background: $titlebackground;
        padding: 20px 25px;
        border: 1px solid $descriptionBorder;
        border-radius: 0 30px;
        line-height: 1.5em;
        transform: rotate(-1.5deg);
        margin: 5% 2% 2%;
      }

      .day1 {
        grid-area: day1;
      .back {
        background: red;// url(attr(data-count));
      }
    }

  $days: 30;
  @for $i from 1 through $days {
    .day#{$i} {
      grid-area: day#{$i};
      .back{
        background: url('../assets/doors/#{$i}.png')
      }
    }
  }

  /* ===  DOOR STYLES ===  */
  .calendar-grid input {
    display: none;
  }

  .door {
    background: rgba(173, 90, 0, 0.1);
    text-shadow:  2px 2px 0 $darkslate,
    2px -2px 0 $darkslate,
    -2px 2px 0 $darkslate,
    -2px -2px 0 $darkslate,
    2px 0px 0 $darkslate,
    0px 2px 0 $darkslate,
    -2px 0px 0 $darkslate,
    0px -2px 0 $darkslate;

    width: 100%;
    height: 100%;
    perspective: 1000px;
    transform-style: preserve-3d;
    transition: all 300ms;
    border: 1.5px dashed $darkslate;
    border-radius: 10px;
    cursor: pointer;
    min-height: 110px;
    font-family: 'BioRhyme', serif;
    font-size: 58px;
    color: $numbers;
    &.no-hover {
       border-color: $darkslate;
     }
    &:not(.no-hover):hover {
       border: 1.5px dashed $numbers;
     }
    small{
      font-size: 0.6em;
      text-align: center;
    }
  }

  .door div {
    position: absolute;
    height: 100%;
    width: 100%;
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .door .front {
    background: rgba(173, 90, 0, 0.1);
  }

  .door .back {
    background-size: contain;
    background-position: center center;
    background-repeat: no-repeat;
    background-color: $doorback;
    transform: rotateY(180deg);
  }

  .open {
    transform: rotateY(180deg);
  }

  footer {
    padding: 15px 15px 10px;
    text-align: center;

    color: lightyellow;
    font-size: 1.1em;
    background-color: rgba(0, 0, 0, 0.16);


  p {
    margin-top: 25px;
    color: #222222;
  }
  }

  /* ===  MODAL (BACKGROUND) ===  */

  .modal {
    display: block; /* Hidden by default */
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto; /* Enable scroll if needed */
    background-color: rgb(0,0,0); /* Fallback color */
    background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
    &.hidden{
      display: none;
    }
  }

  /* === MODAL CONTENT + BOX ===  */

  .modalContent {
    background-color: $titlebackground;
    margin: 15% auto;
    padding: 20px 20px 30px;
    border: 4px solid $bordercolor;
    border-radius: 6px;
    width: 76%;
    max-width: 600px;
    min-height: 200px;
    letter-spacing: 0.5px;
    line-height: 1.4em;
  p {
    font-size: 0.9em;
  }
  }

  .dailyTitle {
    font-family: 'BioRhyme', Courier;
    color: $doorback;
  }

  .dailyDate {
    color: $textcolor;
  }

  .highlight {
    font-weight: bold;
    color: $textcolor;
  }

  .dailyLink {
    background-color: $doorback;
    padding: 5px 8px;
    border-radius: 6px;
    text-decoration: none;
    color: $numbers;
    font-weight: bold;
    line-height: 2.3em;
  &:hover {
     background-color: $linkbuttonhover;
   }
  }

  /* ===  MODAL CLOSE BUTTON (X) ===  */

  .close {
    color: #aaa;
    float: right;
    font-size: 48px;
    line-height: 26px;
    font-weight: bold;
  &:hover,
  &:focus {
     color: black;
     text-decoration: none;
     cursor: pointer;
   }
  }

  @media only screen and (max-width: 519px) {
    .title-line1 {
    @include responsive-font(10vw, 34px, 52px, 38px);
    }
    .title-line2 {
    @include responsive-font(8vw, 27px, 48px, 35px);
    }
    .bigger-text {
    @include responsive-font(10vw, 32px, 52px, 40px);
    }
    .title-intro {
      padding: 18px 18px;
    }
  }

  @media only screen and (max-width: 690px) {
    .door {
      font-size: 46px;
    }
  }

  #modalAbout {
    text-align: center;

    .modalContent {
      .aboutContent {
        padding: 1em;
      }

      p {
        font-size: 1.2em;
      }
    }
  }


  .vibrate{
    -webkit-animation:vibrate .2s 2;
  }

  @-webkit-keyframes vibrate{
    30%{
      transform:rotate(-10deg);
    }
    60%{
      transform:rotate(10deg);
    }
    90%{
      transform:rotate(0deg);
    }
  }



</style>
