<script>
 import { syllable } from 'syllable'
 import { onMount } from 'svelte'
 import { Story } from 'inkjs'
 import P5 from 'p5-svelte';
 import Dpad from './Dpad.svelte';


 let currentLine = ''
 let currentSpeaker = null
 let paragraphs = []
 let story
 let choices = []
 let mode = 'WORLD'
 let waitingForChoice = false
 let portrait = 'p-gus---2.png'
 let syl = 0
 let dialogState = null
 let currentBubble = 0
 let bubbles = []

 onMount(async () => {
     const res = await fetch(`/gus.json`)
     const json = await res.json()
     story = new Story(json)
     // continueStory()
 })

 function currentLineToParagraph() {
     paragraphs.push({ text: currentLine, speaker: currentSpeaker })
     paragraphs = paragraphs

     currentLine = ''
     currentSpeaker = null
 }

 function continueStory() {
     while (story.canContinue) {
         let nextLine = story.Continue()

         if (nextLine.includes(': ')) {
             nextLine = nextLine.split(': ')[1]
         }

         currentLine += nextLine
     }

     bubbles = currentLine.split('\n...\n')
     console.log(bubbles)
     currentBubble = 0

     choices = story.currentChoices
     // getParagraph()
     // currentLineToParagraph()
 }

 function choose(index) {
     waitingForChoice = false
     currentLine = ''
     paragraphs = []
     story.ChooseChoiceIndex(index)
     if (story.canContinue) {
         story.Continue()
     }
     continueStory()
 }

 function backToWorld() {
     currentLine = ''
     waitingForChoice = false
     mode = 'WORLD'
 }

 function advance() {
     const conversationOver = story && !story.canContinue && choices.length === 0 && currentBubble === bubbles.length - 1

     if (conversationOver) {
         return backToWorld()
     }

     if (currentBubble < bubbles.length - 1) {
        currentBubble = (currentBubble + 1)
     } else {
         waitingForChoice = true
     }
 }

 function handlePrimary() {
     if (mode === 'ENGAGEMENT') {
         advance()
     } else if (mode === 'WORLD') {
         console.log('ur in the world')
     }
 }

 let inc
 let x = 0
 let y = 0
 let ox = x
 let oy = y
 let vx = x
 let vy = y
 let s = 3
 let pDir = 1
 let dpadPressed = false
 let lastMove = Date.now()
 const moveTimeout = 300
 // const moveTimeout = 600

 const map = `
.........
.........
..U.T....
.........
.........
.........
.....T...
.........
.........
 `.trim().split('\n')
 // T is at x: 5, y: 3

function mapGet(x, y) {
	 if (map[y] && map[y][x]) {
		 return map[y][x]
	 }

	 return undefined
}

 const legend = {
	 "U": { name: "You" },
	 "T": { name: "Them" },
 }

 const room = []

 let guy

 function populateRoom() {
	 for (let cy = 0; cy < map.length; cy++) {
		 for (let cx = 0; cx < map[0].length; cx++) {
			 const thing = mapGet(cx, cy)

			 switch(thing) {
				 case 'U':
					 x = cx
					 y = cy
                     ox = x
                     oy = y
					 break;
				 case 'T':
			  		 room.push({
						 color: legend[thing],
						 x: cx,
						 y: cy,
					 })
					 break;
			 }

			 // if (thing === 'U') {
			 //
			 // 			 } else if (thing in legend) {
			 // 				 room.push({
			 // 					 color: legend[thing],
			 // 					 x,
			 // 					 y,
			 // 				 })
			 // 			 }
		 }
	 }
 }

 populateRoom()

 // ========
 // OBJECTS
 // ========
 //
 // Background
 // GREEN
 //
 // Target
 // DarkBlue
 //
 // Wall
 // BROWN
 //
 // Player
 // Blue
 //
 // Crate
 // Orange
 //
 // =======
 // LEGEND
 // =======
 //
 // . = Background
 // # = Wall
 // P = Player
 // * = Crate
 // @ = Crate and Target
 // O = Target
 //
 // =======
 // SOUNDS
 // =======
 //
 // ================
 // COLLISIONLAYERS
 // ================
 //
 // Background
 // Target
 // Player, Wall, Crate
 //
 // ======
 // RULES
 // ======
 //
 // [ >  Player | Crate ] -> [  >  Player | > Crate  ]
 //
 // ==============
 // WINCONDITIONS
 // ==============
 //
 // All Target on Crate
 //
 // =======
 // LEVELS
 // =======
 //
 // #########
 // #.......#
 // #.....@.#
 // #.P.*.O.#
 // #.......#
 // #.......#
 // #########





