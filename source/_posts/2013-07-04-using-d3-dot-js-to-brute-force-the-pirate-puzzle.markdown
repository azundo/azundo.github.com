---
layout: post
title: "Using D3.js to Brute Force the Pirate Puzzle"
date: 2013-07-04 23:10
comments: true
published: false
categories: ["d3.js", "puzzles", "geometry"]
---

<style>
  .island {
    stroke: blue;
    stroke-width: 3;
    fill: transparent;
  }
  .tree {
    stroke: green;
    fill: green;
  }
  .grave {
    stroke: black;
    fill: black;
  }
  .flag {
    stroke: red;
    fill: red;
  }
  .treasure {
    stroke: yellow;
    fill: yellow;
  }
  .flag-path {
    stroke: black;
    fill: none;
  }
  .rise {
      stroke: red;
  }
  .run {
      stroke: blue;
  }
  .thick {
      stroke-width: 5px;
  }
</style>

<script src="http://d3js.org/d3.v3.min.js" charset="utf-8"></script>

SPOILER ALERT. This post contains an answer to puzzle 11 Treasure Island from
[http://gurmeet.net/puzzles](http://gurmeet.net/puzzles) which appeared on Hacker News recently. If you
want to solve it yourself first, check it out over there before reading on, or
just don't read too far down the page.

##The Puzzle
Here is the wording of the puzzle:

    An old parchment has directions to a treasure chest buried in an island:

    There is an unmarked grave and two tall oak trees. Walk from the grave to the
    left tree, counting the number of steps. Upon reaching the left tree, turn left
    by 90 degrees and walk the same number of steps. Mark the point with a flag.
    Return to the grave. Now, walk towards the right tree, counting the number of
    steps. Upon reaching the right tree, turn right by 90 degrees and walk the same
    number of steps. Mark this point with another flag. The treasure lies at the
    midpoint of the two flags.

    A party of sailors reached the island. They find a pair of tall oak trees
    merrily swaying in the wind. However, the unmarked grave is nowhere to be
    found. They are planning to dig up the entire island. It'll take a month. Can
    they do any better?


My high school geometry is a little fuzzy so I thought I'd do a little
empirical playing around with d3.js to get a sense of the answer before trying
to work out the math. Almost a sort of graphical brute force method. If you're new
to d3.js this should hopefully act as a good introduction as well.

If you want to skip ahead, the full gist is at
[https://gist.github.com/azundo/5928203](https://gist.github.com/azundo/5928203)
and you can use Mike Bostock's [bl.ocks.org](http://bl.ocks.org) to view it at
[http://bl.ocks.org/azundo/5928203](http://bl.ocks.org/azundo/5928203).

##Brute Forcing

###Set up

We'll start with a little bit of d3 boilerplate to set things up. There are a
few `g` elements to get our layers working properly but hopefully the comments
in the code make things clear.

```javascript
var width = 960,
  height = 600,
  // our svg container
  svg = d3.select('body').append('svg')
  .attr('width', width)
  .attr('height', height),
  // a group within our container that will hold our map features
  features = svg.append('g'),
  // a group within our features group that will show some of the underlying geometry
  geometries = features.append('g');
```

Next lets set up some parameters for our treasure island.

```javascript
var islandRadius = 300,
  // size of the features we are plotting on the map
  featureLength = 30,
  // a magic number for placing our trees. Feel free to play around with this number.
  treeOffset = 210,
  // an array of tree objects with x and y properties
  // our trees will have the same y coordinate and x coordinates centered
  // around the middle of the island.
  trees = [
      {
        x: treeOffset,
        y: treeOffset
    },
    {
        x: islandRadius*2 - treeOffset,
        y: treeOffset
    }],
  // an arbitrary initial point for our grave, we'll easily change it later through the graphic.
  grave = {x: islandRadius, y: islandRadius};
```

###Plotting the Static Features

Now we're ready to plot the static component of our map - the island and the trees.

```javascript
// create the island on top of everything. If we draw our features "over top"
// of the island then they will interfere with our event handling. Appending to
// the end of the svg container will put the island last in the svg hierarchy
// so it will always be on top. We'll give it a transparent fill so we can see
// through it.
svg.append('circle')
  .attr('r', islandRadius)
  .attr('cx', islandRadius)
  .attr('cy', islandRadius)
  .classed('island', true);


// create the trees - these are static so only draw them once
features.selectAll('.tree')
  .data(trees)
  .enter().append('image')
    .attr('xlink:href', 'images/tree.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    // getX and getY are simple helper functions to return properly offset x
    // and y coords for our square features
    .attr('x', getX)
    .attr('y', getY)
    .classed('tree', true);
```

If we fire this code up you'll see the following:

<div id="static"></div>
<script type="text/javascript">
(function(){
var width = 960,
  height = 600,
  // our svg container
  svg = d3.select('#static').append('svg')
  .attr('width', width)
  .attr('height', height),
  // a group within our container that will hold our map features
  features = svg.append('g'),
  // a group within our features group that will show some of the underlying geometry
  geometries = features.append('g');
var islandRadius = 300,
  // size of the features we are plotting on the map
  featureLength = 30,
  // a magic number for placing our trees. Feel free to play around with this number.
  treeOffset = 210,
  // an array of tree objects with x and y properties
  // our trees will have the same y coordinate and x coordinates centered
  // around the middle of the island.
  trees = [
      {
        x: treeOffset,
        y: treeOffset
    },
    {
        x: islandRadius*2 - treeOffset,
        y: treeOffset
    }],
  // an arbitrary initial point for our grave, we'll easily change it later through the graphic.
  grave = {x: islandRadius, y: islandRadius};

// create the island on top of everything. If we draw our features "over top"
// of the island then they will interfere with our event handling. Appending to
// the end of the svg container will put the island last in the svg hierarchy
// so it will always be on top. We'll give it a transparent fill so we can see
// through it.
svg.append('circle')
  .attr('r', islandRadius)
  .attr('cx', islandRadius)
  .attr('cy', islandRadius)
  .classed('island', true);

// create the trees - these are static so only draw them once
features.selectAll('.tree')
  .data(trees)
  .enter().append('image')
    .attr('xlink:href', '../../images/tree.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    // getX and getY are simple helper functions to return properly offset x
    // and y coords for our square features
    .attr('x', getX)
    .attr('y', getY)
    .classed('tree', true);

function getX(d) {
    return d.x - featureLength/2;
}

function getY(d) {
    return d.y - featureLength/2;
}

})();
</script>


Looking good. A couple of trees. Time to get plotting our flags and ultimately the
treasure chest location.

###Adding Flags

The tricky part here is getting the geometry right when calculating the
location of the flags for a given location of the grave. By filling in a couple
of extra triangles it becomes clear how we can do it. Here's a quick diagram
for the left tree:

<div id="tri"></div>

<script type="text/javascript">
var svg = d3.select('#tri').append('svg')
    .attr('width', 400)
    .attr('height', 280),
    line = d3.svg.line()
      .x(function(d) { return d.x; })
      .y(function(d) { return d.y; }),
    tree = {
        x: 120,
        y: 20
    },
    grave = {
        x: 370,
        y: 120
    },
    flag = {
        x: tree.x - (grave.y - tree.y),
        y: tree.y + (grave.x - tree.x)
    },
    featureLength = 40;
// plot the tree
svg.append('image')
    .attr('xlink:href', '../../images/tree.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    .attr('x', tree.x - featureLength / 2)
    .attr('y', tree.y - featureLength / 2)
    .classed('tree', true);
// plot the grave
svg.append('image')
    .attr('xlink:href', '../../images/grave.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    .attr('x', grave.x - featureLength / 2)
    .attr('y', grave.y - featureLength / 2)
    .classed('grave', true);
// plot the flag
svg.append('image')
    .attr('xlink:href', '../../images/flag.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    .attr('x', flag.x - featureLength / 2)
    .attr('y', flag.y - featureLength / 2)
    .classed('flag', true);
// draw the runs
svg.append('path')
    .attr('d', line([tree, {x: grave.x, y: tree.y}]))
    .classed('run thick', true);
svg.append('path')
    .attr('d', line([flag, {x: flag.x, y: tree.y}]))
    .classed('run thick', true);
// draw the rises
svg.append('path')
    .attr('d', line([grave, {x: grave.x, y: tree.y}]))
    .classed('rise thick', true);
svg.append('path')
    .attr('d', line([tree, {x: flag.x, y: tree.y}]))
    .classed('rise thick', true);
svg.append('path')
    .attr('d', line([grave, tree, flag]))
    .classed('flag-path thick', true);
</script>

Remembering that the y-coordinate values increase downwards on our svg canvas, we can see a
couple of equivalent triangles marked by the red, blue and black lines. The
black lines are the path our pirate would mark out,
the red $$rise = g_y - T_{Ly}$$ is the y-difference between the grave and the tree,
and and the blue $$run = g_x - T_{Lx}$$ is the x-difference between the grave and the tree.
The coordinates of the left
flag are then given by:

$$
\begin{aligned}
F_{Lx} & = T_{Lx} - rise \\
       & = T_{Lx} - (g_y - T_{Ly}) \\
\\
F_{Ly} & = T_{Ly} + run \\
       & = T_{Ly} + (g_x - T_{Lx})
\end{aligned}
$$

If we swap the grave and the flag in the above diagram, we can work out the
case for the right tree, turning to the right instead of the left:

$$
\begin{aligned}
F_{Rx} & = T_{Rx} + rise \\
       & = T_{Rx} + (g_y - T_{Ry}) \\
\\
F_{Ry} & = T_{Ry} - run \\
       & = T_{Ry} - (g_x - T_{Rx})
\end{aligned}
$$

So we end up with the following function for calculating our flag placements:


```javascript
// calculate the positions of the flags
function getFlagPositions(treePositions, gravePosition) {
  var leftTree, rightTree, rise, run, leftFlag, rightFlag;

  leftTree = treePositions[0];
  rightTree = treePositions[1];

  // get left flag position
  rise = gravePosition.y - leftTree.y;
  run = gravePosition.x - leftTree.x;
  // turn 90 to the left
  leftFlag = {
    x: leftTree.x - rise,
    y: leftTree.y + run
  };
  // get right flag position
  rise = gravePosition.y - rightTree.y;
  run = gravePosition.x - rightTree.x;
  // turn 90 to the right
  rightFlag = {
    x: rightTree.x + rise,
    y: rightTree.y - run
  };
  return [leftFlag, rightFlag];
}
```


We'll need to dynamically update the positions of these flags so lets create a
draw function and call it. At the same time we'll draw our grave in as well as
the triangles that show the walking directions for illustrative purposes. Check
out the [final version of the code](https://gist.github.com/azundo/5928203) to
see the full `getPathPoints` function.

```javascript
function draw() {
  // calculate positions
  var flags = getFlagPositions(trees, grave),
      paths = getPathPoints(trees, flags, grave);


  // draw the grave
  features.selectAll('.grave')
    .data([grave])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', 'img/grave.svg')
      .classed('grave', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);

  
  // draw the flags
  features.selectAll('.flag')
    .data(flags)
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', 'img/flag.svg')
      .classed('flag', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);
  
  // draw triangles showing the flag calculations
  // these are in the geometries group so they appear underneath the other features
  geometries.selectAll('.flag-path')
    .data(paths)
      .attr('d', line)
    .enter().append('path')
      .classed('flag-path', true)
      .attr('d', line);
}
draw();
```

<div id="flags"></div>
<script type="text/javascript">
(function(){
var width = 960,
  height = 600,
  // our svg container
  svg = d3.select('#flags').append('svg')
  .attr('width', width)
  .attr('height', height),
  // a group within our container that will hold our map features
  features = svg.append('g'),
  // a group within our features group that will show some of the underlying geometry
  geometries = features.append('g');
var islandRadius = 300,
  // size of the features we are plotting on the map
  featureLength = 30,
  // a magic number for placing our trees. Feel free to play around with this number.
  treeOffset = 210,
  // an array of tree objects with x and y properties
  // our trees will have the same y coordinate and x coordinates centered
  // around the middle of the island.
  trees = [
      {
        x: treeOffset,
        y: treeOffset
    },
    {
        x: islandRadius*2 - treeOffset,
        y: treeOffset
    }],
  // an arbitrary initial point for our grave, we'll easily change it later through the graphic.
  grave = {x: islandRadius, y: islandRadius},
  // helper function for generating path values for lines
  line = d3.svg.line()
    .x(function(d) { return d.x; })
    .y(function(d) { return d.y; });

// create the island on top of everything. If we draw our features "over top"
// of the island then they will interfere with our event handling. Appending to
// the end of the svg container will put the island last in the svg hierarchy
// so it will always be on top. We'll give it a transparent fill so we can see
// through it.
svg.append('circle')
  .attr('r', islandRadius)
  .attr('cx', islandRadius)
  .attr('cy', islandRadius)
  .classed('island', true);

// create the trees - these are static so only draw them once
features.selectAll('.tree')
  .data(trees)
  .enter().append('image')
    .attr('xlink:href', '../../images/tree.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    // getX and getY are simple helper functions to return properly offset x
    // and y coords for our square features
    .attr('x', getX)
    .attr('y', getY)
    .classed('tree', true);

function draw() {
  // calculate positions
  var flags = getFlagPositions(trees, grave),
      paths = getPathPoints(trees, flags, grave);


  // draw the grave
  features.selectAll('.grave')
    .data([grave])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/grave.svg')
      .classed('grave', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);

  
  // draw the flags
  features.selectAll('.flag')
    .data(flags)
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/flag.svg')
      .classed('flag', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);
  
  // draw triangles showing the flag calculations
  // these are in the geometries group so they appear underneath the other features
  geometries.selectAll('.flag-path')
    .data(paths)
      .attr('d', line)
    .enter().append('path')
      .classed('flag-path', true)
      .attr('d', line);
}
draw();

function getFlagPositions(treePositions, gravePosition) {
  var leftTree, rightTree, rise, run, leftFlag, rightFlag;

  leftTree = treePositions[0];
  rightTree = treePositions[1];

  // get left flag position
  rise = gravePosition.y - leftTree.y;
  run = gravePosition.x - leftTree.x;
  // turn 90 to the left
  leftFlag = {
    x: leftTree.x - rise,
    y: leftTree.y + run
  };
  // get right flag position
  rise = gravePosition.y - rightTree.y;
  run = gravePosition.x - rightTree.x;
  // turn 90 to the right
  rightFlag = {
    x: rightTree.x + rise,
    y: rightTree.y - run
  };
  return [leftFlag, rightFlag];
}

function getPathPoints(treePositions, flagPositions, grave) {
  // create two lines, grave -> tree -> flag -> grave for each tree
  return [
    [
      grave,
      treePositions[0],
      flagPositions[0],
      grave
    ],
    [
      grave,
      treePositions[1],
      flagPositions[1],
      grave
    ]
  ]
}

function getX(d) {
    return d.x - featureLength/2;
}

function getY(d) {
    return d.y - featureLength/2;
}

})();
</script>

We're getting somewhere. Now lets connect our mouse position to the
grave position and watch our flags dance around.

###Making the Flags Move


```javascript
// move grave location to wherever the mouse is
svg.select('.island')
  .on('mousemove', function() {
      // 8 is a magic number to make things look nice, on my screen at least.
      var mousePosition = d3.mouse(this);
      grave.x = mousePosition[0] - 8;
      grave.y = mousePosition[1] - 8;
      draw();
  });
```

<div id="moving-flags"></div>
<script type="text/javascript">
(function(){
var width = 960,
  height = 600,
  // our svg container
  svg = d3.select('#moving-flags').append('svg')
  .attr('width', width)
  .attr('height', height),
  // a group within our container that will hold our map features
  features = svg.append('g'),
  // a group within our features group that will show some of the underlying geometry
  geometries = features.append('g');
var islandRadius = 300,
  // size of the features we are plotting on the map
  featureLength = 30,
  // a magic number for placing our trees. Feel free to play around with this number.
  treeOffset = 210,
  // an array of tree objects with x and y properties
  // our trees will have the same y coordinate and x coordinates centered
  // around the middle of the island.
  trees = [
      {
        x: treeOffset,
        y: treeOffset
    },
    {
        x: islandRadius*2 - treeOffset,
        y: treeOffset
    }],
  // an arbitrary initial point for our grave, we'll easily change it later through the graphic.
  grave = {x: islandRadius, y: islandRadius},
  // helper function for generating path values for lines
  line = d3.svg.line()
    .x(function(d) { return d.x; })
    .y(function(d) { return d.y; });

// create a mask to keep features from displaying off of the island
svg.append('g')
  .append('clipPath')
    .attr('id','island-mask-1')
    .append('circle')
      .attr('r', islandRadius)
      .attr('cx', islandRadius)
      .attr('cy', islandRadius);

// apply it to the features group
features.attr('clip-path', 'url(#island-mask-1)');

// create the island on top of everything. If we draw our features "over top"
// of the island then they will interfere with our event handling. Appending to
// the end of the svg container will put the island last in the svg hierarchy
// so it will always be on top. We'll give it a transparent fill so we can see
// through it.
svg.append('circle')
  .attr('r', islandRadius)
  .attr('cx', islandRadius)
  .attr('cy', islandRadius)
  .classed('island', true);

// create the trees - these are static so only draw them once
features.selectAll('.tree')
  .data(trees)
  .enter().append('image')
    .attr('xlink:href', '../../images/tree.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    // getX and getY are simple helper functions to return properly offset x
    // and y coords for our square features
    .attr('x', getX)
    .attr('y', getY)
    .classed('tree', true);

svg.select('.island')
  .on('mousemove', function() {
      // 8 is a magic number to make things look nice, on my screen at least.
      var mousePosition = d3.mouse(this);
      grave.x = mousePosition[0] - 8;
      grave.y = mousePosition[1] - 8;
      draw();
  });

function draw() {
  // calculate positions
  var flags = getFlagPositions(trees, grave),
      paths = getPathPoints(trees, flags, grave);


  // draw the grave
  features.selectAll('.grave')
    .data([grave])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/grave.svg')
      .classed('grave', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);

  
  // draw the flags
  features.selectAll('.flag')
    .data(flags)
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/flag.svg')
      .classed('flag', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);
  
  // draw triangles showing the flag calculations
  // these are in the geometries group so they appear underneath the other features
  geometries.selectAll('.flag-path')
    .data(paths)
      .attr('d', line)
    .enter().append('path')
      .classed('flag-path', true)
      .attr('d', line);
}
draw();

function getFlagPositions(treePositions, gravePosition) {
  var leftTree, rightTree, rise, run, leftFlag, rightFlag;

  leftTree = treePositions[0];
  rightTree = treePositions[1];

  // get left flag position
  rise = gravePosition.y - leftTree.y;
  run = gravePosition.x - leftTree.x;
  // turn 90 to the left
  leftFlag = {
    x: leftTree.x - rise,
    y: leftTree.y + run
  };
  // get right flag position
  rise = gravePosition.y - rightTree.y;
  run = gravePosition.x - rightTree.x;
  // turn 90 to the right
  rightFlag = {
    x: rightTree.x + rise,
    y: rightTree.y - run
  };
  return [leftFlag, rightFlag];
}

function getPathPoints(treePositions, flagPositions, grave) {
  // create two lines, grave -> tree -> flag -> grave for each tree
  return [
    [
      grave,
      treePositions[0],
      flagPositions[0],
      grave
    ],
    [
      grave,
      treePositions[1],
      flagPositions[1],
      grave
    ]
  ]
}

function getX(d) {
    return d.x - featureLength/2;
}

function getY(d) {
    return d.y - featureLength/2;
}

})();
</script>

Can you guess the answer to the riddle yet? Let's add a calculation of the
flags' midpoint and plot our treasure chest on the map.

###The Chest

```javascript
// modified draw function
function draw() {
  ...
  var treasure = getTreasurePosition(flags);
  // draw the treasure location
  features.selectAll('.treasure')
    .data([treasure])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', 'img/treasure_chest.svg')
      .classed('treasure', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);
    ...
}

function getTreasurePosition(flagPositions) {
  // averages the x and y values of the two flag positions
  return {
    x: (flagPositions[0].x + flagPositions[1].x) / 2,
    y: (flagPositions[0].y + flagPositions[1].y) / 2
  }
}
```

<div id="treasure"></div>
<script type="text/javascript">
(function(){
var width = 960,
  height = 600,
  // our svg container
  svg = d3.select('#treasure').append('svg')
  .attr('width', width)
  .attr('height', height),
  // a group within our container that will hold our map features
  features = svg.append('g'),
  // a group within our features group that will show some of the underlying geometry
  geometries = features.append('g');
var islandRadius = 300,
  // size of the features we are plotting on the map
  featureLength = 30,
  // a magic number for placing our trees. Feel free to play around with this number.
  treeOffset = 210,
  // an array of tree objects with x and y properties
  // our trees will have the same y coordinate and x coordinates centered
  // around the middle of the island.
  trees = [
      {
        x: treeOffset,
        y: treeOffset
    },
    {
        x: islandRadius*2 - treeOffset,
        y: treeOffset
    }],
  // an arbitrary initial point for our grave, we'll easily change it later through the graphic.
  grave = {x: islandRadius, y: islandRadius},
  // helper function for generating path values for lines
  line = d3.svg.line()
    .x(function(d) { return d.x; })
    .y(function(d) { return d.y; });
// create a mask to keep features from displaying off of the island
svg.append('g')
  .append('clipPath')
    .attr('id','island-mask-2')
    .append('circle')
      .attr('r', islandRadius)
      .attr('cx', islandRadius)
      .attr('cy', islandRadius);

// apply it to the features group
features.attr('clip-path', 'url(#island-mask-2)');

// create the island on top of everything. If we draw our features "over top"
// of the island then they will interfere with our event handling. Appending to
// the end of the svg container will put the island last in the svg hierarchy
// so it will always be on top. We'll give it a transparent fill so we can see
// through it.
svg.append('circle')
  .attr('r', islandRadius)
  .attr('cx', islandRadius)
  .attr('cy', islandRadius)
  .classed('island', true);

// create the trees - these are static so only draw them once
features.selectAll('.tree')
  .data(trees)
  .enter().append('image')
    .attr('xlink:href', '../../images/tree.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    // getX and getY are simple helper functions to return properly offset x
    // and y coords for our square features
    .attr('x', getX)
    .attr('y', getY)
    .classed('tree', true);

svg.select('.island')
  .on('mousemove', function() {
      // 8 is a magic number to make things look nice, on my screen at least.
      var mousePosition = d3.mouse(this);
      grave.x = mousePosition[0] - 8;
      grave.y = mousePosition[1] - 8;
      draw();
  });

function draw() {
  // calculate positions
  var flags = getFlagPositions(trees, grave),
      paths = getPathPoints(trees, flags, grave),
      treasure = getTreasurePosition(flags);

  // draw the treasure location
  features.selectAll('.treasure')
    .data([treasure])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/treasure_chest.svg')
      .classed('treasure', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);


  // draw the grave
  features.selectAll('.grave')
    .data([grave])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/grave.svg')
      .classed('grave', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);

  
  // draw the flags
  features.selectAll('.flag')
    .data(flags)
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/flag.svg')
      .classed('flag', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);
  
  // draw triangles showing the flag calculations
  // these are in the geometries group so they appear underneath the other features
  geometries.selectAll('.flag-path')
    .data(paths)
      .attr('d', line)
    .enter().append('path')
      .classed('flag-path', true)
      .attr('d', line);
}
draw();

function getFlagPositions(treePositions, gravePosition) {
  var leftTree, rightTree, rise, run, leftFlag, rightFlag;

  leftTree = treePositions[0];
  rightTree = treePositions[1];

  // get left flag position
  rise = gravePosition.y - leftTree.y;
  run = gravePosition.x - leftTree.x;
  // turn 90 to the left
  leftFlag = {
    x: leftTree.x - rise,
    y: leftTree.y + run
  };
  // get right flag position
  rise = gravePosition.y - rightTree.y;
  run = gravePosition.x - rightTree.x;
  // turn 90 to the right
  rightFlag = {
    x: rightTree.x + rise,
    y: rightTree.y - run
  };
  return [leftFlag, rightFlag];
}

function getPathPoints(treePositions, flagPositions, grave) {
  // create two lines, grave -> tree -> flag -> grave for each tree
  return [
    [
      grave,
      treePositions[0],
      flagPositions[0],
      grave
    ],
    [
      grave,
      treePositions[1],
      flagPositions[1],
      grave
    ]
  ]
}

function getTreasurePosition(flagPositions) {
  // averages the x and y values of the two flag positions
  return {
    x: (flagPositions[0].x + flagPositions[1].x) / 2,
    y: (flagPositions[0].y + flagPositions[1].y) / 2
  }
}

function getX(d) {
    return d.x - featureLength/2;
}

function getY(d) {
    return d.y - featureLength/2;
}

})();
</script>

Success! We've now got a clear answer - there's only a single location where
the chest could be. If you play around with the grave position it becomes
obvious that the chest's x-coordinate is the midpoint of the two trees'
x-coordinates and the y coordinate is simply that same midpoint distance added
to the y-coordinate of the trees. Here we're assuming the trees are at the same
y-coordinate and remember that y increases in the downward direction on our
screen. (The y-coordinate assumption is ok because we could always redraw our
coordinate system so that the two trees are level in the y direction.)

##The Math##

Now that we've played around with getting the example to work and
seen that it is a single point solution (not a line or other area function) we
can take a more informed approach to the math.


###The X-coordinate

We'll use the trick that our x and y coordinates are independent. Lets work in
the x-direction first. We know our final x-coordinate is the average of our two
flags' x-coordinates. Lets start with the left tree's flag. Here we use
$$T_{Lx}$$ as the left tree's x-coordinate, $$F_{Lx}$$ as the left flag's
x-coordinate, $$g_{x}$$ as the grave's x-coordinate and $$X_{x}$$ (since X
marks the spot) as the chest's x-coordinate. Going back to our earlier
formulas:

$$
\begin{aligned}
F_{Lx} & = T_{Lx} - (g_{y} - T_{Ly}) \\
F_{Rx} & = T_{Rx} + (g_{y} - T_{Ry})
\end{aligned}
$$

Averaging these two we can see that our $g_{y}$ term cancels out.

$$
\begin{aligned}
X_{x} & = \frac{F_{Lx} + F_{Rx}}{2} \\
& = \frac{T_{Lx} - (g_{y} - T_{Ly}) + T_{Rx} + (g_{y} - T_{Ry})}{2} \\
& = \frac{T_{Lx} + T_{Ly} + T_{Rx} - T_{Ry}}{2}
\end{aligned}
$$

In our case where $$T_{Ly} == T_{Ry}$$ we get $$X_{x}$$ as the average of
$$T_{Lx}$$ and $$T_{Rx}$$, not dependent on the graveyard location, as we saw
in our brute force method.

###The Y-coordinate

Now we can do the same for the y-coordinate.

$$
\begin{aligned}
F_{Ly} & = T_{Ly} + (g_{x} - T_{Lx}) \\
F_{Ry} & = T_{Ry} - (g_{x} - T_{Rx}) \\
\\
X_{y} & = \frac{F_{Ly} + F_{Ry}}{2} \\
      & = \frac{T_{Ly} + g_{x} - T_{Lx} + T_{Ry} - g_{x} + T_{Rx}}{2} \\
      & = \frac{T_{Ly} + T_{Ry} + T_{Rx} - T_{Lx}}{2}
\end{aligned}
$$

In the case where
$$T_{Ly} == T_{Ry} == T_{y}$$ then 
$$X_{y} = T_{y} + \frac{T_{Rx} - T_{Lx}}{2}$$, the result we saw earlier in
the brute force method.


##One More Thing

We actually don't have it quite right yet, there is an omission. In the cases
where the grave lies above the trees (from our vantage point looking down at
the map of the island) the left and right trees are actually reversed. Here's
the final implementation accounting for this possibility. Check out the 
gist at
[https://gist.github.com/azundo/5928203](https://gist.github.com/azundo/5928203)
for the full final version.

<div id="full"></div>
<script type="text/javascript">
(function(){
var width = 960,
  height = 600,
  // our svg container
  svg = d3.select('#full').append('svg')
  .attr('width', width)
  .attr('height', height),
  // a group within our container that will hold our map features
  features = svg.append('g'),
  // a group within our features group that will show some of the underlying geometry
  geometries = features.append('g');
var islandRadius = 300,
  // size of the features we are plotting on the map
  featureLength = 30,
  // a magic number for placing our trees. Feel free to play around with this number.
  treeOffset = 210,
  // an array of tree objects with x and y properties
  // our trees will have the same y coordinate and x coordinates centered
  // around the middle of the island.
  trees = [
      {
        x: treeOffset,
        y: treeOffset
    },
    {
        x: islandRadius*2 - treeOffset,
        y: treeOffset
    }],
  // an arbitrary initial point for our grave, we'll easily change it later through the graphic.
  grave = {x: islandRadius, y: islandRadius},
  // helper function for generating path values for lines
  line = d3.svg.line()
    .x(function(d) { return d.x; })
    .y(function(d) { return d.y; });
// create a mask to keep features from displaying off of the island
svg.append('g')
  .append('clipPath')
    .attr('id','island-mask-3')
    .append('circle')
      .attr('r', islandRadius)
      .attr('cx', islandRadius)
      .attr('cy', islandRadius);

// apply it to the features group
features.attr('clip-path', 'url(#island-mask-3)');

// create the island on top of everything. If we draw our features "over top"
// of the island then they will interfere with our event handling. Appending to
// the end of the svg container will put the island last in the svg hierarchy
// so it will always be on top. We'll give it a transparent fill so we can see
// through it.
svg.append('circle')
  .attr('r', islandRadius)
  .attr('cx', islandRadius)
  .attr('cy', islandRadius)
  .classed('island', true);

// create the trees - these are static so only draw them once
features.selectAll('.tree')
  .data(trees)
  .enter().append('image')
    .attr('xlink:href', '../../images/tree.svg')
    .attr('width', featureLength)
    .attr('height', featureLength)
    // getX and getY are simple helper functions to return properly offset x
    // and y coords for our square features
    .attr('x', getX)
    .attr('y', getY)
    .classed('tree', true);

svg.select('.island')
  .on('mousemove', function() {
      // 8 is a magic number to make things look nice, on my screen at least.
      var mousePosition = d3.mouse(this);
      grave.x = mousePosition[0] - 8;
      grave.y = mousePosition[1] - 8;
      draw();
  });

function draw() {
  // calculate positions
  var flags = getFlagPositions(trees, grave),
      paths = getPathPoints(trees, flags, grave),
      treasure = getTreasurePosition(flags);

  // draw the treasure location
  features.selectAll('.treasure')
    .data([treasure])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/treasure_chest.svg')
      .classed('treasure', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);


  // draw the grave
  features.selectAll('.grave')
    .data([grave])
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/grave.svg')
      .classed('grave', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);

  
  // draw the flags
  features.selectAll('.flag')
    .data(flags)
      .attr('x', getX)
      .attr('y', getY)
    .enter().append('image')
      .attr('xlink:href', '../../images/flag.svg')
      .classed('flag', true)
      .attr('height', featureLength)
      .attr('width', featureLength)
      .attr('x', getX)
      .attr('y', getY);
  
  // draw triangles showing the flag calculations
  // these are in the geometries group so they appear underneath the other features
  geometries.selectAll('.flag-path')
    .data(paths)
      .attr('d', line)
    .enter().append('path')
      .classed('flag-path', true)
      .attr('d', line);
}
draw();

function getFlagPositions(treePositions, gravePosition) {
  var leftTree, rightTree, rise, run, leftFlag, rightFlag,
      flagPositions, leftTreeIdx, rightTreeIdx;

  // XXX assume grave position is either above or below both, 
  // i.e. y positions are equal for both trees
  // this code determines which tree is "left" and which is "right" based
  // on the location of the grave
  if (
      (treePositions[0].x < treePositions[1].x && treePositions[0].y < grave.y) ||
      (treePositions[0].x > treePositions[1].x && treePositions[0].y > grave.y)
     ) {
    leftTreeIdx = 0;
    rightTreeIdx = 1;
  } else {
    leftTreeIdx = 1;
    rightTreeIdx = 0;
  }

  leftTree = treePositions[leftTreeIdx];
  rightTree = treePositions[rightTreeIdx];

  // get left flag position
  rise = gravePosition.y - leftTree.y;
  run = gravePosition.x - leftTree.x;
  // turn 90 to the left
  leftFlag = {
    x: leftTree.x - rise,
    y: leftTree.y + run
  };
  // get right flag position
  rise = gravePosition.y - rightTree.y;
  run = gravePosition.x - rightTree.x;
  // turn 90 to the right
  rightFlag = {
    x: rightTree.x + rise,
    y: rightTree.y - run
  };
  flagPositions = [];
  // use these indexes to keep the order of our flags consistent with the order
  // of our trees in the trees array
  flagPositions[leftTreeIdx] = leftFlag;
  flagPositions[rightTreeIdx] = rightFlag;
  return flagPositions;
}

function getPathPoints(treePositions, flagPositions, grave) {
  // create two lines, grave -> tree -> flag -> grave for each tree
  return [
    [
      grave,
      treePositions[0],
      flagPositions[0],
      grave
    ],
    [
      grave,
      treePositions[1],
      flagPositions[1],
      grave
    ]
  ]
}

function getTreasurePosition(flagPositions) {
  // averages the x and y values of the two flag positions
  return {
    x: (flagPositions[0].x + flagPositions[1].x) / 2,
    y: (flagPositions[0].y + flagPositions[1].y) / 2
  }
}

function getX(d) {
    return d.x - featureLength/2;
}

function getY(d) {
    return d.y - featureLength/2;
}

})();
</script>


##Conclusion

So that's it. A quick prototype in D3 to brute force our solution so we've got
some intution on how to go about solving the geometry. Great practice in
getting simple things done using d3.js and jogging the old geometry proofs
portion of the brain.

##Attributions

Thanks to the folks over at [The Noun Project](http://thenounproject.com) for
their svg icon sets. Here are the attributions:

* [Tree](http://thenounproject.com/noun/tree/#icon-No17078) James Keunning from The Noun Project
* [Grave](http://thenounproject.com/noun/grave/#icon-No11453) Alex Fuller from The Noun Project
* [Flag](http://thenounproject.com/noun/flag/#icon-No485) The Noun Project
* [Chest](http://thenounproject.com/noun/chest/#icon-No7173) Victor N. Escorsin da silva from The Noun Project
