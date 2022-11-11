<template>
    <div id="wrapper" @mousedown="setMouseDown($event)" @mouseup="setMouseUp()" @mousemove="dragBackground($event)">
    <div id="tile_holder" v-bind:style="'transform:translate('+holder.pos_x+'px,'+holder.pos_y+'px) scale(1);'">
        <div class="row" v-for="row in session.tiles" :key="row">
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
        <button @click="restartGame()">Leave Game</button>
    </div>
</template>
<script>
    import axios from 'axios'
    
    export default {
      name: 'GameLive',
      props: {
          available_tiles:Object,
          players:Object,
          tiles:Object,
          mouse_down:Boolean,
          mouse_positions:Object,
          holder:Object,
          join_code:String
      },
      data: function(){
        return {
          session:{
              available_tiles:this.available_tiles,
              players:this.players,
              tiles:this.tiles,
              mouse_down:this.mouse_down,
              mouse_positions:this.mouse_positions,
              holder:this.holder,
              join_code:this.join_code,
              canMove:true
          }
        }
      },
      methods:{
      setMouseDown: function(e){
        document.getElementById("wrapper").style.cursor = "grabbing";
        this.session.mouse_down = true;
    
        //GET THE START POSITION OF THE MOUSE SO WE CAN OFFSET WHEN MOVE
        this.session.mouse_positions.start_x = e.clientX;
        this.session.mouse_positions.start_y = e.clientY;
      },
      setMouseUp: function(){
        document.getElementById("wrapper").style.cursor = "default";
        this.session.mouse_down = false;
      },
      dragBackground: function(e){
        if(this.session.mouse_down){
          //CALCULATE HOW MUCH OUR MOUSE MOVED TO OFFSET THE HOLDER
          this.session.holder.pos_x += e.clientX - this.session.mouse_positions.start_x;
          this.session.holder.pos_y += e.clientY - this.session.mouse_positions.start_y;
    
          if(this.session.holder.pos_x>=0){
            this.session.holder.pos_x = 0;
          }
    
          if(this.session.holder.pos_y>=0){
            this.session.holder.pos_y = 0;
          }
    
          if(this.session.holder.pos_x<=-10369/2){
            this.session.holder.pos_x = -10369/2;
          }
    
          if(this.session.holder.pos_y<=-11210/2){
            this.session.holder.pos_y = -11210/2;
          }
    
          //RESET THE START POSITIONS TO THE NEW MOUSE POSITION
          this.session.mouse_positions.start_x = e.clientX;
          this.session.mouse_positions.start_y = e.clientY;
        }
      },
      getID(row, col) {
        return 'col-'+String(row)+'-'+String(col)
      },
      getImg(img) {
        return require(`../assets/images/${img}`);
      },
      movePlayers(dir){
          if(this.session.canMove){
              var curr_row = this.session.players[0]['row'];
              var curr_col = this.session.players[0]['col'];
              if(dir == 'up'){
                  if(this.canMove(curr_row, curr_col, 'up', curr_row-1, curr_col, 'down')){
                  this.session.players[0]['row'] -= 1;
                  console.log('UP')
                  }
              }
              else if(dir == 'down'){
                  if(this.canMove(curr_row, curr_col, 'down', curr_row+1, curr_col, 'up')){
                  this.session.players[0]['row'] += 1;
                  console.log('DOWN')
                  }
              }
              else if(dir == 'left'){
                  if(this.canMove(curr_row, curr_col, 'left', curr_row, curr_col-1, 'right')){
                  this.session.players[0]['col'] -= 1;
                  console.log('LEFT')
                  }
              }
              else if(dir == 'right'){
                  if(this.canMove(curr_row, curr_col, 'right', curr_row, curr_col+1, 'left')){
                  this.session.players[0]['col'] += 1;
                  console.log('RIGHT')
                  }
              }
              this.calculatePlayers();
          }
      },
      canMove(row, col, dir, new_row, new_col, new_dir){
        if(this.session.tiles[row][col]['directions'].includes(dir)){
          if(this.session.tiles[new_row][new_col]['directions'].includes(new_dir)){
            return true;
          }
          else if(this.session.tiles[new_row][new_col]['available']){
            this.session.tiles[new_row][new_col]['available'] = false;
            return this.generateTile(new_row, new_col, new_dir);
          }
        }
        return false;
      },
      generateTile(new_row, new_col, new_dir){
        this.session.available_tiles = this.shuffle(this.session.available_tiles);
        for(var i = 0; i<this.session.available_tiles.length; i++){
          if(this.session.available_tiles[i]['directions'].includes(new_dir) && this.session.available_tiles[i]['floor'].includes('ground')){
            this.session.tiles[new_row][new_col]['background'] = 'tiles/'+this.session.available_tiles[i]['img'];
            this.session.tiles[new_row][new_col]['directions'] = this.session.available_tiles[i]['directions'];
            this.session.available_tiles.splice(i, 1);
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
          for(var x=0; x<this.session.tiles.length; x++){
              for(var y=0; y<this.session.tiles[x].length; y++){
                  this.session.tiles[x][y]['players'] = [];
              }
          }
          console.log(this.session.tiles);
          for(var i = 0; i<this.session.players.length; i++){
              console.log(this.session.players[i]['row'])
              console.log(this.session.players[i]['col'])
              this.session.tiles[this.session.players[i]['row']][this.session.players[i]['col']]['players'].push(this.session.players[i]);
          }
          this.session.canMove = false;
          axios.put("http://localhost:5000/v1/game", {'join_code':localStorage.join_code, 'available_tiles':this.session.available_tiles, 'players':this.session.players, 'tiles':this.session.tiles})
          .then(response => {
              console.log(response.data);
              this.session.canMove = true;
          })
      },
      restartGame(){
          localStorage.join_code = null;
          window.location.reload();
      }
    },
    mounted() {
      axios.get("http://localhost:5000/v1/game?join_code="+localStorage.join_code)
        .then(response => {
          console.log(response.data);
          this.session.available_tiles = response.data['available_tiles'];
          this.session.players = response.data['players'];
          this.session.tiles = response.data['tiles'];
          console.log(this.session.tiles);
          if(this.session.tiles.length>0){
              this.calculatePlayers();
          }
          if(!this.session.tiles.length>0){
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
                  this.session.tiles.push(row_tiles);
              }
              this.session.players = [{
                  "col": 12,
                  "floor": "ground",
                  "row": 13
              }]
              this.session.canMove = false;
              console.log(this.session.tiles)
              axios.put("http://localhost:5000/v1/game", {'join_code':localStorage.join_code, 'players':this.session.players, 'tiles':this.session.tiles})
              .then(response => {
                  console.log(response.data);
                  this.session.canMove = true;
                  this.calculatePlayers();
              })
          }
        })
    }
    }
</script>