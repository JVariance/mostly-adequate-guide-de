# Kapitel 4: Currying

## Ich kann nicht leben, wenn das Leben ohne dich ist
Mein Vater erklärte mir einst, wie es gewisse Dinge gäbe, ohne die man leben könne, bis man sie erwirbt. Eine Mikrowelle ist eine solche Sache. Smartphones eine andere. Die Älteren unter uns werden sich an ein erfüllendes Leben ohne Internet erinnern. Für mich zählt *Currying* zu dieser Liste.

Das Konzept ist simpel: Du kannst eine Funktion mit weniger Argumenten aufrufen, als sie erwartet. Sie gibt wiederum eine Funktion zurück, die die restlichen Argumente aufnimmt.

Du kannst wählen, ob du die Funktion mit allen Argumenten auf einmal fütterst oder stückweise.

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

Hier haben wir eine Funktion `add`, die ein Argument erwartet und eine Funktion zurückgibt. Indem du sie aufrufst, merkt sich die zurückgegebene Funktion das erste Argument von nun an über den Funktionsabschluss. Der Aufruf mit beiden Argumenten auf einmal ist jedoch ein wenig krampfhaft, sodass wir eine spezielle Hilfsfunktion namens `curry` verwenden können, um das Bestimmen und Aufrufen von Funktionen zu vereinfachen.

Lass' uns ein paar "curried" Funktionen zu unserem Vergnügen aufsetzen.

```js
var curry = require('lodash.curry');

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});
```
Das von mir verfolgte Modell ist ein simples, aber wichtiges. Ich habe die zu verarbeitenden Daten (String, Array) strategisch als letztes Argument gesetzt. Im Verlaufe der Nutzung wird klar, warum.


```js
match(/\s+/g, "hello world");
// [ ' ' ]

match(/\s+/g)("hello world");
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces("hello world");
// [ ' ' ]

hasSpaces("spaceless");
// null

filter(hasSpaces, ["tori_spelling", "tori amos"]);
// ["tori amos"]

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(["tori_spelling", "tori amos"]);
// ["tori amos"]

var noVowels = replace(/[aeiou]/ig);
// function(replacement, x) { return x.replace(/[aeiou]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiou]/ig, "*") }

censored("Chocolate Rain");
// 'Ch*c*l*t* R**n'
```

Was hier demonstriert wird, ist die Fähigkeit des "Vorladens" (englisch: pre-load) einer Funktion mit einem oder zwei Argumenten, um eine neue Funktion zu erhalten, die sich diese Argumente merkt.

Ich ermutige Dich dazu _lodash_ zu installieren (`npm install lodash`), den Code oben zu kopieren und ihn im REPL auszuprobieren. Du kannst dies ebenfalls in einem Browser tun, in dem _lodash_ oder _ramda_ verfügbar ist.

## Mehr als ein Wortspiel / spezielle Sauce

__Currying__ ist für vieles nützlich. Wir können über das bloße Übergeben einiger Argumente an unsere Basisfunktionen neue Funktionen kreieren, wie wir in `hasSpaces`, `findSpaces` und `censored` sehen können.

Wir haben ebenso die Möglichkeit, eine jegliche Funktion, die mit einzelnen Elementen arbeitet über ein simples Einwickeln in `map` in eine Funktion umzuwandeln, die mit Arrays arbeitet.

```js
var getChildren = function(x) {
  return x.childNodes;
};

var allTheChildren = map(getChildren);
```

Einer Funktion weniger Argumente zu übergeben, als sie erwartet wird üblicherweise *partial application* (zu deutsch: partielle/teilweise Verwendung) genannt. Das partielle Anwenden einer Funktion kann eine Menge _Boilerplate-Code_ einsparen. Überlege, wie die obige `allTheChildren`-Funktion mit _lodashs_  _uncurried_ `map` aussähe (bemerke, dass die Argumente in einer anderen Reihenfolge sind).

```js
var allTheChildren = function(elements) {
  return _.map(elements, getChildren);
};
```

