---
title: Game of Life
layout: post
description: A remix of Conway's game of life using timer, onclick and algorithms.
image: /images/game_of_life.svg
tags: [javascript]
---


{% comment %}
Conway's Game of Life https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life
{% endcomment %}

<style>
    #container {
    display: grid;
    }

    .cell {
    background-color: royalblue;
    border: 1px solid black;
    }
</style>

<div class="container">
    <header class="pb-3 mb-4 border-bottom border-primary text-dark">
        <span class="fs-4">Remix of Game of Life</span>
    </header>
    <!-- Buttons that link to functions in javascript -->
    <button onclick="start()" id="start-btn">start</button>
    <button onclick="step()">step</button>
    <!-- Container drawn by JavaScript -->
    <div id="container" class="container py-4">
    </div>
</div>

<script>

// Constants
let GRID_SIZE = 40;
let CELL_SIZE = "15px";
let container = document.getElementById("container");

// Add CSS to container to make it a grid
container.style["grid-template-columns"] = `repeat(${GRID_SIZE}, ${CELL_SIZE})`;
// Add squares to grid
for(let i=0; i<GRID_SIZE*GRID_SIZE; i++) {
    let di = document.createElement('div');
    di.style['width'] = CELL_SIZE;
    di.style['height'] = CELL_SIZE;
    di.onclick = clicked; // setting event listener to clicked function
    di.ondragstart = dragged;
    di.ondragover = dragged;
    di.className = 'cell';
    di.id = 'cell-'+i;
    container.appendChild(di)
}


const CELLS = Array(GRID_SIZE).fill().map(() => Array(GRID_SIZE).fill(0)); // create 2D array filled with '0'
const safeindex = (x, y) => !(x < 0 || x >= GRID_SIZE || y<0 || y >= GRID_SIZE); // Anonymous function to check bounds of index
// safeindex(0,0) = true
// safeindex(-1,-1) = false, because -1 falls out of bounds


function safeGet(x, y) {
    if(!safeindex(x,y)) return 0;
    if(CELLS[y][x] === 0) return 0; // blank square
    if(CELLS[y][x] === 1) return 1; // filled square
    if(CELLS[y][x] === 2) return 0; // previously blank square
    if(CELLS[y][x] === 3) return 1; // previously filled square
    console.error("AHH"); // should never get here
}

// helper function to set a square on the grid
function setCell(n, v) {
    let row = Math.floor(n/GRID_SIZE);
    let col = n%GRID_SIZE;
    CELLS[row][col] = v;
}

// Helper function to toggle a cell by it's index
function toggleCell(n) {
    let row = Math.floor(n/GRID_SIZE);
    let col = n%GRID_SIZE;
    CELLS[row][col] = CELLS[row][col] === 0 ? 1 : 0;
}


// Call this every interval, it will look through our CELLS array and reflect its data on the grid
function updateContainer() {
    CELLS.forEach((arr, r) => {
    arr.forEach((val, c) => {
        let n = r*GRID_SIZE + c;
        if(val === 1) {
        document.getElementById("cell-"+n).style['background-color'] = 'yellow';
        }else {
        document.getElementById("cell-"+n).style['background-color'] = 'royalblue'
        }
    });
    })
}


// will be called whenever a cell is clicked
function clicked() {
    const id = parseInt(this.id.substring(5), 10); // the id of a cell is "cell-XX" where XX is the index
    toggleCell(id); // if a cell is clicked we will toggle it
    updateContainer();
}

// Same thing as clicked except tied to drag event
function dragged() {
    const id = parseInt(this.id.substring(5), 10);
    setCell(id, 1);
    updateContainer()
}

// Randomly setting cells to 1 in the grid
function randomInit(n) {
    let max = GRID_SIZE*GRID_SIZE;
    for(let i=0; i<n; i++) {
    setCell(Math.floor(Math.random()*max), 1)
    }
    updateContainer();
}

// check how many alive neighbors a cell has
function getNeighbors(x,y) {
    const l = x-1;
    const r = x+1;
    const u = y-1;
    const d = y+1;

    return safeGet(l, y) +
            safeGet(r, y) +
            safeGet(x, u) +
            safeGet(x, d) +
            safeGet(l, u) +
            safeGet(r, u) +
            safeGet(l, d) +
            safeGet(r, d);
}

function step() {
    for(let y=0; y<GRID_SIZE; y++) {
        for(let x=0; x<GRID_SIZE; x++) { // for every square in the grid...

            const n = getNeighbors(x, y); // find how many alive neighbors it has
            const val = safeGet(x, y); // Get the value of the cell

            if(val === 0) { // if the cell is dead...
                if(n === 3) { // and it has three alive neighbors...
                    CELLS[y][x] = 2; // Make this cell come to life
                }
            } else { // If the cell is alive...
                if(n === 2 || n === 3) { // and it has ONLY 2 or ONLY 3 alive neighbors...
                    CELLS[y][x] = 3; // The cell gets to live another round
                }
            }

        }
    }

    // Run through every modified cell and convert it to either alive or dead
    for(let y=0; y<GRID_SIZE; y++) {
        for(let x=0; x<GRID_SIZE; x++) {
            CELLS[y][x] = Math.floor(CELLS[y][x]/2);
        }
    }

    updateContainer(); // call the update function to reflect changes in the
}


let paused = false;
const startbtn = document.getElementById("start-btn");

// used to toggle the animation
function togglePause() {
    paused = !paused;
    if(paused) {
    startbtn.innerHTML = "resume"
    } else {
    startbtn.innerHTML = "pause";
    }
}

// Used to start the animation
function start() {
    startbtn.innerHTML = "pause";
    startbtn.onclick = togglePause;

    // setInterval will call the function within it ever 100ms
    const interval = setInterval(function() {
    if(!paused) {
        step();
    }
    }, 100);
}

</script>