<script>
 import P5 from 'p5-svelte';
 import Dpad from './Dpad.svelte';

 let inc
 let x = 0
 let y = 0

 const sketch = (p5) => {

	 p5.setup = () => {
		 p5.createCanvas(450, 450);
		 inc = p5.width * 0.1
	 };

	 p5.draw = () => {
		 // p5.rect(0, 0, 100, 100)
		 p5.clear()
		 p5.circle(p5.width/2 + x * inc, p5.height/2 + y * inc, inc)
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
	 x += e.detail.dir[0]
	 y += e.detail.dir[1]
 }

</script>

<main>
	<P5 {sketch} />

	<Dpad on:press={ handlePress } />

	<pre>
x: {x}
y: {y}
inc: {inc}
	</pre>
</main>
