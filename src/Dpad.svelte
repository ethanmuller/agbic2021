<script>

import { createEventDispatcher } from 'svelte';

const dispatch = createEventDispatcher();

 const DIRS = {
     up: [0,-1],
     down: [0,1],
     left: [-1,0],
     right: [1,0],
 }

 let dpadPressed = false;
 let dpadDirClass = ''


 const dpadPress = function(e) {
     // this nasty little function takes a touchstart or mousedown event
     // and determines which direction was pressed
     const rect = e.target.getBoundingClientRect()
     const elCenterPos = [rect.left + rect.width/2, rect.top + rect.height/2]
     let point;
     let dir;

     if (e.type === 'touchstart') {
         point = [e.touches[e.touches.length - 1].clientX, e.touches[e.touches.length - 1].clientY]
     } else if (e.type === 'mousedown') {
         point = [e.clientX, e.clientY]
     }

     let p = [point[0] - elCenterPos[0], point[1] - elCenterPos[1]]

     if (Math.abs(p[0]) > Math.abs(p[1])) {
         if (p[0] > 0) {
             dpadGo(DIRS.right)
         } else {
             dpadGo(DIRS.left)
         }
     } else {
         if (p[1] > 0) {
             dpadGo(DIRS.down)
         } else {
             dpadGo(DIRS.up)
         }
     }
 }
 const dpadGo = function(dir) {
     if (dpadPressed) {
         return
     }

     dpadPressed = true;

     switch(dir) {
         case DIRS.up:
             dpadDirClass = 'up'
             break;
         case DIRS.down:
             dpadDirClass = 'down'
             break;
         case DIRS.left:
             dpadDirClass = 'left'
             break;
         case DIRS.right:
             dpadDirClass = 'right'
             break;
     }

     // this.pressSound()
     // this.$emit('dpadPress', dir)
     dispatch('press', { dir })
 }

 const dpadRelease = function() {
     dpadPressed = false
     dpadDirClass = ''

     dispatch('release')
 }


 function handleKeydown(e) {
     console.log(e)
     switch(e.key) {
         case 'ArrowUp':
         case 'w':
             dpadGo(DIRS.up)
             break;
         case 'ArrowDown':
         case 's':
             dpadGo(DIRS.down)
             break;
         case 'ArrowLeft':
         case 'a':
             dpadGo(DIRS.left)
             break;
         case 'ArrowRight':
         case 'd':
             dpadGo(DIRS.right)
             break;
     }
     // if (e.key === 'ArrowUp') {
     // 		 dpadGo(DIRS.up)
     //     e.preventDefault()
     //     playNote('C2')
     // }
 }

 function handleKeyup(e) {
     console.log(e)
     switch(e.key) {
         case 'ArrowUp':
         case 'w':
         case 'ArrowDown':
         case 's':
         case 'ArrowLeft':
         case 'a':
         case 'ArrowRight':
         case 'd':
             dpadRelease()
             break;
     }
     //     e.preventDefault()
     //     stopNote()
     // }
 }
</script>

<svelte:window on:keydown={handleKeydown} on:keyup={handleKeyup}/>

<div class="ui">

    <!-- dpad -->
    <button id="dpad" class={ dpadDirClass }
                on:touchstart|preventDefault={ dpadPress }
                on:mousedown|preventDefault={ dpadPress }
                on:touchend|preventDefault={ dpadRelease }
                on:mouseup|preventDefault={ dpadRelease }
    >
        <span class="y"></span>
        <span class="x"></span>
    </button>
</div>

<style>
 .ui {
	 --bg: #666BAB;
     background-color: var(--bg);
 }

#dpad {
  --length: 9rem;
  --thicc: 40px;
  --grace: 1rem;

  border: none;

  width: var(--length);
  height: var(--length);

  position: relative;

  transition: none !important;

  padding: 5rem;
}

 .dpad--pressed {
 }

#dpad.up {
  transform: translate3d(0, -2px, 0);
}

#dpad.down {
  transform: translate3d(0, 2px, 0);
}

#dpad.left {
  transform: translate3d(-2px, 0, 0);
}

#dpad.right {
  transform: translate3d(2px, 0, 0);
}

button#dpad {
  background: transparent;
}

#dpad .y {
  pointer-events: none;
  display: block;
  background: #DBDBDB;
  width: var(--thicc);
  position: absolute;
  top: var(--grace);
  bottom: var(--grace);
  left: 50%;
  transform: translateX(-50%);
  border-radius: 1rem;
}

#dpad .x {
  pointer-events: none;
  display: block;
  background: #DBDBDB;
  height: var(--thicc);
  position: absolute;
  top: 50%;
  left: var(--grace);
  right: var(--grace);
  transform: translateY(-50%);
  border-radius: 1rem;
}
</style>
