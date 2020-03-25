# project-spacegame
//This side of the editor is the code that controls your game
//Anything after two slashes (like this line) is ignored by the game

var canvasWidth = 270; //You may need to tweak these slightly to fit your mobile
var canvasHeight = 420; //You may need to tweak these slightly to fit your mobile
var gameStartSpeed = 2;
var playerFallSpeed = 2;
var playerRiseSpeed = 2;
var wallImageWidth = 32; // Make sure you change if you change the width of walls
var threshold = 10; // The minimum vertical gap between walls 
var playerStartHeight = 100;
var playerForwardPosition = 100;
var speedIncreasePoints = 5; //increase game speed for every 5 points
var speedIncreaseRate = 1; //How much the game speed increases for every 5 points

var frameRate = 30; //Frames per second
var canvas;
var player;
var numberOfWalls; //Calculated later
var topWalls;
var score;
var scoreContainer;
var bottomWalls;
var gameSpeed; //Current speed
var gameState = "menu"; //Where the player is in the game



//Initialise game when window is fully loaded
window.onload = function(){
	document.title = "My Chopper Game"; //The title of your game at the top of the browser
	initialise();
}


// -------------- Below here the proper game code start --------------


//Create and initialise all game components
var initialise = function(){

	//Main is a div acting as a canvas for HTML4
	canvas = document.getElementById("main");
	canvas.innerHTML = ""; //Clear out anything already in the canvas
	canvas.style.width = canvasWidth + 'px';
	canvas.style.height = canvasHeight + 'px';
	gameSpeed = gameStartSpeed;
	score = 0; // Reset score to 0
	
	//Set the background
	canvas.style.backgroundImage = "url(images/background.png)";
	
	//Make all text on the canvas white
	canvas.style.color = "white";
	
	//Create player
	player = Object();
	//Player starting coordinates - arbitrary but near to top left is better
	player.xCoordinate = playerForwardPosition; 
	player.yCoordinate = playerStartHeight;
	player.img = document.createElement('img');
	player.img.src = "images/choppa.png";
	player.img.style.position = "absolute";
	canvas.appendChild(player.img);
	player.rising = false; // starts traveling down
	
	//Calculate number of walls:
	//i.e. Width of the canvas divided by with of a wall (+ 1 extra to avoid seams)
	wallImageWidth = canvasHeight/8;
	numberOfWalls = (canvasWidth / wallImageWidth) + 1;
	
	//Create walls
	topWalls = Array();
	for(var i = 0; i < numberOfWalls; i++){
		var wall = Object();
		wall.img = document.createElement('img');
		wall.img.src = "images/wall.png";
		wall.img.height = canvasHeight/2;
		wall.xCoordinate = (i * wallImageWidth-4) + canvasWidth;
		wall.yCoordinate = 0 - Math.floor(Math.random()*canvasHeight / 2) - threshold;
		wall.img.style.position = "absolute";
		canvas.appendChild(wall.img);
		topWalls.push(wall); //Add the wall to the list of walls
	}
	bottomWalls = Array();
	for(var i = 0; i < numberOfWalls; i++){
		var wall = Object();
		wall.img = document.createElement('img');
		wall.img.src = "images/wall.png";
		wall.img.height = canvasHeight/2;
		wall.xCoordinate = (i * wallImageWidth-4) + canvasWidth;
		wall.yCoordinate = canvasHeight/2 + Math.floor(Math.random()*(canvasHeight / 2)) + threshold;
		wall.img.style.position = "absolute";
		canvas.appendChild(wall.img);
		bottomWalls.push(wall); //Add the wall to the list of walls
	}
	
	//Create container for displaying player score
	scoreContainer = document.createElement("div");
	scoreContainer.style.position = "absolute";
	scoreContainer.style.bottom = "5px";
	scoreContainer.style.left = "5px";
	scoreContainer.style.color = "white";
	scoreContainer.innerHTML = "Score : 0";
	canvas.appendChild(scoreContainer);
	
	//Add event listeners for canvas
	listenForEvent(window, "touchstart", handleClick);
	listenForEvent(window, "mousedown", handleClick);
	listenForEvent(window, "touchend", handleRelease);
	listenForEvent(window, "mouseup", handleRelease);
	listenForEvent(window, "touchmove", handleSwipe);
	
	//If the window is resized (changed to landscape/portrait ) reset the game with new size
	listenForEvent(window, "resize", resetGame);
	
	//Start update and draw loops
	update();
	draw();
	clock();
}

