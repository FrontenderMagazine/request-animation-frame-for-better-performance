Request Animation Frame for Better Performance
==============================================

![cover][Cover Image]

Traditionally, JavaScript animation can be handled a number of ways. One
approach involves using a timing function such as setTimeout or setInterval to
adjust styling every couple milliseconds. Another approach creates a loop that
changes styling as many times as possible along the animation’s time frame. The
logic of both of these approaches is to throw a large number of animation frames
at the browser, so that it hopefully renders smooth-looking motion.

However, in practice, these approaches leave a lot to be desired. The renderer
tends to choke on the large number of rendering tasks, and often isn’t able to
render a certain frame before it has already received instructions to render the
next. So, even though the browser renders as many frames of the animation as
possible, the dropped frames still make for a choppy-looking animation, which is
not to mention the performance implications of overloading the processor with as
many tasks as possible.

In reality, it’s actually better to render fewer frames, provided they are
rendered consistently. That’s because our eyes pick up these subtle variations,
and a few dropped frames tend to stick out more than an overall lower frame
rate. That’s where HTML5′s requestAnimationFrame API comes in.

## Advantages of requestAnimationFrame

requestAnimationFrame gives the browser control over how many frames it renders.
Instead of demanding the browser render frames it will ultimately have to drop,
you allow it to spit out frames whenever it’s ready, and above all consistently.
The benefit is two-fold:

* The animation looks smoother since it uses a consistent frame rate. * Rather
than being overloaded with rendering tasks, the processor is able to handle
other tasks while also rendering the animation. In fact, it is able to determine
a frame rate that works with the other tasks it is handling.

Another advantage to using requestAnimationFrame is that it works with the
screen’s refresh rate, which defines how quickly the monitor can display a new
frame. You see, even if your user has a suped-up processor that can render a ton
of frames, it won’t matter if those frames don’t make it onto the screen. And if
your animation loop falls out of sync with the screen refresh rate, it will mean
a ton of inconsistently dropped frames, and a decidedly choppy-looking
animation.

Finally, if the browser’s current tab loses focus, requestAnimationFrame will
stop animating. This has important power-saving and performance implications. So
go green, and use requestAnimationFrame!

## How to Use requestAnimationFrame

Now that you understand the advantages of requestAnimationFrame, I’d like to
dive in and show you how to use it. I’m going to use an example from my new book
[JavaScript Programming: Pushing the Limits][1], which you should pick up if you
want to learn more about requestAnimationFrame and other advanced HTML5
techniques.

First create some markup to work with:

    <div id="my-element">Click to start</div>

Along with some basic styling:

    #my-element {
      position: absolute;
      left: 0;
      width: 200px;
      height: 200px;
      padding: 1em;
      background: tomato;
      color: #FFF;
      font-size: 2em;
      text-align: center;
    }

And finally the script:

    var elem = document.getElementById('my-element'),
        startTime = null,
        endPos = 500, // in pixels
        duration = 2000; // in milliseconds

    function render(time) {
      if (time === undefined) {
        time = new Date().getTime();
      }
      if (startTime === null) {
        startTime = time;
      }

      elem.style.left = ((time - startTime) / duration * endPos % endPos) + 'px';
    }

    elem.onclick = function() {
      (function animationLoop(){
        render();
        requestAnimationFrame(animationLoop, elem);
      })();
    };

The script adds a click handler to the element, which creates an animation that
moves it 500 pixels to the right over the course of two seconds, and then loops
around to the beginning.

The first part of the script sets up a render() function, which compares the
current time against the start time to determine the right frame in the
animation to render (similar to how this might be handled with a traditional
animation script).

Next, animationLoop() renders the current frame of the animation, and then calls
requestAnimationFrame. As you can see, animationLoop() first passes itself into
requestAnimationFrame, along with the element you want to animate. This
establishes an animation loop, which renders new frames whenever the browser is
ready to do so.

Run this script in your browser. You’ll see how smooth the animation looks, even
if you crank up the speed substantially.

## Polyfill for Backwards Compatibility

At this point, I’m sure you agree that requestAnimationFrame is a great solution
to a number of animation problems. However, the reality is that a significant
number of users are still on non-HTML5 browsers that do not support this API.
Fortunately, there is a polyfill released by Erik Möller and Paul Irish which
provides support for older browsers. No, the older browsers won’t get any of the
advantages of the new API, but they will still see animation through a
traditional timing function approach.

Simply include the following code, which you can read about [here][2]:

    (function() {
      var lastTime = 0;
      var vendors = ['ms', 'moz', 'webkit', 'o'];
      for(var x = 0; x &lt; vendors.length &amp;&amp; !window.requestAnimationFrame; ++x) {
        window.requestAnimationFrame = window[vendors[x]+'RequestAnimationFrame'];
        window.cancelAnimationFrame = window[vendors[x]+'CancelAnimationFrame']
                                   || window[vendors[x]+'CancelRequestAnimationFrame'];
      }

      if (!window.requestAnimationFrame)
        window.requestAnimationFrame = function(callback, element) {
          var currTime = new Date().getTime();
          var timeToCall = Math.max(0, 16 - (currTime - lastTime));
          var id = window.setTimeout(function() { callback(currTime + timeToCall); },
            timeToCall);
          lastTime = currTime + timeToCall;
          return id;
        };

      if (!window.cancelAnimationFrame)
        window.cancelAnimationFrame = function(id) {
          clearTimeout(id);
        };
    }());

## Conclusion

I hope you enjoyed learning about requestAnimationFrame. It’s one of many HTML5
advancements you can discover on HTML5 Hub. Try out the example, and let me know
how you like it, either in the comments below, or by messaging me on Twitter
[@jonraasch][3]. And make sure to check out my book [JavaScript Programming:
Pushing the Limits][4], which covers a wide variety of advanced HTML5 and
JavaScript techniques.


[cover]: http://html5hub.com/wp-content/uploads/2013/07/requestAnimationFrame.png

[1]: http://bit.ly/jspptl
[2]: https://gist.github.com/paulirish/1579671
[3]: http://twitter.com/jonraasch
[4]: http://bit.ly/jspptl