function lerp(v0, v1, t) {
    return v0*(1-t)+v1*t
}

 const drawMap = (p5) => {
	 p5.background('#ff77a8')

	 // dot grid
	 for (let i = 0; i <= 10; i++) {
		 for (let j = 0; j <= 10; j++) {
			 p5.noStroke()
			 p5.fill(0, 50)
			 p5.circle(i*inc, j*inc, 3)
		 }
	 }
 }

 const drawPlayer = (p5) => {
	 p5.push()
	 p5.noSmooth()
	 const w = guy[1].width * s
	 const h = guy[1].height * s
	 // p5.scale(dir, 1)
	 p5.imageMode(p5.CENTER)
	 p5.image(guy[pDir], vx * inc, vy * inc, w, h)
	 p5.pop()
 }

 const sketch = (p5) => {

	 p5.setup = () => {
		 p5.createCanvas(450, 450);
		 inc = p5.width / map.length

		 guy = [
			 p5.loadImage("zon-0.png"),
			 p5.loadImage("zon-1.png"),
			 p5.loadImage("zon-2.png"),
			 p5.loadImage("zon-3.png"),
		 ]
	 };

	 p5.draw = () => {
		 if (dpadPressed) {
			 const DIRS = [
				 [0,-1],
				 [1,0],
				 [0,1],
				 [-1,0],
			 ]
			 movePlayer(DIRS[pDir])
		 }

		 // p5.rect(0, 0, 100, 100)
		 drawMap(p5)


		 // draw map


	 const now = Date.now()
		 const percentDone = (now - lastMove)/moveTimeout
		 vx = lerp(ox, x, Math.min(percentDone, 1))
		 vy = lerp(oy, y, Math.min(percentDone, 1))
         // vx = x
         // vy = y
		 // const dx = x - vx
		 // 		 vx = vx + dx * 0.3
		 // 		 const dy = y - vy
		 // 		 vy = vy + dy * 0.3
		 // vy = y


		 // draw characters
		 room.forEach((thing) => {
			 p5.rectMode(p5.RADIUS)
			 p5.fill("#ffec27")
			 p5.rect(thing.x * inc, thing.y * inc, 5 * s, 5 * s)
		 })

		 drawPlayer(p5)

		 // p5.fill('red')
		 // p5.circle(x * inc, y * inc, 8)
		 // p5.stroke(1);
		 // 		 threshold = p5.random(0.75);
		 //
		 // 		 if (threshold < 0.1) p5.line(x, y, x + size, y + size);
		 // 		 else if (0.505 > threshold > 0.5) p5.line(x, y, x, y + size);
		 // 		 else p5.line(x, y + size, x + size, y);
		 //
		 // 		 x = x + size;
		 // 		 if (x > p5.width) {
		 // 			 x = 0;
		 // 			 y = y + size;
		 // 		 }
	 };
 };

 function handlePress(e) {
	 dpadPressed = true
     if (mode === 'WORLD') {
         movePlayer(e.detail.dir)
     }
 }

 function handleRelease() {
	 dpadPressed = false
 }

 function movePlayer(dir) {
	 const now = Date.now()

	 const [dx, dy] = dir

	 if (dy < 0) pDir = 0
	 if (dx > 0) pDir = 1
	 if (dy > 0) pDir = 2
	 if (dx < 0) pDir = 3

	 if (now - lastMove < moveTimeout) return

	 ox = x
	 oy = y

	 const tx = x + dx
	 const ty = y + dy

	 if (mapGet(tx, ty) === 'T') {
         mode = 'ENGAGEMENT'
         dpadPressed = false
         story.ChoosePathString('talk')
         continueStory()
	 } else {
		 x = tx
		 y = ty
	 }

	 lastMove = now
 }

 function open() {
    portrait = 'p-gus---3.png'
 }
 function close() {
     portrait = 'p-gus---2.png'
 }

