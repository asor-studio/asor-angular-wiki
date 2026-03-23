# Utilities

Le utility di `asor-core` esistono per ridurre codice ripetitivo e rendere piu` uniforme il modo in cui l'applicazione gestisce routing, oggetti, stringhe, collezioni e logging.

In questa pagina il taglio e` volutamente piu` pratico: prima capiamo a cosa serve ogni famiglia di utility, poi raccogliamo i metodi in tabelle veloci da consultare.

---

## RoutingUtility <a id="utils-routing"></a>

`RoutingUtility` e` la utility da usare quando vuoi interagire con il routing senza riscrivere ogni volta il solito codice di lettura parametri o navigazione.

### Quando ti torna utile

| Caso | Metodo tipico |
|---|---|
| Leggere query param | `getNavParams()` |
| Leggere path param | `getPathParams()` |
| Capire se sei in una certa sezione | `currentNavPass()` |
| Navigare con query string | `navigate()` |
| Navigare con state object | `navigateWithStateObj()` |
| Recuperare lo state passato | `getStateParams()` |

### Utilizzo

```typescript
import { RoutingUtility } from '@asor-studio/asor-core';
import { inject } from '@angular/core';

export class MyComponent {
    private routingUtility = inject(RoutingUtility);
}
```

### Metodi principali

| Metodo | Restituisce / Fa | Uso tipico |
|---|---|---|
| `getNavParams(key)` | Legge un query param | `/page?userId=123` |
| `getPathParams(key)` | Legge un parametro di route | `/users/456` |
| `currentNavPass(route)` | Verifica se l'URL contiene una stringa | evidenziare menu attivi |
| `navigate(route, params?)` | Naviga aggiungendo query params | passaggio pagina con filtri |
| `navigateWithStateObj(route, obj?, params?)` | Naviga passando state non visibile in URL | dettagli, wizard, handoff temporanei |
| `getStateParams()` | Recupera lo state passato in navigazione | leggere l'oggetto trasferito |

---

## ObjectUtils <a id="utils-object"></a>

`ObjectUtils` raccoglie piccoli strumenti che aiutano a lavorare in modo piu` sicuro con oggetti, path annidati e fallback.

### Quando usarla

Usala quando vuoi evitare:

- controlli null ripetuti
- if annidati per leggere proprieta` profonde
- copie manuali di oggetti
- merge sparsi e poco uniformi

### Metodi principali

| Metodo | Cosa fa |
|---|---|
| `isNullOrUndefined(obj)` | Verifica se il valore e` `null` o `undefined` |
| `isNotNullOrUndefined(obj)` | Opposto del precedente |
| `isPrimitive(value)` | Controlla se il valore e` primitivo |
| `isPlainObject(value)` | Controlla se il valore e` un oggetto semplice |
| `isArray(value)` | Verifica se il valore e` un array |
| `defaultIfNullOrUndefined(value, defaultValue)` | Applica un fallback |
| `isEmpty(obj)` | Controlla se oggetto o array e` vuoto |
| `isNotEmpty(obj)` | Opposto del precedente |
| `deepClone(obj)` | Crea una copia profonda |
| `getProperty(obj, path)` | Legge una proprieta` annidata con dot notation |
| `setProperty(obj, path, value)` | Scrive una proprieta` annidata creando i nodi mancanti |
| `assign(target, ...sources)` | Esegue un merge superficiale |

---

## StringUtils <a id="utils-string"></a>

`StringUtils` rende piu` leggibili e uniformi operazioni molto comuni sulle stringhe. Il suo valore non sta nella complessita`, ma nel fatto che centralizza casi frequenti e riduce implementazioni improvvisate.

### Metodi principali

| Metodo | Cosa fa |
|---|---|
| `isEmpty(str)` | Controlla se la stringa e` nulla, undefined o vuota |
| `isBlank(str)` | Controlla se la stringa e` vuota o solo spazi |
| `isNotEmpty(str)` | Opposto di `isEmpty` |
| `isNotBlank(str)` | Opposto di `isBlank` |
| `defaultString(str)` | Restituisce `''` se la stringa non e` valorizzata |
| `defaultIfEmpty(str, defaultStr)` | Applica fallback se la stringa e` vuota |
| `defaultIfBlank(str, defaultStr)` | Applica fallback se la stringa e` blank |
| `trim(str)` | Rimuove spazi iniziali e finali |
| `capitalize(str)` | Mette in maiuscolo il primo carattere |
| `uncapitalize(str)` | Mette in minuscolo il primo carattere |
| `equals(str1, str2)` | Confronto case-sensitive |
| `equalsIgnoreCase(str1, str2)` | Confronto case-insensitive |
| `contains(str, searchStr)` | Verifica presenza di una sottostringa |
| `startsWith(str, prefix)` | Verifica prefisso |
| `endsWith(str, suffix)` | Verifica suffisso |
| `join(elements, separator)` | Unisce un array in una stringa |
| `repeat(str, count)` | Ripete una stringa |
| `countOccurrences(str, sub)` | Conta le occorrenze di una sottostringa |

