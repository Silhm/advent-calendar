<template>
  <div class="cookie-consent-banner" :style="visible? 'display:inherit':'display:none'">
    <div class="cookie-consent-banner__inner">
      <div class="cookie-consent-banner__copy">
        <div class="cookie-consent-banner__header">Ce site utilise des cookies</div>
        <div class="cookie-consent-banner__description">
          Chez nous, les cookies, on les mange! Ici, ils nous servent à créer une expérience unique et personnalisée mais aucunement à des fin statistiques.
          <br>En acceptant, vous nous autorisez à utiliser vos cookies. (vous pouvez utiliser ce site sans cookies en cliquant sur Refuser, mais votre expérience ne sera pas aussi amusante!)
        </div>
      </div>

      <div class="cookie-consent-banner__actions">
        <a href="#" class="cookie-consent-banner__cta" @click="clicked(true)">
          OK
        </a>

        <a href="#" class="cookie-consent-banner__cta cookie-consent-banner__cta--secondary" @click="clicked(false)">
          Refuser
        </a>

        <input type="checkbox" id="switch" v-model="remember"/><label for="switch">Se souvenir de mon choix</label><span class="remember"> Se souvenir de mon choix</span>

      </div>
    </div>
  </div>

</template>


<script>

  export default {
    data() {
      return {
        accept: false,
        visible: true,
        remember: false,
      }
    },

    mounted() {
      this.visible = localStorage.getItem('hide-cookie-banner') === null? true:localStorage.getItem('hide-cookie-banner') === "false";
      this.accept = localStorage.getItem('use-cookies');
    },
    methods: {
      clicked(consent){
        this.accept = consent;
        this.visible = false;
        this.$emit('consent', consent);
        localStorage.setItem('use-cookies', consent);
        localStorage.setItem('hide-cookie-banner', this.remember);
      },
    }
  }

</script>

<style lang="scss">
  .cookie-consent-banner {
    position: fixed;
    bottom: 0;
    left: 0;
    z-index: 2147483645;
    box-sizing: border-box;
    width: 100%;

    padding: 1em;
    background-color: #F1F6F4;
  }

  .cookie-consent-banner__inner {
    max-width: 960px;
    margin: 0 auto;
    padding: 32px 0;
  }

  .cookie-consent-banner__copy {
    margin-bottom: 16px;
  }

  .cookie-consent-banner__actions {
  }

  .cookie-consent-banner__header {
    margin-bottom: 8px;

    font-family: "CeraPRO-Bold", sans-serif, arial;
    font-weight: normal;
    font-size: 16px;
    line-height: 24px;
  }

  .cookie-consent-banner__description {
    font-family: "CeraPRO-Regular", sans-serif, arial;
    font-weight: normal;
    color: #838F93;
    font-size: 16px;
    line-height: 24px;
  }

  .cookie-consent-banner__cta {
    box-sizing: border-box;
    display: inline-block;
    min-width: 164px;
    padding: 11px 13px;

    border-radius: 2px;

    background-color: #2CE080;

    color: #FFF;
    text-decoration: none;
    text-align: center;
    font-family: "CeraPRO-Regular", sans-serif, arial;
    font-weight: normal;
    font-size: 16px;
    line-height: 20px;
  }

  .cookie-consent-banner__cta--secondary {
    padding: 9px 13px;

    border: 2px solid #3A4649;

    background-color: transparent;

    color: #2CE080;
  }

  .cookie-consent-banner__cta:hover {
    background-color: #20BA68;
  }

  .cookie-consent-banner__cta--secondary:hover {
    border-color: #838F93;

    background-color: transparent;

    color: #22C870;
  }

  .cookie-consent-banner__cta:last-child {
    margin-left: 16px;
  }


  $checkboxSize: 50;

  input[type=checkbox]{
    height: 0;
    width: 0;
    visibility: hidden;
  }

  label {
    cursor: pointer;
    text-indent: -9999px;
    width: $checkboxSize+px;
    height: $checkboxSize/2+px;
    background: grey;
    display: inline-block;
    border-radius: $checkboxSize+px;
    position: relative;
  }

  label:after {
    content: '';
    position: absolute;
    top: 1px;
    left: 2px;
    width: 22px;
    height: 22px;
    background: #fff;
    border-radius: 22px;
    transition: 0.3s;
  }

  input:checked + label {
    background: #2CE080;
  }

  input:checked + label:after {
    left: calc(100% - 2px);
    transform: translateX(-100%);
  }

  label:active:after {
    width: 33px;
  }

  input:after{
    content: "waaaa"
  }
  .remember {
    font-family: "CeraPRO-Regular", sans-serif, arial;
    font-weight: normal;
    color: #838F93;
    font-size: 16px;
    line-height: 24px;
  }


</style>
