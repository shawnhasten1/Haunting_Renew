<template>
  <div id="wrapper" @mousedown="setMouseDown($event)" @mouseup="setMouseUp()" @mousemove="dragBackground($event)">
    <div id="tile_holder" v-bind:style="'transform:translate('+holder.pos_x+'px,'+holder.pos_y+'px) scale(1);'">
      <div class="row" v-for="row in tiles" :key="row">
        <div class="tile" v-for="col in row" :key="col" v-bind:id="getID(col.row, col.col)" :row="col.row" :col="col.col" v-bind:style="{'background-image':'url('+getImg(col.background)+')'}">
          {{col.row}} - {{col.col}}
          <div class="token_holder">
            <div v-for="player in col.players" :key="player" style="width:75px; height:75px; background-size: cover;" v-bind:style="{'background-image':'url('+getImg('players/player_1.png')+')'}"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div id="controller_holder">
    <div class="controller_key" style="margin-top:105px; margin-left:160px;" @mouseup="movePlayers('left')">&mapstoleft;</div>
    <div class="controller_key" style="margin-top:105px; margin-left:290px;" @mouseup="movePlayers('right')">&mapsto;</div>
    <div class="controller_key" style="margin-top:105px; margin-left:225px;" @mouseup="movePlayers('down')">&mapstodown;</div>
    <div class="controller_key" style="margin-top:30px; margin-left:225px;" @mouseup="movePlayers('up')">&mapstoup;</div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data: function(){
    return {
      available_tiles:[],
      players:[],
      tiles:[],
      mouse_down:false,
      is_card_blocker:false,
      top_z_index:1,
      mouse_positions:{
        start_x:0,
        start_y:0
      },
      holder:{
        pos_x:-5484/2,
        pos_y:-5484/2
      },
      cards:[
        {
          pos_x:0, 
          pos_y:0,
          z_index:0
        },
        {
          pos_x:400, 
          pos_y:0,
          z_index:0
        }
      ]
    }
  },
  methods:{
    setMouseDown: function(e){
      document.getElementById("wrapper").style.cursor = "grabbing";
      this.mouse_down = true;

      //GET THE START POSITION OF THE MOUSE SO WE CAN OFFSET WHEN MOVE
      this.mouse_positions.start_x = e.clientX;
      this.mouse_positions.start_y = e.clientY;
    },
    setMouseUp: function(){
      document.getElementById("wrapper").style.cursor = "default";
      this.mouse_down = false;
      this.is_card_blocker = false;
    },
    dragBackground: function(e){
      if(this.mouse_down && !this.is_card_blocker){
        //CALCULATE HOW MUCH OUR MOUSE MOVED TO OFFSET THE HOLDER
        this.holder.pos_x += e.clientX - this.mouse_positions.start_x;
        this.holder.pos_y += e.clientY - this.mouse_positions.start_y;

        if(this.holder.pos_x>=0){
          this.holder.pos_x = 0;
        }

        if(this.holder.pos_y>=0){
          this.holder.pos_y = 0;
        }

        if(this.holder.pos_x<=-10369/2){
          this.holder.pos_x = -10369/2;
        }

        if(this.holder.pos_y<=-11210/2){
          this.holder.pos_y = -11210/2;
        }

        //RESET THE START POSITIONS TO THE NEW MOUSE POSITION
        this.mouse_positions.start_x = e.clientX;
        this.mouse_positions.start_y = e.clientY;
      }
    },
    getID(row, col) {
      return 'col-'+String(row)+'-'+String(col)
    },
    getImg(img) {
      return require(`./assets/images/${img}`);
    },
    movePlayers(dir){
      var curr_row = this.players[0]['row'];
      var curr_col = this.players[0]['col'];
      if(dir == 'up'){
        if(this.canMove(curr_row, curr_col, 'up', curr_row-1, curr_col, 'down')){
          this.players[0]['row'] -= 1;
          console.log('UP')
        }
      }
      else if(dir == 'down'){
        if(this.canMove(curr_row, curr_col, 'down', curr_row+1, curr_col, 'up')){
          this.players[0]['row'] += 1;
          console.log('DOWN')
        }
      }
      else if(dir == 'left'){
        if(this.canMove(curr_row, curr_col, 'left', curr_row, curr_col-1, 'right')){
          this.players[0]['col'] -= 1;
          console.log('LEFT')
        }
      }
      else if(dir == 'right'){
        if(this.canMove(curr_row, curr_col, 'right', curr_row, curr_col+1, 'left')){
          this.players[0]['col'] += 1;
          console.log('RIGHT')
        }
      }
      this.calculatePlayers();
    },
    canMove(row, col, dir, new_row, new_col, new_dir){
      if(this.tiles[row][col]['directions'].includes(dir)){
        if(this.tiles[new_row][new_col]['directions'].includes(new_dir)){
          return true;
        }
        else if(this.tiles[new_row][new_col]['available']){
          this.tiles[new_row][new_col]['available'] = false;
          return this.generateTile(new_row, new_col, new_dir);
        }
      }
      return false;
    },
    generateTile(new_row, new_col, new_dir){
      this.available_tiles = this.shuffle(this.available_tiles);
      for(var i = 0; i<this.available_tiles.length; i++){
        if(this.available_tiles[i]['directions'].includes(new_dir) && this.available_tiles[i]['floor'].includes('ground')){
          this.tiles[new_row][new_col]['background'] = 'tiles/'+this.available_tiles[i]['img'];
          this.tiles[new_row][new_col]['directions'] = this.available_tiles[i]['directions'];
          this.available_tiles.splice(i, 1);
          return true;
        }
      }
      return false;
    },
    shuffle(array) {
      let currentIndex = array.length,  randomIndex;

      // While there remain elements to shuffle.
      while (currentIndex != 0) {

        // Pick a remaining element.
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex--;

        // And swap it with the current element.
        [array[currentIndex], array[randomIndex]] = [
          array[randomIndex], array[currentIndex]];
      }

      return array;
    },
    calculatePlayers(){
      for(var x=0; x<24; x++){
        for(var y=0; y<24; y++){
          this.tiles[x][y]['players'] = [];
        }
      }
      
      for(var i = 0; i<this.players.length; i++){
        this.tiles[this.players[i]['row']][this.players[i]['col']]['players'].push(this.players[i]);
      }
    }
  },
  mounted() {
    for(var x=0; x<24; x++){
      var row_tiles = [];
      for(var y=0; y<24; y++){
        var url = 'tiles/blank.jpg';
        var directions = [];
        var available = true;
        if(y == 12){
          if(x==11){
            url = 'tiles/foyer-top.jpg';
            directions = ['down'];
            available = false;
          }
          else if(x==12){
            url = 'tiles/foyer-mid.jpg';
            directions = ['left','right', 'down', 'up'];
            available = false;
          }
          else if(x==13){
            url = 'tiles/foyer-bottom.jpg';
            directions = ['left','right', 'down', 'up'];
            available = false;
          }
        }
        row_tiles.push({
          'background':url,
          'available':available,
          'directions':directions,
          'row':x,
          'col':y,
          'players':[]
        })
      }
      this.tiles.push(row_tiles)
    }
    
    axios.get("http://localhost:5000/v1/game")
      .then(response => {
        console.log(response.data);
        this.available_tiles = response.data['available_tiles'];
        this.players = response.data['players'];
        this.calculatePlayers();
        console.log(this.tiles);
      })
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
