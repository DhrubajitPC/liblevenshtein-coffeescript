# liblevenshtein

## CoffeeScript / JavaScript / Node.js

### A library for generating Finite State Transducers based on Levenshtein Automata.

[![Build Status](https://travis-ci.org/universal-automata/liblevenshtein-coffeescript.svg?branch=master)](https://travis-ci.org/universal-automata/liblevenshtein-coffeescript)

Levenshtein transducers accept a query term and return all terms in a
dictionary that are within n spelling errors away from it. They constitute a
highly-efficient (space _and_ time) class of spelling correctors that work very
well when you do not require context while making suggestions.  Forget about
performing a linear scan over your dictionary to find all terms that are
sufficiently-close to the user's query, using a quadratic implementation of the
[Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) or
[Damerau-Levenshtein
distance](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance),
these babies find _all_ the terms from your dictionary in linear time _on the
length of the query term_ (not on the size of the dictionary, on the length of
the query term).

If you need context, then take the candidates generated by the transducer as a
starting place, and plug them into whatever model you're using for context (such
as by selecting the sequence of terms that have the greatest probability of
appearing together).

For a quick demonstration, please visit the [Github Page,
here](http://universal-automata.github.io/liblevenshtein/).

The library is currently written in Java, CoffeeScript, and JavaScript, but I
will be porting it to other languages, soon.  If you have a specific language
you would like to see it in, or package-management system you would like it
deployed to, let me know.

### Basic Usage:

#### Node.js

Install the module via `npm`:

```
% npm install liblevenshtein
info trying registry request attempt 1 at 12:59:16
http GET https://registry.npmjs.org/liblevenshtein
http 304 https://registry.npmjs.org/liblevenshtein
liblevenshtein@2.0.4 node_modules/liblevenshtein
```

Then, you may `require` it to do whatever you need:

```javascript
var levenshtein = require('liblevenshtein');

// Assume "completion_list" is a list of terms you want to match against in
// fuzzy queries.
var builder = new levenshtein.Builder()
  .dictionary(completion_list, false)  // generate spelling candidates from unsorted completion_list
  .algorithm("transposition")          // use Levenshtein distance extended with transposition
  .sort_candidates(true)               // sort the spelling candidates before returning them
  .case_insensitive_sort(true)         // ignore character-casing while sorting terms
  .include_distance(false)             // just return the ordered terms (drop the distances)
  .maximum_candidates(10);             // only want the top-10 candidates

// Maximum number of spelling errors we will allow the spelling candidates to
// have, with regard to the query term.
var MAX_EDIT_DISTANCE = 2;

var transducer = builder.build();

// Assume "term" corresponds to some query term. Once invoking
// transducer.transduce(term, MAX_EDIT_DISTANCE), candidates will contain a list
// of all spelling candidates from the completion list that are within
// MAX_EDIT_DISTANCE units of error from the query term.
var candidates = transducer.transduce(term, MAX_EDIT_DISTANCE);
```

#### In the Browser

To use the library on your website, reference the desired file from the
`<head/>` of your document, like so:

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- stuff ... -->
    <script type="text/javascript"
      src="http://universal-automata.github.com/liblevenshtein/javascripts/2.0.4/levenshtein-transducer.min.js">
    </script>
    <!-- more stuff ... -->
  </head>
  <body>
    <!-- yet another fancy document ... -->
  </body>
</html>
```

Once the script loads, you should construct a transducer via the [Builder
Api](http://universal-automata.github.io/liblevenshtein/docs/coffeescript/builder.html):

```javascript
$(function ($) {
  "use strict";

  // Maximum number of spelling errors we will allow the spelling candidates to
  // have, with regard to the query term.
  var MAX_EDIT_DISTANCE = 2;

  var completion_list = getCompletionList(); // fictitious method

  var builder = new levenshtein.Builder()
    .dictionary(completion_list, false)  // generate spelling candidates from unsorted completion_list
    .algorithm("transposition")          // use Levenshtein distance extended with transposition
    .sort_candidates(true)               // sort the spelling candidates before returning them
    .case_insensitive_sort(true)         // ignore character-casing while sorting terms
    .include_distance(false)             // just return the ordered terms (drop the distances)
    .maximum_candidates(10);             // only want the top-10 candidates

  var transducer = builder.build();

  var $queryTerm = $('#query-term-input-field');
  $queryTerm.keyup(function (event) {
    var candidates, term = $.trim($queryTerm.val());

    if (term) {
      candidates = transducer.transduce(term, MAX_EDIT_DISTANCE);
      printAutoComplete(candidates); // print the list of completions
    } else {
      clearAutoComplete(); // user has cleared the search box
    }

    return true;
  });
});
```

This will give the user autocompletion hints as he types in the search box.

### Reference

This library is based largely on the work of [Stoyan
Mihov](http://www.lml.bas.bg/~stoyan/), [Klaus
Schulz](http://www.cis.uni-muenchen.de/people/schulz.html), and Petar Nikolaev Mitankin: "[Fast
String Correction with
Levenshtein-Automata](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.16.652
"Klaus Schulz and Stoyan Mihov (2002)")".  For more details, please see the
[wiki](https://github.com/universal-automata/liblevenshtein/wiki).
