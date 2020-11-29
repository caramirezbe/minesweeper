<template>
  <v-container class="grey lighten-5">
      <v-banner>
        <v-col>
          <v-btn class="primary ma-5" @click="startGame">Start game</v-btn>
          <v-btn text>Flags: <v-icon color="red">mdi-flag </v-icon>{{flag}}</v-btn>
          <v-btn text>Timer: {{outputTimer}}</v-btn>
        </v-col>
      </v-banner>    
      <div v-if="winner">
        <v-alert type="error" v-if="winner === 'lose'">Game Over!!! <h4>You lose</h4></v-alert>
        <v-alert type="success" v-if="winner === 'winner'">Game Over!!! <h4>You win</h4></v-alert>
      </div>
      <div class="grid-container">    
          <div v-for="(rows, idx1) in cells" :key="idx1">
                <div v-for="(cols, idx2) in cells" :key="idx2"
                class="cell pt-2"
                :class="{active: cells[idx1][idx2].usedCell}"
                @click="openCell(idx1,idx2)"
                @contextmenu.prevent="blockCell(idx1,idx2)"
                ><p v-if="cells[idx1][idx2].usedCell && cells[idx1][idx2].isBomb" ><v-icon color="red">mdi-bomb</v-icon></p>
                <p v-else-if="cells[idx1][idx2].usedCell && cells[idx1][idx2].bombsAround != 0" :style='{color:cells[idx1][idx2].color}'>{{cells[idx1][idx2].bombsAround}}</p>
                <p v-else-if="cells[idx1][idx2].flag"><v-icon color="red">mdi-flag</v-icon></p>
                </div>
            </div>  
      </div>
  </v-container>
</template>