</script>

<main>
	<div class="screen">
		<P5 {sketch} />
		<div class="dialog-box">
			<div class={ `dialog-box__inner ${mode === 'ENGAGEMENT' ? 'dialog-box__inner--is-open' : ''}` }>
                <div class="dialog-box__inner__text">{bubbles[currentBubble]}</div>
                <img src={ portrait } alt="">
            </div>
		</div>
	</div>

        {#if !waitingForChoice}
            <div class="controller">
                {#if mode === 'WORLD'}
                    <Dpad on:press={ handlePress } on:release={ handleRelease } />
                {/if}
                <!-- <input type="range" min="1" max="8" step="0.01" bind:value={ s }> -->
                <button class="btn btn--primary" on:click={ handlePrimary }></button>

            </div>
        {/if}

        {#if story && choices && choices.length > 0 && waitingForChoice}
            <div class="choice-list">
                {#each choices as choice,i}
                    <button class="ab__option" on:click|preventDefault={() => choose(i)}>{choice.text}</button>
                {/each}
            </div>
        {/if}

	<pre class="debug">
mode: <strong>{mode}</strong>

ink stuff
---------
currentLine: <span style="color: white">{currentLine}</span>
{#if mode === 'ENGAGEMENT'}
canContinue: {story && story.canContinue}
choices: {choices.length}
{/if}
{#if mode === 'WORLD'}
x: {x}
y: {y}
pDir: {pDir}
dpadPressed: {dpadPressed}
s: {s}
{/if}
	</pre>
</main>

<style>
 main {
     max-width: 450px;
     margin: 0 auto;

     position: relative;
 }

 .debug {
     display: none;

	 position: fixed;
	 top: 1rem;
	 left: 1rem;
	 color: black;
	 opacity: 0.2;
	 /* mix-blend-mode: difference; */
 }

 .screen {
	 position: relative;
	 overflow: hidden;
 }

 .controller {
     display: flex;
     justify-content: space-between;
     min-height: 168px;
 }

 .btn {
     width:  132px;
     height: 132px;
     align-self: center;
     margin-right:1.5rem;

     cursor: pointer;

     position: relative;
     border-radius: 99rem;

     background: red;
     border: none;

     margin-left: auto;
     background: #00CAD0;
 }

 /* .btn[disabled] {
    background: transparent;
    border: 3px solid #00CAD0;
    } */

 .btn::after {
     content: '';

     --pull: -16px;

     /* this pulls the element outside its parent,
     creating a larger clickable area */
     position: absolute;
     top: var(--pull);
     right: var(--pull);
     bottom: var(--pull);
     left: var(--pull);

     border-radius: 99rem;
 }

 .btn--primary {

 }

 .dialog-box {
	 position: absolute;
	 top: 0;	 left: 0;
	 width: 100%;

	 /* this stuff is a hack
	    for a 1/1 aspect ratio */
	 height: 0;
	 padding-bottom: 100%;
 }

 .dialog-box__inner {
     display: flex;
     flex-direction: column;
     justify-content: flex-end;

	 position: absolute;
	 top: 0;
	 left: 0;
	 width: 100%;
	 height: 100%;
	 background: #051EFF;
	 color: white;

	 transition: 500ms transform cubic-bezier(0.190, 1.000, 0.220, 1.000);
     transform-origin: 50% 100%;
	 transform: translate3d(0, 100%, 0) scale(1);

     white-space: pre; }

 .dialog-box__inner__text {
     margin-left: 1.5rem;
     justify-self: flex-end;
 }

 .dialog-box__inner img {
     align-self: flex-start;
     image-rendering: crisp-edges;
 }

 .dialog-box__inner--is-open {
	 transform: translate3d(0, 0px, 0) scale(1) rotateX(0);
 }

 .choice-list {
     display: flex;
     flex-direction: column;
     width: 100%;
     min-height: 150px;
     align-items: end;
 }

 .choice-list button {
     flex: 1;
     display: block;
     padding: 1.5rem;
     background: #A7ACE8;
     border-radius: 99rem;
     border: none;
     margin-bottom: 0.5rem;
 }

 .choice-list button:first-child {
     margin-top: 0.5rem;
 }

 .choice-list button:last-child {
     margin-bottom: none;
 }

</style>
