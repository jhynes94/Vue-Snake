<template>
    <canvas id="gameScreen" v-bind:width="xsize" v-bind:height="ysize" ></canvas>
</template>


<script> 
export default {
    name: 'Snake',
    props: {
        msg: String
    },
    data() {
        return {
            snakeDirection: 'right',
            playerLost: false,
            positions: [],
            xsize: 500,
            ysize: 500,
            xgrid: 30,
            ygrid: 30,
            gameTimer: null,
            snakePostiton: [],
            foodPostiton: [],
        };
    },

    mounted() {
        this.prepareForGame();
    },

    methods: {

        prepareForGame(){
            //----------Create starting conditions-----------------

            //Properly Create 2d array for game grid
            for (let i = 0; i < this.xgrid; i++) { 
                this.positions[i] = new Array(this.ygrid);// new array of 3 locations
            } 

            //Fill Grid with 0 to represent empty
            this.clearAllPositions();

            //Place food location
            this.placeFoodPosition();

            //Set Default direction
            this.snakeDirection = 'right'

            //Add Snake
            this.snakePostiton = []
            this.snakePostiton[0] = [5, 5]
            this.snakePostiton[1] = [4, 5]
            this.snakePostiton[2] = [3, 5]
            this.updatePositionsWithSnakePosition(); 


            //Start basic Rendering
            /** @type {CanvasRenderingContext2D} */
            this.canvas = document.getElementById("gameScreen")
            this.ctx = this.canvas.getContext("2d");

            //Add listener for keystrokes
            document.addEventListener('keydown', this.onKeyDown);

            //start game
            this.gameTimer = setInterval(() => {
                this.gameLoop();
            }, 200)
        },


        gameLoop(){

            this.clearAllPositions();

            //Update head position for new move
            let newPos = []
            switch(this.snakeDirection) {
            case 'up':
                newPos = [this.snakePostiton[0][0], this.snakePostiton[0][1] - 1]
                break;
            case 'down':
                newPos = [this.snakePostiton[0][0], this.snakePostiton[0][1] + 1]
                break;
            case 'left':
                newPos = [this.snakePostiton[0][0] - 1, this.snakePostiton[0][1]]
                break;
            case 'right':
                newPos = [this.snakePostiton[0][0] + 1, this.snakePostiton[0][1]]
            }
            
            //Check if snake collidied with itself
            for(var i=0; i< this.snakePostiton.length; i++){
                if(this.snakePostiton[i][0] == newPos[0] && this.snakePostiton[i][1] == newPos[1]){
                    this.endGame();
                    return
                }
            }
            
            //Check if snake collidied with outside wall
            if(newPos[0] < 0 || newPos[1] < 0 || newPos[0] >= this.xgrid || newPos[1] >= this.ygrid){
                this.endGame();
                return
            }

            this.snakePostiton.unshift(newPos); //Adds head to one palce ahead
            

            //See if snake is eating food
            if(this.snakePostiton[0][0] == this.foodPostiton[0] && this.snakePostiton[0][1] == this.foodPostiton[1]){
                console.log("Food eaten")
                this.placeFoodPosition()
                this.$emit('IncrementScore'); //Increment score in parent component
            }
            else {
                this.snakePostiton.pop() //remove last to stop snake from growing
            }

            //Place food
            this.positions[this.foodPostiton[0]][this.foodPostiton[1]] = 'food'
            
            //Update main model with snake array
            this.updatePositionsWithSnakePosition();
            

            //Update Rendering
            this.ctx.clearRect(0, 0, this.xsize, this.ysize) //Clear Canvas
            for (var x=0; x < this.xgrid; x++){
                for (var y=0; y < this.ygrid; y++){

                    let xpos = (this.xsize/this.xgrid)*x
                    let ypos = (this.ysize/this.ygrid)*y

                    let xwidth = (this.xsize/this.xgrid)+1
                    let ywidth = (this.ysize/this.ygrid)+1

                    if(this.positions[x][y] == 'Snake'){
                        this.ctx.fillStyle = '#088523'
                        this.ctx.fillRect(xpos, ypos, xwidth, ywidth);
                    }
                    // else if(this.positions[x][y] == 'empty'){
                    //     this.ctx.fillStyle = '#ffffff'
                    //     this.ctx.fillRect(xpos, ypos, xwidth, ywidth);
                    // }
                    
                    else if(this.positions[x][y] == 'food'){
                        this.ctx.fillStyle = '#d10a32'
                        this.ctx.fillRect(xpos, ypos, xwidth, ywidth);
                    }
                  
                }
            }

            
            
        },


        endGame(){
            this.$emit('ClearScore'); //Clear score in parent component
            //Restart Game
            clearInterval(this.gameTimer);
            this.prepareForGame()
        },

        clearAllPositions(){
            //Fill Grid with 0 to represent empty
            for (var x=0; x < this.xgrid; x++){
                for (var y=0; y < this.ygrid; y++){
                  this.positions[x][y]='empty';
                }
            }
        },

        getRandomInt(max) {
            return Math.floor(Math.random() * Math.floor(max));
        },

        placeFoodPosition(){
            this.foodPostiton = [this.getRandomInt(this.xgrid-1), this.getRandomInt(this.ygrid-1)]
        },

        updatePositionsWithSnakePosition(){
            //console.log([this.snakePostiton[0][0]])
            for(var i=0; i < this.snakePostiton.length; i++){
                this.positions[this.snakePostiton[i][0]][this.snakePostiton[i][1]] = "Snake"
            }
        },

        onKeyDown(event) {
            const codes = ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'Space'];
            if (!codes.includes(event.code)) return;
            
            //Verifies if snake will collide with itself before it changes direction
            switch (event.code) {
            case 'ArrowUp':
                if(this.snakeDirection == 'down') return;
                this.changeSnakeDirection('up');
                break;
            case 'ArrowDown':
                if(this.snakeDirection == 'up') return;
                this.changeSnakeDirection('down');
                break;
            case 'ArrowLeft':
                if(this.snakeDirection == 'right') return;
                this.changeSnakeDirection('left');
                break;
            case 'ArrowRight':
                if(this.snakeDirection == 'left') return;
                this.changeSnakeDirection('right');
                break;
            case 'Space':
                //this.playPauseGame();
                break;
            default:
                break;
            }
        },

        changeSnakeDirection(direction) {
            this.snakeDirection = direction;
        },

    }
};
</script>


<style scoped>
    #gameScreen {
        border: 1px solid black;
    }
</style>