<script>
  export default {
    name: 'MineSweeper',
    data(){
      return{
        cols: 10,
        rows: 10,
        bombs: 15,
        totalUncoverd: 0,
        flag: '',
        cells:[],
        winner: '',
        color:['black','blue','green','orange','grey','pink','purple','brown','red'],
        outputTimer: '00:00',
        time: 0,
        running: true,
      }
    },

    computed:{
      checkColor(idx1,idx2){
        const value = this.cells[idx1][idx2].bombsAround
        console.log('entra');
        console.log(this.color[value]);
        return ':style="color=this.color[value]"';
      }

    },
    
    watch:{
      flag(value){
        if (value === 0 && this.checkBombs() === this.bombs)
        {
          this.winner = 'winner';
          this.running = true;
        }
      },
       winner(value){
        if(value === 'lose'){
          this.openBombs();
          this.running = true;
        } 
      },   
    },


    methods: {
      startGame(){ 
        this.restartGame();
        this.resetTimer();
        this.createGrid();
        this.spreadBomb();
        this.bombsAround();
        this.checkBombs();
        this.startTimer();
      },

      restartGame(){
        this.cells = [];
        this.winner = '';
        this.flag = this.bombs;
        this.totalUncoverd = 0;
      },
      
      blockCell(idx1,idx2){
        this.cells[idx1][idx2].flag = !this.cells[idx1][idx2].flag;
        if(this.cells[idx1][idx2].flag){
          this.flag--;
          console.log('flag--');
        } else {this.flag++;
                console.log('Flag ++');        
        }
        console.log(this.cells[idx1][idx2].flag);
      },

      createGrid(){
        for (let r=0; r < this.rows; r++ ){
            let col= [];
            for (let c=0; c < this.cols; c++){
               col.push({bombsAround: 0, isBomb: false, usedCell: false, flag: false, color: 'black'});
               console.log(col[c].isBomb)
            }
            this.cells.push(col);
          }
      },

      spreadBomb(){
        let b = 1;
        // Bomb Spread
         while(b <= this.bombs){
          let c = Math.floor((Math.random() * this.cols-1) + 1);
          let r = Math.floor((Math.random() * this.rows-1) + 1);
          if (!this.cells[r][c].isBomb){
            console.log(b);
            this.cells[r][c].isBomb = true;
            b++;
          }
        }
      },

      checkBombs(){
        let match = 0;
        for (let x = 0; x < this.rows; x++){
            for(let y= 0; y < this.cols; y++){
              if (this.cells[x][y].isBomb){  
                console.log(x +',' +y);  
                if(this.cells[x][y].flag){
                  match++;
                }
              }
            }
          }
          console.log(match);
        return match;
      },

      bombsAround(){
         for (let r=0; r < this.rows; r++ ){
            for (let c=0; c < this.cols; c++){
             const value = this.countBombs(r,c);
             this.cells[r][c].bombsAround = value;
             this.cells[r][c].color = this.color[value];
            }
          } 
      },

      countBombs(row,col){
        let bombsNumber= 0;
        for(let r = row-1; r <= row+1; r++){
          if (r < 0 || r > this.rows -1) {
            continue;
          }
          for(let c = col-1; c <= col+1; c++){
            if (c < 0 || c > this.cols -1){
              continue;
            }
            if (this.cells[r][c].isBomb){
              bombsNumber++;
            }
          }
        }
        return bombsNumber;
      },

      inBoard(idx1,idx2){
          if(idx1 >= 0 && idx1 < this.rows){
            if(idx2 >= 0 && idx2 < this.cols){
              return true;
            }
          }
          return false;  
          //return idx1 >= 0 && idx1 < this.rows && idx2 >= 0 && idx2 < this.cols;
      },
      
      openCell(idx1,idx2){
        const dx = [-1, -1, -1,  0, 0,  1, 1, 1];
        const dy = [-1,  0, +1, -1, 1, -1, 0, 1];
        if (this.cells[idx1][idx2].usedCell || this.cells[idx1][idx2].flag){
          console.log('Used Cell');
          return true; // alert('This cell has been played');
        }
        else if (this.cells[idx1][idx2].isBomb){
            this.cells[idx1][idx2].usedCell = true;
            this.winner='lose';
            return true; 
          }
          else { 
            this.totalUncoverd++;
            this.cells[idx1][idx2].usedCell = true;  
            console.log('totalUncovered' + ' ' +this.totalUncoverd + ' Change to' + this.cells[idx1][idx2].usedCell);    
            if (this.totalUncoverd === (this.rows*this.cols)-this.bombs){
              this.winner = 'winner';
              return true;  
            }
          }

        if (this.cells[idx1][idx2].bombsAround != 0){
          console.log('diffrent from 0');
          return true;
        }
        for (let d = 0; d < dx.length; d++){
          const newIdx1 = idx1 + dx[d];
          const newIdx2 = idx2 + dy[d];

          if (this.inBoard(newIdx1,newIdx2)){
            this.openCell(newIdx1,newIdx2);  
          }
          
        }
      },

      openBombs(){
        for(let x = 0; x < this.rows; x++){
          for(let y  = 0; y < this.cols; y++){
            if (this.cells[x][y].isBomb){
              this.cells[x][y].usedCell = true;
            }
          }
        }
      },

      startTimer(){
          if(this.running){
            this.running = false;
            this.increment();
          }else{
            this.running = true;
          }
      },

      resetTimer(){
        this.running = true;
        this.time= 0;
        this.output= '00:00';
      },

      increment(){
        if(!this.running){
          setTimeout(() => {
            this.time++;
            let remain = this.time;
            console.log('before min time: ' + remain);
            let min = Math.floor(remain/60);
            console.log('remain: ' + remain)
            remain -= min *60;
            console.log('afert min: ' + remain);
            let sec = remain;
            if (min <= 9){
              min= "0" + min;
            }
            if (sec <= 9){
              sec = "0" + sec;
            }
            this.outputTimer = min + ':' + sec;
            this.increment();
          }, 1000);
        }
      },







    },//method close

}// export default close
</script>



<style scoped>

.container {
  width: 90%;
  max-width: 40rem;
  text-align: center;
  margin: 1rem auto;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  border-radius: 12px;
}

.grid-container{
  display: grid;
  justify-content: center;
  grid-template-columns: repeat(10, 50px);
  grid-template-rows: repeat(10, 50px); 
}

.cell{
  border-radius: 3px;
  height: 50px;
  border: 1px solid black;
  background-color: #81C784;
}

.cell:hover{
  background-color: #1B5E20;
}

.active{
  background-color: white;
} 

</style>