//Update method performs all maths calculations and movement for game objects
var update = function(){
	if(gameState == "ingame"){
		for(var i = 0; i < topWalls.length; i++){
			var wall = topWalls[i];
			wall.xCoordinate -= gameSpeed;
			if(wall.xCoordinate <= 0 - wall.img.width){
				wall.xCoordinate = canvasWidth;
				wall.yCoordinate = 0 - Math.floor(Math.random()*canvasHeight / 2) - threshold;
			}
			if(isCollision(player, wall)){
				gameState = "gameover";
			}
		}
		for(var i = 0; i < bottomWalls.length; i++){
			var wall = bottomWalls[i];
			wall.xCoordinate -= gameSpeed;
			if(wall.xCoordinate <= 0 - wall.img.width){
				wall.xCoordinate = canvasWidth;
				wall.yCoordinate = canvasHeight/2 + Math.floor(Math.random()*(canvasHeight / 2)) + threshold;
			}
			if(isCollision(player, wall)){
				gameState = "gameover";
			}
		}
		if(player.rising){
			player.yCoordinate -= playerRiseSpeed;
		}else{
			player.yCoordinate += playerFallSpeed;
		}
		setTimeout(update, 1000/frameRate);
	}
}

//Draw method draws all game compoments to the screen
var draw = function(){
	if(gameState == "menu"){
		//Menu text
		canvas.innerHTML = "<h1>Chopper<br/>Press to Begin</h1>"
			+"<button ontouchstart='showControls();' onclick='showControls();'>Start</button>";
	}else if(gameState == "controls"){
		canvas.innerHTML = "<h1>Controls</h1>"
			+"<p>Hold mouse button to float up</p>"
			+"<p>Release mouse button to float down</p>"
			+"<button ontouchstart='startGame();' onclick='startGame();'>Start</button>";
	}else if(gameState == "ingame"){
		//Draw background
		for(var i = 0; i < topWalls.length; i++){
			var wall = topWalls[i];
			wall.img.style.left = wall.xCoordinate + 'px';
			wall.img.style.top = wall.yCoordinate + 'px';
		}
		for(var i = 0; i < bottomWalls.length; i++){
			var wall = bottomWalls[i];
			wall.img.style.left = wall.xCoordinate + 'px';
			wall.img.style.top = wall.yCoordinate + 'px';
		}
		player.img.style.left = player.xCoordinate + 'px';
		player.img.style.top = player.yCoordinate + 'px';
		setTimeout(draw, 1000/frameRate);
	}else if(gameState == "gameover"){
		canvas.innerHTML = "<h1>Game Over<br/>Score: " + score + "<br/>Push to restart:</h1>"
			+"<button ontouchstart='startGame();' onclick='startGame()'>RESTART</button>";
	}
}

//Checks for collisions between 2 game objects
var isCollision = function(object1, object2){
	if(
		object1.xCoordinate > (object2.xCoordinate + object2.img.width)  ||
		object1.yCoordinate > (object2.yCoordinate + object2.img.height) ||
		(object1.xCoordinate + object1.img.width) < object2.xCoordinate  ||
		(object1.yCoordinate + object1.img.height) < object2.yCoordinate
	)
		return false;
	else
		return true;
}

//Increases score periodically
var clock = function(){
	if(gameState == "ingame"){
		score = score + 1;
		scoreContainer.innerHTML = "Score : " + score;
		//Increase the speed every 5 points
		if(score % speedIncreasePoints == 0){
			gameSpeed += speedIncreaseRate;
		}
		//Run this function again in 1 second
		setTimeout(clock, 1000);
	}
}

var handleClick = function(event){
	if(event.type == "touchstart"){
		event.preventDefault();
	}
	event.preventDefault();
		player.rising = true;
}

var handleRelease = function(event){
		player.rising = false;
}

var showControls = function(){
	gameState = "controls";
	draw();
}

var startGame = function(){
	gameState = "ingame";
	initialise();
	setTimeout(function(){player.rising=false}, 2); //This fixes a bug
}

var resetGame = function(){
	gameState = "menu";
	initialise();
}

//This stops the browser from scrolling when you
//swipe around
var handleSwipe = function(event){
	event.preventDefault();
}
