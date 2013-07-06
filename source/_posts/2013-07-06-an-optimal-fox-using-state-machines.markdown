---
layout: post
title: "An Optimal Fox Using State Machines"
date: 2013-07-05 16:34
comments: true
published: true
categories: ["d3.js", "puzzles"]
---

<style>
    .fox-hole.selected {
        fill: white;
        stroke: black;
    }
    #fox {
        width: 600px;
        overflow: auto;
        min-height: 300px;
    }
</style>


I've really enjoyed thinking about the puzzles at
[http://gurmeet.net/puzzles/](http://gurmeet.net/puzzles/), especially
looking for ways to solve them or get hints as to how to solve them using
visualization without knowing the answers.
[Last Time](http://azundo.github.io/blog/using-d3-dot-js-to-brute-force-the-pirate-puzzle/)
I explored puzzle 11, Treasure Island, and this time I'll look at puzzle 5, Fox
in a Hole.

##The Puzzle
Here is the wording from [http://gurmeet.net/puzzles/](http://gurmeet.net/puzzles/):

    Consider five holes in a line. One of them is occupied by a fox. Each night,
    the fox moves to a neighboring hole, either to the left or to the right. Each
    morning, you get to inspect a hole of your choice. What strategy would ensure
    that the fox is eventually caught?

##The Challenge
The question I asked was, can we, without knowledge of a working strategy,
create a fox that plays optimally against us so we can be sure we've come up
with a strategy that will work? We could simulate a fox that chooses to move
left or right randomly, but we would never be sure that our strategy is fool
proof. Can we make sure our fox plays hard to get if we don't even know how
to catch it? Turns out we can. Try out the demo of my solution and see if you
can both figure out how to catch the fox and understand why a strategy works.
Implementation details are below the demo. Code is at 
[https://gist.github.com/azundo/5941503](https://gist.github.com/azundo/5941503).


###Instructions

    Click on a hole to select it for inspection. You win if the fox has nowhere
    left to hide. If you give up, click the button and one possible path the fox
    could have taken to avoid your inspections will be shown.


<div id="fox"></div>


<script src="http://d3js.org/d3.v3.min.js" charset="utf-8"></script>
<script type="text/javascript">
(function(){
  var
    foxHoleR = 45,
    xOffset = foxHoleR*2 + 15,
    yOffset = foxHoleR/4 + 40,
    container = d3.select('#fox'),
    svg,
    currentDay,
    button,
    selector,

  /*
   * State machine functions
   *
   * In this example, a state is the set of possible holes that the fox could
   * be in on a given day after accounting for the farmer's strategy up to that
   * point.
   *
   * For example on the first day, after the farmer inspects a hole, it is
   * possible that the fox is in any of the other holes, so the state for the
   * first day would have four entries.
   *
   * We could represent our state space as a 5-bit bit vector, where a bit is
   * on if the fox could be in that hole, and off if the fox couldn't be in
   * that hole. I think this representation (an array of possible values) is a
   * bit easier to follow and less accessible bit manipulation.
   */

  ADJACENCY_LIST = [
    // hole 0 is next to hole 1
    [1],
    // hole 1 is next to hole 0 and hole 2
    [0, 2],
    // etc.
    [1, 3],
    [2, 4],
    [3]
    ],
  state,
  stateHistory;


  function nextState(pastState, guess) {
    /*
     * Calculates our next state based on the past state and the next hole to be inspected.
     */
    var next = [], i, j, possiblePastHole, possibleNextHoles, possibleNextHole;
    for (i = 0; i < pastState.length; i++) {
      // any value in our pastState represents a hole that the fox was possibly in yesterday
      possiblePastHole = pastState[i];

      // look up holes next to the possiblePastHole in our ADJACENCY_LIST
      possibleNextHoles = ADJACENCY_LIST[possiblePastHole];
      for (j = 0; j < possibleNextHoles.length; j++) {
        // any holes that are ajacent to a possible hole from the pastState
        // are valid holes for the fox in the next state, provided that hole wasn't inspected.
        // Note that the fox cannot stay in the same hole, it has to move to an adjacent hole.
        possibleNextHole = possibleNextHoles[j]
        // don't add holes that are already in the next state array since our
        // state is the set of possible holes
        if (possibleNextHole !== guess && next.indexOf(possibleNextHole) === -1) {
          next.push(possibleNextHole);
        }
      }
    }
    return next;
  }

  function getPossibleSequence() {
    // only depends on our stateHistory
    var i, finalState, stateToDetermine, possibleHoles, possibleHole, seq = [];
    if (stateHistory.length === 0) {
      return seq;
    }
    finalState = stateHistory.pop();
    // pick whatever hole happens to be listed first in the final state to end our sequence.
    seq.unshift(finalState[0]);
    // work backwards through the state history, picking a valid state from each 
    // entry and unshifting it onto the front of the sequence
    while (stateHistory.length > 0) {
      stateToDetermine = stateHistory.pop();
      // find a hole beside the hole at the front of our sequence
      // since our 'graph' is not directed, we can safely use
      // the same adjacency values to move backwards through
      // the state list
      possibleHoles = ADJACENCY_LIST[seq[0]];
      for (i = 0; i < possibleHoles.length; i++) {
        possibleHole = possibleHoles[i];
        // if the possible hole is in our stateToDetermine, use it.
        if (stateToDetermine.indexOf(possibleHole) !== -1){
          seq.unshift(possibleHole);
          break;
        }
      }
    }
    return seq;
  }

  /*
   * End of State Machine Functions
   */

  function selectHole() {
      // pull the hole number from the id. XXX should fix this
      var selection = parseInt(d3.select(this).attr('id').split('-').pop());
      // update state information
      state = nextState(state, selection);
      // if there are no possible holes for the fox to be in, we caught him!
      if (state.length === 0) {
        foxCaught(selection);
      } else {
        advanceOneDay(selection);
      }
  }

  function foxCaught(selection) {
    alert("You got him!");
    // remove the selector
    selector.remove();

    // our last state was where we caught the fox
    stateHistory.push([selection]);

    // draw the current selection
    drawHoles(currentDay, currentDay, selection);

    // draw all foxes in
    drawFoxes();

    // reset the button
    button
      .text('Try Again')
      .on('click', init);
  }

  function advanceOneDay(selection) {
    // update our stateHistory
    stateHistory.push(state.slice());

    // increase size of container div and/or svg canvas
    updateCanvasSizes();

    // move the selector down
    moveSelector();

    // draw in row with our selection
    drawHoles(currentDay, currentDay, selection);

    // fill in 'missed' label
    labelMissed(currentDay, selection);

    // increase our day counter
    currentDay += 1;

  }

  init();

  function init() {
    state = [0, 1, 2, 3, 4];
    stateHistory = [];
    container.html('');
    svg = container.append('svg')
      .attr('width', 600)
      .attr('height', 2*yOffset);
    currentDay = 0;
    button = container.append('button')
      .text('I give up. Show me the fox!')
      .on('click', giveUp);
    selector = drawHoles('selector', 0);
    selector.selectAll('.fox-hole')
      .on('mouseover', function() {
        d3.select(this).classed('selected', true);
      })
      .on('mouseout', function() {
        d3.select(this).classed('selected', false);
      })
      .on('click', selectHole);
  };


  function giveUp() {
    selector.remove();
    // case where the user didn't make any guesses, put the fox in the first hole
    if (stateHistory.length === 0) {
      drawHoles(currentDay, currentDay);
      drawFox(currentDay, 0);
    }
    else {
      drawFoxes();
    }
    button
      .text('Try Again')
      .on('click', init);
  }

  /*
   * D3 Drawing functions
   */

  function drawHoles(idSuffix, row, selected) {
    var i, hole, holes = svg.append('g')
    .attr('id', 'fox-holes-' + idSuffix)
      .attr('transform', 'translate(0,' + ((row+1) * yOffset) + ')');
    holes.append('text')
      .text('Day ' + (row + 1))
      .attr('y', yOffset / 4);
    for (i = 0; i < 5; i++) {
      hole = holes.append('ellipse')
        .attr('id', 'fox-hole-' + idSuffix + '-' + i)
        .attr('rx', foxHoleR)
        .attr('ry', foxHoleR / 4)
        .attr('cx', i * xOffset + xOffset)
        .attr('cy', foxHoleR / 4 )
        .classed('fox-hole', true);
      if (i === selected) {
        hole.classed('selected', true)
      }
    }
    return holes;
  }

  function drawFoxes() {
    var seq = getPossibleSequence(), i;
    for (i=0; i < seq.length; i++) {
      drawFox(i, seq[i]);
    }
  }

  function drawFox(row, hole) {
    d3.select("#fox-holes-" + row)
      .append('image')
      .attr('xlink:href', '../../images/fox_white.svg')
      .attr('width', foxHoleR)
      .attr('height', foxHoleR)
      .attr('x', (hole+1) * xOffset - foxHoleR / 2 )
      .attr('y', -1 * foxHoleR / 2 );
  }

  function updateCanvasSizes() {
    // increase size of svg canvas if need be
    if (svg.attr('height') < (currentDay + 3) * yOffset) {
      svg.attr('height', (currentDay + 3) * yOffset);
    }
    // add to container's height if it's too small
    // to facilitate scrolling
    var containerHeight = parseInt(container.style('min-height'));
    if (containerHeight < (currentDay + 3) * yOffset) {
      container.style('min-height', (containerHeight + yOffset*5) + 'px');
    }
  }

  function moveSelector() {
    selector
      .transition()
        .attr('transform', 'translate(0,' + (currentDay+2) * yOffset + ')');
    selector.select('text')
      .text('Day ' + (currentDay+2));
    // mouseout doesn't fire when using transform to move the selector until the mouse
    // is moved again, so manually de-select the hole
    selector.select('.fox-hole.selected')
      .classed('selected', false);
  }

  function labelMissed(row, selection) {
    d3.select("#fox-holes-" + row)
      .append('text')
      .text('Missed!')
      .attr('x', (selection+1) * xOffset - foxHoleR / 2 )
      .attr('y', foxHoleR / 3);
  }

})();
</script>

##A Quantum Fox
Instead of modeling our fox as being in one concrete location, we'll keep track
of all possible holes the fox could be in at any given time without getting
caught, depending on which holes we have inspected in the past. We'll use a
[finite-state machine](https://en.wikipedia.org/wiki/Finite-state_machine) to
keep track of which holes the fox could be in. If we ever get to the state
where the fox couldn't be in any holes without being caught, we know our
strategy is a good one. We don't require any knowledge of the best path for
the fox to choose, or a winning strategy.

###Our State Space
Each state in our state space will consist of all of the possible holes the fox
could be in at any one time. There are 5 holes and each hole has two states: it
is either possible for the fox to be in that hole, or not possible. This gives
us a state space of size `2^5 = 32`. We could model this as a bit vector with 5
bits, but for clarity I will use a simple array. If it is possible for the fox
to be in hole `i`, then our state array will contain `i`. We'll number our
holes `0`, `1`, `2`, `3`, `4`.

For example, before we check any holes, our state would be `[0, 1, 2, 3, 4]`.
The fox could be in any hole. If we check hole `2`, then our state would move
to `[0, 1, 3, 4]`. In the case where there is no possible hole for our fox to
be in without catching him at some point, our state is an empty array `[]`.

###State Transitions
The key implementation detail when using a state machine is our state
transitions. How do we decide which state we move to next? In our case our next
state is dependent on two things, our previous state (the set of possible holes
the fox could have been in yesterday) and our selection of which hole to check.

To start let's forget about the action of checking a hole. How can we determine
what holes the fox can be in based on what holes it could have been in
yesterday? If it was possible for the fox to be in hole `i` yesterday, then it
is possible for the fox to be in any hole beside `i` today. If we iterate over
all of the holes that the fox could have been in yesterday then we'll get the
set of holes it could be in today.

We'll use a very simple adjacency list to make our lives easy. If we access
element `i` of the adjacency list we will get back an array containing all of
the holes next to `i`.



```javascript
var ADJACENCY_LIST = [
    // hole 0 is next to hole 1
    [1],
    // hole 1 is next to hole 0 and hole 2
    [0, 2],
    // etc.
    [1, 3],
    [2, 4],
    [3]
];
```

This [adjacency list](http://en.wikipedia.org/wiki/Adjacency_list) defines a
very simple [graph](https://en.wikipedia.org/wiki/Graph_(mathematics)) where
each hole is a node connected by edges to each of its adjacent holes. The first
and last holes only have one adjacent hole (and thus one edge) while the others
have two.

Accessing element `0` of our `ADJACENCY_LIST` tells us which holes are next
to hole `0`. If we want to know the set of possible future holes, we take the
set of possible past holes and add all of the adjacent holes. Simple enough.

Lastly we don't include the hole that we chose to inspect since the fox would
get caught if he ended up there. Hence, our transition function looks like this:

```javascript
  function nextState(pastState, guess) {
    /*
     * Calculates our next state based on the past state and
     * the next hole to be inspected.
     */
    var next = [], i, j, possiblePastHole, possibleNextHoles, possibleNextHole;
    for (i = 0; i < pastState.length; i++) {
      // any value in our pastState represents a hole that
      // the fox was possibly in yesterday
      possiblePastHole = pastState[i];

      // look up holes next to the possiblePastHole in our ADJACENCY_LIST
      possibleNextHoles = ADJACENCY_LIST[possiblePastHole];
      for (j = 0; j < possibleNextHoles.length; j++) {
        // any holes that are ajacent to a possible hole
        // from the pastState are valid holes for the fox in
        // the next state, provided that hole wasn't
        // inspected.  Note that the fox cannot stay in the
        // same hole, it has to move to an adjacent hole.
        possibleNextHole = possibleNextHoles[j]
        // don't add holes that are already in the next
        // state array since our state is the set of
        // possible holes
        if (possibleNextHole !== guess && next.indexOf(possibleNextHole) === -1) {
          next.push(possibleNextHole);
        }
      }
    }
    return next;
  }
```

If we ever end up with an empty state, the fox has no where to go. We must have
caught it at some point, no matter where it started and how it decided to move.

###Reconstructing the Fox's Moves

If we save each state we reach, we can reconstruct a possible path the fox
could have taken to get to the current state. Let's choose any arbitrary hole
from the current state, and work backwards by looking for adjacent holes in the
previous state. We are guaranteed to find a backwards path since every hole in
our current state must be possible. Here is the code:

```javascript

  function getPossibleSequence() {
    // only depends on stateHistory, an array of our past states
    var i, finalState, stateToDetermine, possibleHoles, possibleHole, seq = [];
    if (stateHistory.length === 0) {
      return seq;
    }
    finalState = stateHistory.pop();
    // pick whatever hole happens to be listed first in the
    // final state to end our sequence.
    seq.unshift(finalState[0]);
    // work backwards through the state history, picking a
    // valid state from each entry and unshifting it onto
    // the front of the sequence
    while (stateHistory.length > 0) {
      stateToDetermine = stateHistory.pop();
      // find a hole beside the hole at the front of our
      // sequence since our 'graph' is not directed, we can
      // safely use the same adjacency values to move
      // backwards through the state list
      possibleHoles = ADJACENCY_LIST[seq[0]];
      for (i = 0; i < possibleHoles.length; i++) {
        possibleHole = possibleHoles[i];
        // if the possible hole is in our stateToDetermine, use it.
        if (stateToDetermine.indexOf(possibleHole) !== -1){
          seq.unshift(possibleHole);
          break;
        }
      }
    }
    return seq;
  }
```

##Demo and Further Steps

Throwing this together with some d3.js drawing and we get the demo from above.
The full code is in a gist at
[https://gist.github.com/azundo/5941503](https://gist.github.com/azundo/5941503)
and viewable on [bl.ocks.org](http://bl.ocks.org) at
[http://bl.ocks.org/azundo/5941503](http://bl.ocks.org/azundo/5941503).

You could also make solving the puzzle a bit easier if you showed all possible
holes the fox could be in as you go along, instead of only showing one possible
path at the end. Feel free to fork and implement this as a good exercise in d3
basics.

Now can you catch the fox?

##Attributions
* <a href="http://thenounproject.com/noun/fox/#icon-No4144" target="_blank">Fox</a> designed by <a href="http://thenounproject.com/ceqq" target="_blank">Sebastian Blei</a> from The Noun Project

