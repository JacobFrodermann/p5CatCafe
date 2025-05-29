## Analyse der Baumstruktur (`Tree<T>`)

### Struktur & Design

Der verwendete Binärbaum ist durch ein gemeinsames Interface `Tree<T>` mit zwei konkreten Implementierungen modelliert:

- `Empty<T>`: repräsentiert ein leeres Blatt
- `Node<T>`: enthält ein Datenelement sowie Verweise auf linken und rechten Teilbaum

Diese rekursive Modellierung ermöglicht eine funktionale, unveränderliche Baumstruktur, in der jede Modifikation (`addData`) eine neue Bauminstanz erzeugt. Dadurch sind parallele Zugriffe sicher.

---

### Vorteile

- **Typensicherheit:** `Empty` statt `null` vermeidet `NullPointerExceptions`.
- **Immutable Design:** Änderungen erzeugen neue Objekte → thread-sicher.
- **Erweiterbarkeit:** Durch das Visitor-Pattern (`TreeVisitor`) können Traversierungsstrategien flexibel ergänzt werden.
- **Gute Stream-Integration:** Dank `Iterable<T>`, `spliterator()` und `stream()` lassen sich moderne Java-Streams problemlos nutzen.

---

### Nachteile

- **Leistung:** Keine Balancierung → Worst-Case-Leistung beim Suchen und Einfügen ist linear (`O(n)`).
- **Speicherverbrauch:** Viele Objekte (insbesondere `Empty`) können bei großen Bäumen Speicher kosten.

---

## Iteration & Streams

Die `Tree<T>`-Struktur ist als `Iterable<T>` implementiert. Beide Implementierungen (`Empty`, `Node`) überschreiben:

- `iterator()` – verwendet den eigenen `TreeIterator<T>`
- `forEach(...)` – für Lambda-Ausdrücke
- `spliterator()` – zur Nutzung von Streams

Damit lässt sich der Baum verwenden wie eine Liste:

```java
for (FelineOverLord cat : clowder) { ... }

    clowder.stream()
       .filter(c -> c.weight() > 2)
    .map(FelineOverLord::name)
       .forEach(System.out::println);
```

---

## TreeIterator – Inorder-Traversierung

Der `TreeIterator<T>` durchläuft den Baum iterativ in **Inorder-Reihenfolge** (Links – Wurzel – Rechts) mit einem Stack:

1. Im Konstruktor werden alle linken Teilbäume von der Wurzel aus bis zum untersten Blatt auf den Stack gelegt (`pushAllLeftNodes`).
2. Bei jedem Aufruf von `next()` wird das oberste Element vom Stack entfernt:
    - Dieses Element ist der nächste Knoten in der Inorder-Reihenfolge.
    - Anschließend wird der **rechte Teilbaum** des Knotens genommen, und wieder alle **seine linken Knoten** auf den Stack gepusht.

Dadurch wird rekursives Verhalten über einen Stack **iterativ simuliert**:
- **Linke Teilbäume** werden direkt beim Start und nach jedem Schritt vorbereitet.
- **Rechte Teilbäume** werden verzögert nachgeladen, sobald ein Knoten verarbeitet wurde.

Dies vermeidet Rekursion und spart Stack-Speicher im Java-Call-Stack.

Beispiel:

```
      5
     / \
    3   7
```

→ Iteration: `3 → 5 → 7`
