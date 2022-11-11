<template>
  <gameLive :available_tiles="available_tiles" :players="players" :tiles="tiles" :mouse_positions="mouse_positions" :holder="holder"/>
</template>

<script>
import gameLive from './components/GameLive.vue'

export default {
  name: 'App',
  components: {
    gameLive
  },
  data: function(){
    return {
      available_tiles:[],
      players:[],
      tiles:[],
      mouse_positions:{
        start_x:0,
        start_y:0
      },
      holder:{
        pos_x:-5484/2,
        pos_y:-5484/2
      }
    }
  },
  methods:{
  },
  mounted() {
  }
}
</script>

<style>
*{
  -webkit-touch-callout: null; /* iOS Safari */
    -webkit-user-select: null; /* Safari */
     -khtml-user-select: null; /* Konqueror HTML */
       -moz-user-select: null; /* Old versions of Firefox */
        -ms-user-select: null; /* Internet Explorer/Edge */
            user-select: null; /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */
}
#wrapper {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  position: absolute;
  left: 0px;
  top: 0px;
  width: 100vw;
  height:100vh;
  background: black;
  overflow: hidden;
  -webkit-touch-callout: null; /* iOS Safari */
    -webkit-user-select: null; /* Safari */
     -khtml-user-select: null; /* Konqueror HTML */
       -moz-user-select: null; /* Old versions of Firefox */
        -ms-user-select: null; /* Internet Explorer/Edge */
            user-select: null; /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */
}
#tile_holder{
  width: 12288px;
  height: 12288px;
  background-size: cover;
  transform: translate(-6144px,-6144px);
}
.row{
  width: 12288px;
  max-height: 256px;
  display: flex;
  overflow: hidden;
}
.tile{
  width: 256px;
  height: 256px;
  max-height: 256px;
  border: solid 1px white;
  color: white;
  background-size: cover;
}
.token_holder{
  margin-top: 40px;
  margin-left: 40px;
}
#controller_holder{
  background:black; 
  width:500px; 
  height: 250px; 
  position:absolute; 
  bottom:0px; 
  left: 0px;
  border: solid 1px white;
}
.controller_key{
  position:absolute;
  font-size:24px;
  color:white;
  background: black;
  border: solid 1px white;
  border-radius: 3px;
  width: 55px;
  height: 62px;
  line-height: 2.5;
  text-align: center;
  -webkit-touch-callout: null; /* iOS Safari */
    -webkit-user-select: null; /* Safari */
     -khtml-user-select: null; /* Konqueror HTML */
       -moz-user-select: null; /* Old versions of Firefox */
        -ms-user-select: null; /* Internet Explorer/Edge */
            user-select: null; /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */
}
.controller_key:hover{
  opacity: .7;
  cursor: pointer;
}
</style>
