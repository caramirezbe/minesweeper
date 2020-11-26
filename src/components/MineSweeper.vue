<template>
  <v-container class="grey lighten-5">
    
    <v-row>
      <v-col cols="4">
        <v-list-item>
            <v-btn class="primary" @click="startGame">Start game</v-btn>
        </v-list-item>
        <v-list-item>
            <p>Flags: <v-icon color="red">mdi-flag </v-icon>{{flag}}</p>
        </v-list-item>
        <v-list-item v-if="winner">
            <v-alert type="error" v-if="winner === 'lose'">Game Over!!! <h4>You lose</h4></v-alert>
            <v-alert type="success" v-if="winner === 'winner'">Game Over!!! <h4>You win</h4></v-alert>  
        </v-list-item>      
      </v-col>
      <v-col id="grid" cols="8" class="success pa-5" >
            <div v-for="(rows, idx1) in cells" :key="idx1">
                <div v-for="(cols, idx2) in cells" :key="idx2"
                class="cell"
                       
                :class="{active: cells[idx1][idx2].usedCell}"
                @click="openCell(idx1,idx2)"
                @contextmenu.prevent="blockCell(idx1,idx2)"
                ><p v-if="cells[idx1][idx2].usedCell && cells[idx1][idx2].isBomb" ><v-icon>mdi-bomb</v-icon></p>
                <p v-else-if="cells[idx1][idx2].usedCell" @play="chekColor(idx1, idx2)">{{cells[idx1][idx2].bombsAround}}</p>
                <p v-else-if="cells[idx1][idx2].flag"><v-icon color="red">mdi-flag</v-icon></p>
                </div>
            </div>  
      </v-col> 
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'MineSweeper',
    data(){
      return{
        cols: 10,
        rows: 10,
        bombs: 18,
        totalUncoverd: 0,
        flag: 0,
        cells:[],
        winner: '',
        color:['black','blue','green','yellow','grey','pink','purple','orange','red']
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


    methods: {
      startGame(){ 
        this.cells = [];
        this.winner = '';
        this.flag = this.bombs
        this.createGrid();
        this.spreadBomb();
        this.bombsAround();
        this.checkBombs();

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
               col.push({bombsAround: 0, isBomb: false, usedCell: false, flag: false});
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
        for (let x = 0; x < this.rows; x++){
            for(let y= 0; y < this.cols; y++){
              if (this.cells[x][y].isBomb){  
                console.log(x +',' +y);
              }
            }
          }
      },

      bombsAround(){
         for (let r=0; r < this.rows; r++ ){
            for (let c=0; c < this.cols; c++){
             this.cells[r][c].bombsAround = this.countBombs(r,c);
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
      } 
    },//method close

}// export default close
</script>

<style scoped>
	
#grid{
  display:grid;
  grid-template-columns: repeat(10, 40px);
  grid-template-rows: repeat(10, 40px);
  
}
	
.cell {
  text-align: center;
  background-color: #42A5F5;
  height: 40px;
  border: 1px solid black;
}

#grid div div:hover{
  background-color: #90CAF9;
}

.active{
  background-color: white;
}

</style>
