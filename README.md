##Website Performance Optimization - Project 4

###Getting started

All files can be downloaded here:
https://github.com/simonsanchez/simonsanchez.github.io

Original project files can be found here:
https://github.com/simonsanchez/udportfolio

###Objectives

1. Achieve a PageSpeed score of 90 or better, on both Mobile and Desktop, for the file index.html
2. Achieve a frame rate of 60fps for the file views/pizza.html

As of the time of this writing, I am currently hosting index.html with GitHub Pages. You can find a link to the live site at:
http://simonsanchez.github.io./

All testing was done on the most recent version of Google Chrome.

###Optimizations
#####index.html
The following optimizations were made:

* the main stylesheet was relatively short, and so all CSS was inlined into the head of the document
* instead of linking to the Google Fonts API, I applied @font-face rules (a big thanks to my instructor at UdaciHome for assisting me with this portion)
* the secondary stylesheet for printing was given a media query
* made all script tags asynchronous
* used compressed images provided by PageSpeed Insights

#####views/js/main.js
The following optimizations were made:
* moved various calculations outside the for-loop in the following function
```js
// Iterates through pizza elements on the page and changes their widths

/*
Simon Optimizations:
- We can make a variable to store all the pizzas and put it outside the loop. This avoids
having to obtain this collection on every iteration.
- Instead of using document.querySelectorAll, we utilize document.getElementsByClassName to gather the desired collection.
- We also note that dx and newwidth always yield the same values, respectivey, on every
iteration. We move these calculations outside the loop as well and use the first element in our pizza collection for the calculations.
*/
var allPizzas = document.getElementsByClassName("randomPizzaContainer");
var allPizzasLength = allPizzas.length;

var dx = determineDx(allPizzas[0], size);
var newwidth = (allPizzas[0].offsetWidth + dx) + 'px';

function changePizzaSizes(size) {
	for (var i = 0; i < allPizzasLength; i++) {
	  allPizzas[i].style.width = newwidth;
	}
}
```
* similar calcuations were carried out in this function
```js
// Moves the sliding background pizzas based on scroll position
function updatePositions() {
  frame++;
  window.performance.mark("mark_start_frame");

  /*
  Simon Optimizations:
  - As before, we use document.getElementsByClassName to obtain our collection.
  - We note that document.body.scrollTop/1250 is the same throughout every iteration.
  As with the previous calculations, we move it outside the loop.
  - We also note that inside our loop, the variable 'phase' is only taking five unique
  values. We calculate these values and store them in an array outside the loop.

  - Consider using transform instead of style.left ***
  */

  var items = document.getElementsByClassName("mover");
  var scrollVal = document.body.scrollTop/1250;

  var phaseValues = [];

  for(var i = 0; i < 5; i++) {
  	phaseValues.push(Math.sin(scrollVal + i));
  }

  var phase;

  for (var i = 0; i < items.length; i++) {
	switch (i % 5) {
		case 0:
			phase = phaseValues[0];
			break;
		case 1:
			phase = phaseValues[1];
			break;
		case 2:
			phase = phaseValues[2];
			break;
		case 3:
			phase = phaseValues[3];
			break;
		case 4:
			phase = phaseValues[4];
			break;
		default:
			console.log("Hello, World.");
	}

	items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }

  // User Timing API to the rescue again. Seriously, it's worth learning.
  // Super easy to create custom metrics.
  window.performance.mark("mark_end_frame");
  window.performance.measure("measure_frame_duration", "mark_start_frame", "mark_end_frame");
  if (frame % 10 === 0) {
	var timesToUpdatePosition = window.performance.getEntriesByName("measure_frame_duration");
	logAverageFrame(timesToUpdatePosition);
  }
}
```
* lastly, the number of pizzas on load were significantly reduced from 200 to 25
```js
// Generates the sliding pizzas when the page loads.
/*
Simon Optimizations:
- There is no need to generate 200 pizzas on load, just enough to fill the screen.
*/
document.addEventListener('DOMContentLoaded', function() {
  var cols = 8;
  var s = 256;
  for (var i = 0; i < 25; i++) {
	var elem = document.createElement('img');
	elem.className = 'mover';
	elem.src = "images/pizza.png";
	elem.style.height = "100px";
	elem.style.width = "73.333px";
	elem.basicLeft = (i % cols) * s;
	elem.style.top = (Math.floor(i / cols) * s) + 'px';
	document.querySelector("#movingPizzas1").appendChild(elem);
  }
  updatePositions();
});
```