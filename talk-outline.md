# Module Loading in Node.js
## oder auch: Wie naiv darf eine Programmiersprache sein?

- Schneller Einstieg: Kurzes Code-Beispiel zum Module Loading
    + Angepasstes Beispiel von https://nodejs.org/docs/latest/api/modules.html#modules_modules

```js
// circle.js

const PI = Math.PI;
exports.area = function (radius) {
    return PI * radius * radius;
}
exports.circumference = function (radius) {
    return 2 * PI * radius;
}
```

```js
// main.js
const circle = require("./circle");

console.log("The area of a circle with radius 4 is " + circle.area(4));
```

---

- Kurzzusammenfassung zu `require`:
    + Importiert `.js`, `.json` und `.node` Dateien (auf .node nicht eingehen)
    + Dateiendung muss nicht mit angegeben werden
    + Wenn Dateiname (= beginnt mit `./` oder `/`), dann wird diese Datei importiert (relativ zum aktuellen directory)
    + Wenn kein Dateiname (ohne `./` oder `/`) wird ein node_module importiert
        * Alle node_modules Ordner werden beginnend ab dem aktuellen working directory abwärts nach diesem Modul durchsucht
        * In node_modules/ ist dann ein Ordner, der exakt so heißt wie das Modul

---

- Kurzzusammenfassung `exports`:
    + Ist ein Verweis auf `module.exports`
        * Alternativer Code zum obrigen Beispiel:

        ```js
        // circle.js

        const PI = Math.PI;
        module.exports.area = function (radius) {
            return PI * radius * radius;
        }
        module.exports.circumference = function (radius) {
            return 2 * PI * radius;
        }
        ```

        * `exports` ist einfach kürzer zu schreiben
    + `module.exports` ist ein (default leeres) Objekt
    + `module.exports` wird exportiert, nicht `exports`

---

- exports vs. module.exports
    + Da `exports = module.exports` gilt, funktioniert folgendes nicht:

        ```js
        // circle-area.js
        exports = function (radius) {
            return Math.PI * radius * radius;
        }
        ```

        * In dem Beispiel wird exports neu zugewiesen und referenziert nicht mehr auf `module.exports`

    + Man kann aber auch eine andere Variable an `exports` verweisen.
        * Schönes Pattern, weil es wie eine Nicht-Modul-Datei aussieht
        * "sprechend"
        * wird oft von Node.js Core Modules verwendet

    ```js
    const circle = exports;

    circle.area = function (radius) {
        return PI * radius * radius;
    }
    circle.circumference = function (radius) {
        return 2 * PI * radius;
    }
    ```

    + "As a guideline, if the relationship between exports and module.exports seems like magic to you, ignore exports and only use module.exports". https://nodejs.org/docs/latest/api/modules.html#modules_exports_alias

---

- Wie module loading im Node-Core umgesetzt wird
    + So banal, dass es eigentlich schon frech ist
        * Tatsächlich wollte ich einmal (ohne zu wissen, wie `require` funktioniert), ein module-loading nachbauen, und habe in 2 Stunden ausversehen das Node module loading system grob nachgebaut
    + **Node.js hat als JavaScript-Engine nur die V8 Engine von Google zur Verfügung, die kein modul-system unterstützt**
    + Folglich musste mit den gegebenen Mitteln ein Modul-System gebaut werden
    + `module`, `exports` und `require` sind daher gar nicht global verfügbar, im Gegensatz zu den Globalen `console`, `Math`, ...

---

- THE WAY TO MODULE
    + 1. Module Wrapper:
        * Der module-code wird mit einer Funktion umklammert
        * Die "Globalen" sind nur Funktions-Parameter

        ```js
        (function (exports, require, module, __filename, __dirname) {
        // Your module code actually lives in here
        });
        ```

    + 2. Stringify all the things! \o/
        * Die Datei (die mit `require` importiert wird), wird **als String** eingelesen
        * Das Einlesen **blockiert** das aktuell ausgeführte Skript, bis Datei eingelesen ist
        * Der Module-wrapper wird **als String** der eingelesenen Datei hinzugefügt

    + 3. do a nice eval: vm()
        * `eval()` liest einen String ein und führt ihn als JS-Code aus
        * eval = evil
            * `eval` kann den globalen Scope manipulieren
            * `vm()` führt einen "sicheren eval" aus, der in einer Sandbox ausgeführt wird
        * Der String (bestehend aus module-wrapper und module-code) wird über vm() ausgeführt

    + 4. return module.exports
        * ... das war's.
        * `module` ist im module-wrapper nur ein Parameter, der danach returned werden kann.

        ```js
        const module = {};
        module.exports = {};
        const exports = module.exports

        function(exports, module) {
            // Eigentlicher module-code:
            exports.area = function (radius) { ... }
        }(exports, module) // selbstausführende Funktion

        return module.exports;
        ```

---

- One last lie: require caching
    + Ein modul, was über `require` eingelesen wird, wird im cache gespeichert
    + `require.cache` ist ein einfaches Objekt, was das jeweilige module.exports speichert
    + Key: Dateiname, Value: `module.exports`
    + Beim `require` wird überprüft, ob modul bereits gecached wurde

    ```js
    function myUncachedRequire(file) {
        delete require.cache[file];
        return require(file);
    }
    ```

    + ... wieso?
        * Web Applications müssen wegen diesem Cache jedes Mal neu gestartet werden, wenn eine Datei geändert wird.
        * Dieses kurze Codesnippet verhindert das.
    + KEIN WEB FRAMEWORK FÜR NODE MACHT DAS
        * "Don't mess with globals"
            * Warum nicht, wenn man es nur im Development macht
            * Wir wissen ja jetzt, was wir tun...

---

- Zusammenfassung
    + Um ein bestimmtes Verhalten in einer Programmiersprache zu verstehen, hilft es, sich die tatsächliche Implementierung anzuschauen
    + Am Anfang ist es trotzdem nicht schlimm, das Verhalten als Magie anzusehen
    + Implementation ist so naiv und wird trotzdem als Hauptbestandteil von Node.js genutzt
        => Man darf also auch selbst mal naiv programmieren
