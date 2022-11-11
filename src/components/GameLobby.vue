<template>
    <div id="wrapper" class="p-4">
        <form class="form-inline col-md-6 col-lg-3 col-sm-12 border p-4 rounded">
            <div class="form-group">
                <label for="join_code">JOIN CODE</label>
                <input type="text" id="join_code" class="form-control" placeholder="Enter Join Code">
            </div>
            <div class="btn btn-primary mt-2" @click="getGame()">Join</div>
        </form>
        <form class="form-inline col-md-6 col-lg-3 col-sm-12 border p-4 rounded mt-2">
            <div class="form-group">
                <label for="join_code">Generate Game</label>
            </div>
            <div class="btn btn-primary mt-2" @click="generateGame()">Generate</div>
        </form>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'GameLobby',
    props: {
        available_tiles:Object,
        players:Object,
        tiles:Object,
        mouse_down:Boolean,
        mouse_positions:Object,
        holder:Object,
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
        }
      }
    },
    methods:{
        getGame: function(){
            axios.post("http://localhost:5000/v1/game", {'join_code':document.getElementById("join_code").value})
            .then(response => {
                console.log(response.data);
                this.session.available_tiles = response.data['available_tiles'];
                this.session.players = response.data['players'];
                localStorage.join_code = response.data['join_code'];
                window.location.reload();
            })
        },
        generateGame: function(){
            axios.get("http://localhost:5000/v1/generate_game")
            .then(response => {
                console.log(response.data);
                document.getElementById("join_code").value = response.data['join_code'];
                this.getGame();
            })
        }
    }
}
</script>

<style scoped>

@import 'bootstrap/dist/css/bootstrap.min.css';
label{
    color: white !important;
}
</style>
