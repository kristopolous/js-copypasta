# Sometimes you just want one isolated function.

Sometimes I find myself writing the same helper functions in situations where I don't want to pull in a library, just put in some really small function.

This is a *different* kind of toolkit ... there's no source to include.

Instead, there's a description of each function, what it does, and then a multi and one line version of it for you to put into your code.

> Note: The one-line version is intentionally *not* "minified" or obfuscated ... it's intended to be a readable variation of the multi-line that fits in one line.

Let's get started

# when

## Purpose 
Similar to an [AMD Require](http://requirejs.org/docs/whyamd.html) except designed to delay execution until a library that is included through say an HTML `<script>` tag is loaded.

## Example Usage

This will wait until JQuery is loaded ( presuming it's being included somewhere else on the page ):

Sometimes, given a not-so-hotly designed system, this can lead to a race-condition.  

    <script>
    $(function() { ... });
    </script>

You could either be an "architect" and spend endless hours trying to fix it, or use when:

    <script>
    when('$').run(function() { ... });
    </script>

## Implementation

### Multi-line

    function when(lib) {
      var _cb, _ival = setInterval(function(){

        if(self[lib]) {
          _cb();
          clearInterval(_ival);
        }
      }, 20);

      return {
        run: function(cb) { _cb = cb; }
      }
    }

### Single-line

    function when(lib){ var _cb, _ival=setInterval(function(){ if(self[lib]) { _cb(); clearInterval(_ival); } }, 20); return{ run: function(cb) { _cb=cb; } } }

 
function multi() {
  var _list = [], _invoke = function() {
    for(var ix = 0; ix < _list.
  }
  _invoke.add = function() {
    _list = _list.concat(arguments);
  }
  return _invoke;
}

# multi

## Purpose
Some libraries don't allow an easy way to specify an `Array` of callbacks as opposed to just one.  `multi` allows any number of functions to be glued together and executed in sequence

## Example Usage
Pretend you have some old crufty code that wants you to specify things like this:

    flashObject.onclick = fn0;

But you also want it to run fn1, fn2, and fn3.

With multi you can do this like so:

    var fn_wrap = multi();

    fn_wrap.add(fn0).add(fn1).add(fn2).add(fn3);

    flashObject.onclick = fn_wrap;

The `arguments` and `this` pointer are of course maintained.

## Implementation

### Multi-line

    function multi() {
      var _list = [], _invoke = function() {
        var _args = arguments, _this = this;

        _list.forEach(function(cb) {
          cb.apply(this, _args);
        }, _this);
      }

      _invoke.add = function(cb) {
        _list.push(cb);
        return _invoke;
      }

      return _invoke;
    }

### Single-line

    function multi(){var list=[], invoke=function(){var _args=arguments, _this=this; _list.forEach( function(cb){cb.apply(this, _args);}, _this );}  _invoke.add=function(cb){_list.push(cb); return _invoke;}  return _invoke;}