Wir definieren arraybasierte Funktionen üblicherweise nicht, da wir `map(getChildren)` ganz einfach inline (einzeilig) aufrufen können. Selbiges gilt für `sort`, `filter` und andere _Funktionen höherer Ordnung_ (Funktion höherer Ordnung: Eine Funktion, die eine Funktion erwartet oder zurückgibt).

Als wir über _pure Funktionen_ sprachen, sagten wir, dass sie *eine* Eingabe in *eine* Ausgabe umwandeln. _Currying_ tut genau das: jedes einzelne Argument gibt eine neue, die verbliebenen Argumente erwartende Funktion zurück. Das, alter Knabe, ist *eine* Eingabe in *eine* Ausgabe.

Ganz egal, ob die Ausgabe eine weitere Funktion ist - sie gilt as pur. Wir erlauben zwar mehr als ein Argument zur Zeit, aber dies gilt as bloßes Entfernen des extra `()`'s aus Bequemlichkeit.


## Alles in allem

_Currying_ ist nützlich und ich genieße es sehr tagtäglich mit _curried_ Funktionen zu arbeiten. Es ist ein Werkzeug für den Gürtel, der funktionale Programmierung weniger langatmig und mühsam macht.

Wir können neue, nützliche Funktionen durch simples Übergeben weniger Argumente auf die Schnelle kreieren und bewahren als Bonus die mathematische Funktionsdefinition trotz mehrerer Argumente.

Lass' uns ein weiteres essentielles Werkzeug namens `compose` beschaffen.

[Kapitel 5: Programmierung über Komposition](ch5.md)

## Übungen

Ein schnelles Wort bevor wir beginnen. Wir werden eine Bibliothek namens *ramda* benutzen, welche standardmäßig jede Funktion "curried". Alternativ könntest du *lodash-fp* wählen, welches selbiges tut und von _lodashs_ Kreierer geschrieben und verwaltet wird. Beide werden prima laufen, es ist lediglich eine Sache der Präferenz.

[ramda](http://ramdajs.com)
[lodash-fp](https://github.com/lodash/lodash-fp)

Es gibt [Modultests](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises), die gegen deine Übungen laufen, während du sie programmierst oder du copy-pastest sie in ein JavaScript-REPL für die ersten Übungen, wenn du so wünschst.

Lösungen mit Code werden im [Repository für dieses Buch](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers) angeboten.

```js
var _ = require('ramda');


// Übung  1
//==============
// Schreibe den Code so um, dass alle Argumente entfernt werden, indem du die Funktion partiell anwendest

var words = function(str) {
  return _.split(' ', str);
};

// Übung 1a
//==============
// Nutze map,um eine neue _words_ Funktion zu erstellen, die mit einem Array aus Strings funktioniert

var sentences = undefined;


// Übung 2
//==============
// Schreibe den Code so um, 
// Schreibe den Code so um, dass alle Argumente entfernt werden, indem du die Funktion partiell anwendest

var filterQs = function(xs) {
  return _.filter(function(x){ return match(/q/i, x);  }, xs);
};


// Übung 3
//==============
// Nutze die Hilfsfunktion _keepHighest, um _max_ so umzuschreiben, dass es keine Argumente bezieht

// LEAVE BE:
var _keepHighest = function(x,y){ return x >= y ? x : y; };

// REFACTOR THIS ONE:
var max = function(xs) {
  return _.reduce(function(acc, x){
    return _keepHighest(acc, x);
  }, -Infinity, xs);
};


// Bonus 1:
// ============
// umschließe array's slice so, dass es funktional und curried ist
// //[1,2,3].slice(0, 2)
var slice = undefined;


// Bonus 2:
// ============
// nutze slice, um eine Funktion "take" zu definieren, die _n_ Elemente vom Anfang eines Strings aufnimmt. Mache sie _curried_.
// // Ergebnis für "Bauernhof" mit n=4 sollte "Baue" sein.
var take = undefined;
```
