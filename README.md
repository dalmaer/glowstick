# Glowstick

A jQuery like API for making LEDs blink in delightful patterns.

## Colors

All colors are specified in an RGB array in this format:

    [255, 255, 255]

    pixels.fade([0, 0, 0], [255, 255, 255], 2000);

Alternatively [CSS color names](http://www.w3schools.com/tags/ref_colornames.asp) may be specified:

    pixels.fade('blue', 'red', 2000);

## Board

The `board` object allows you to run a few set commands (such as `clear`), and allows you to select pixels on the board. Most methods return a `Pixels` object which allows you to control the individual pixels.

    board.all().set('blue');

### every **board.every(duration, callback)**

Set an callback to be performed every `duration`. Use this instead of `setInterval` so that tasks can be cleared via `board.clear`

    board.every(50, function() {
      board.random().fade('white', 'black', 500);
    });

### after **board.after(duration, callback**

Set an callback to be performed after `duration`. Use this instead of `setTimeout` so that tasks can be cleared via `board.clear`

### clear **board.clear()**

Set all the pixels to black.

    board.clear();

### all **board.all()**

Select all of the pixels.

    var pixels = board.all();

### set **board.set(data)**

Set all of the pixels on the board. May be a color:

    board.set('red')

or an array of 64 colors:

    board.set([[r, g, b], [r, g, b], ...])

### at **board.at(index)**

Select a pixel at a given index:

    board.at(0).set('green');

Or select by x and y:

    board.at(2, 2).set('blue');

### row **board.row(index)**

Select a row of pixels.

    var pixels = board.row(0).fade('white', 'black', 1000);

### column **board.column(index)**

Select a column of pixels.

    var pixels = board.row(0).fade('black', 'white', 1000);

### random **board.random([number])**

Return a given number of random pixels.

    // get a single random pixel
    board.random().set('purple')

    // get 5 random pixels
    board.random(5).set('orange')

### js **board.js()**

Displays a lovely JS logo.

## Pixels

An array like object allowing you to operate on a set of pixels. Each command is generally run on the entire set of selected pixels.

### set **pixels.set([r, g, b])**

Set pixels to be a certain color.

### fade **pixels.fade([from,] to, duration, callback)**

Fade between two colors.

    board.row(2).fade('black','white', 1000);
    board.column(1).fade([255, 255, 255], [0, 0, 0], 1000);

### add

Add pixels to the currently selected set.

    var pixels = board.select(0, 6);
    pixels.add(board.select(7, 9));

    var pixels = board.row(0);
    pixels.add(board.row(1));

### filter

Filter the selected pixels.

    board.all().filter(function(pixel, i) {
      return i % 2 === 0;
    }).set('white');

### next **pixels.next({wrap: true})**

Select the next pixel by index. Pass `wrap: false` to prevent the first pixel from being selected if you are on the last pixel.

### previous **pixels.previous({wrap: true})**

Select the previous pixel by index. Pass `wrap: false` to prevent the last pixel from being selected if you are on the first pixel.

### above **pixels.above({wrap: true})*

Select the pixels above the present pixels. Pass `wrap: false` to prevent it from wrapping around the board.

### below **pixels.below({wrap: true})*

Select the pixels below the present pixels. Pass `wrap: false` to prevent it from wrapping around the board.

### left **pixels.left({wrap: true})*

Select the pixels to the left of the present pixels. Pass `wrap: false` to prevent it from wrapping around the board.

### right **pixels.right({wrap: true})*

Select the pixels to the right of the present pixels. Pass `wrap: false` to prevent it from wrapping around the board.

### Pixel

An individual pixel can be accessed via the array index either through the `board` object, or a `pixels object:

    var pixel = board[0];
    var pixels = board.select(0);

It contains the following properties:

- index
- x
- y
- color

Individual pixels objects should only be used to check the status of a pixel, and not to manipulate / set the pixel.

## Examples

### Random

    board.every(50, function() {
      board.random().fade('black', 'white', 500)
    });

### Walk

    function walk(pixel) {
      pixel.fade('white', 'teal', 100, function() {
        pixel.fade('teal','black', 100);
        walk(pixel.next());
      });
    }
    walk(board.at(0, 0));

### Column Walk

    function columnWalk(pixel) {
      pixel.fade('white', 'black', 400);
      board.after(100, function() {
        columnWalk(pixel.right());
      });
    }
    columnWalk(board.column(0));

### Blink (Red / Black)

    board.clear();

    function blink(pixel) {
      pixel.fade('white', 'red', 250, function() {
        pixel.fade('red', 'black', 250);
      });
    }

    board.every(25, function() {
      blink(board.random());
    });

### Blink (Blue / Orange)
  
    board.all().set('orange');

    function blink(pixel) {
      pixel.fade('orange', 'teal', 250, function() {
        pixel.fade('teal', 'orange', 250);
      });
    }
    
    board.every(25, function() {
      blink(board.random());
    });

### Star

    function streak(pixel) {
      function fade(pixel, callback) {
        pixel.fade([128, 128, 128], 'black', 250);
        board.after(50, callback);
      }
    
      pixel.fade('white', 'black', 1000);
    
      ['below','above','left','right'].forEach(function(direction) {
        function walk(pixel) {
          var next = pixel[direction]({wrap: false});
          if (next.length) {
            fade(next, function() {
              walk(next);
            })
          }
        }
        walk(pixel);
      });
    }
    
    board.every(500, function() {
      streak(board.random());
    });

### Gradient Fade

    function fadingWalk(pixel) {
      pixel.fade('blue', 'red', 2000);
      board.after(2000 / 64, function() {
        fadingWalk(pixel.next());
      });
    }
    fadingWalk(board.at(0));