---

## CollectionUtils <a id="utils-collection"></a>

`CollectionUtils` raccoglie utilita` per lavorare con gli array in modo uniforme, soprattutto quando vuoi un comportamento null-safe o una sintassi piu` espressiva.

### Metodi principali

| Metodo | Cosa fa |
|---|---|
| `isEmpty(collection)` | Verifica se l'array e` nullo, undefined o vuoto |
| `isNotEmpty(collection)` | Opposto del precedente |
| `sort(collection, key, order?)` | Ordina per chiave |
| `getFirst(collection)` | Restituisce il primo elemento |
| `getLast(collection)` | Restituisce l'ultimo elemento |
| `concat(...arrays)` | Concatena piu` array |
| `filter(collection, predicate)` | Filtra la collezione |
| `map(collection, mapper)` | Trasforma la collezione |
| `reduce(collection, reducer, initialValue)` | Riduce a un valore singolo |
| `forEach(collection, action)` | Itera eseguendo un'azione |
| `contains(collection, item)` | Controlla se l'elemento e` presente |
| `exists(collection, predicate)` | Verifica se almeno un elemento soddisfa il predicato |
| `removeIf(collection, predicate)` | Rimuove e restituisce il primo elemento che soddisfa il predicato |
| `shallowCopy(collection)` | Crea una copia superficiale |

### Nota pratica

`removeIf` modifica l'array originale. E` utile, ma va usato sapendo che produce side effect sul dato di partenza.

---

## ConsoleLogsUtility <a id="utils-console"></a>

`ConsoleLogsUtility` non serve solo a stampare messaggi in console. In ASOR, il logging e` parte dell'osservabilita` dell'applicazione e dialoga anche con il widget di debug.

### Quando usarla

Usala quando vuoi:

- log coerenti tra classi diverse
- livelli di severita` espliciti
- controllo centralizzato della verbosita`
- compatibilita` con build minificate e widget ASOR

### Livelli principali

| Metodo | Significato |
|---|---|
| `info(...)` | Evento informativo |
| `debug(...)` | Dettaglio tecnico utile in sviluppo |
| `warning(...)` | Anomalia non bloccante |
| `error(...)` | Errore o condizione critica |

### Regola importante su `className`

Ogni classe che usa il logging ASOR dovrebbe dichiarare un `className` statico. Questo rende i log riconoscibili anche in produzione, dove il nome reale della classe JavaScript puo` essere alterato dalla minificazione.

### Configurazione dei livelli (`ConsoleLogsConfig`)

| Strumento | Ruolo |
|---|---|
| `setClassLevels(className, levels)` | Definisce i livelli attivi per una classe |
| `silent` | Nasconde i log dalla console del browser mantenendoli disponibili nel widget |
| `defaultLevels` | Livelli di default applicati dal sistema |

### Nota operativa

Se una classe non viene configurata esplicitamente, il comportamento documentato e` silenzioso di default. Questo aiuta a evitare rumore, ma implica che in sviluppo convenga attivare in modo intenzionale i livelli che servono davvero.
