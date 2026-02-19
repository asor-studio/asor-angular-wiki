# Utilities

In questa sezione vengono descritte le utility disponibili in `asor-core` per semplificare operazioni comuni.

## <a id="utils-routing"></a>RoutingUtility

`RoutingUtility` è un servizio iniettabile (`providedIn: 'root'`) che fornisce metodi per gestire la navigazione e il routing nelle applicazioni Angular, semplificando operazioni come il recupero di parametri e la navigazione con stato.

### Utilizzo

Per utilizzare `RoutingUtility`, è sufficiente iniettarlo nel costruttore o tramite la funzione `inject`.

```typescript
import { RoutingUtility } from '@asor-studio/asor-core';
import { inject } from '@angular/core';

export class MyComponent {
    private routingUtility = inject(RoutingUtility);
    // oppure
    // constructor(private routingUtility: RoutingUtility) {}
}
```

### Metodi Principali

#### `getNavParams(key: string): string`
Recupera il valore di un parametro di query dalla rotta corrente.

```typescript
// URL: /page?userId=123
const userId = this.routingUtility.getNavParams('userId'); // Restituisce '123'
```

#### `getPathParams(key: string): string`
Recupera il valore di un parametro di percorso definito nella configurazione della rotta.

```typescript
// Route config: { path: 'users/:id', component: UserComponent }
// URL: /users/456
const id = this.routingUtility.getPathParams('id'); // Restituisce '456'
```

#### `currentNavPass(route: string): boolean`
Verifica se l'URL corrente contiene una stringa specifica. Utile per evidenziare voci di menu attive.

```typescript
// Current URL: /dashboard/settings
const isDashboard = this.routingUtility.currentNavPass('dashboard'); // Restituisce true
```

#### `navigate(route: string, params?: any): void`
Naviga verso una rotta specificata, opzionalmente aggiungendo parametri di query. Gestisce automaticamente la rimozione del `SiteBaseUrl`.

```typescript
this.routingUtility.navigate('/users', { page: 1, sort: 'name' });
// Naviga verso: /users?page=1&sort=name
```

#### `navigateWithStateObj(route: string, obj?: any, params?: any)`
Naviga verso una rotta passando un oggetto di stato (non visibile nell'URL) e opzionalmente parametri di query.

```typescript
const userData = { name: 'Mario', role: 'Admin' };
this.routingUtility.navigateWithStateObj('/details', { user: userData }, { id: 123 });
// Naviga verso: /details?id=123 (con userData nello stato della history)
```

#### `getStateParams(): any`
Recupera l'oggetto di stato passato durante la navigazione con `navigateWithStateObj`.

```typescript
const stateData = this.routingUtility.getStateParams();
// Restituisce l'oggetto { user: userData } passato nella navigazione precedente
```

---

## <a id="utils-object"></a>ObjectUtils

`ObjectUtils` fornisce metodi statici per operazioni comuni su oggetti.

#### `isNullOrUndefined(obj: any): boolean`
Verifica se un oggetto è `null` o `undefined`.

```typescript
ObjectUtils.isNullOrUndefined(null); // true
ObjectUtils.isNullOrUndefined(undefined); // true
ObjectUtils.isNullOrUndefined(0); // false
```

#### `isNotNullOrUndefined(obj: any): boolean`
Opposto di `isNullOrUndefined`.

```typescript
ObjectUtils.isNotNullOrUndefined({}); // true
```

#### `isPrimitive(value: any): boolean`
Verifica se il valore è un tipo primitivo (string, number, boolean, symbol, bigint, null, undefined).

```typescript
ObjectUtils.isPrimitive(123); // true
ObjectUtils.isPrimitive("test"); // true
ObjectUtils.isPrimitive({}); // false
```

#### `isPlainObject(value: any): boolean`
Verifica se il valore è un oggetto JavaScript semplice (non array, null, o oggetti nativi come Date).

```typescript
ObjectUtils.isPlainObject({}); // true
ObjectUtils.isPlainObject(new Date()); // false
ObjectUtils.isPlainObject([]); // false
```

#### `isArray(value: any): boolean`
Verifica se il valore è un array. Wrapper per `Array.isArray()`.

```typescript
ObjectUtils.isArray([]); // true
ObjectUtils.isArray({}); // false
```

#### `defaultIfNullOrUndefined<T>(value: T, defaultValue: T): T`
Restituisce `value` se valido, altrimenti restituisce `defaultValue`.

```typescript
ObjectUtils.defaultIfNullOrUndefined(null, 'default'); // 'default'
ObjectUtils.defaultIfNullOrUndefined('valid', 'default'); // 'valid'
```

#### `isEmpty(obj: object): boolean`
Verifica se un oggetto è "vuoto" (nessuna proprietà enumerabile), un array è vuoto, o se è null/undefined.

```typescript
ObjectUtils.isEmpty({}); // true
ObjectUtils.isEmpty([]); // true
ObjectUtils.isEmpty(null); // true
ObjectUtils.isEmpty({a: 1}); // false
```

#### `isNotEmpty(obj: object): boolean`
Opposto di `isEmpty`.

```typescript
ObjectUtils.isNotEmpty({a: 1}); // true
```

#### `deepClone<T>(obj: T): T`
Esegue una copia profonda (deep copy) di un oggetto o array.

```typescript
const original = { a: 1, b: { c: 2 } };
const clone = ObjectUtils.deepClone(original);
clone.b.c = 3;
console.log(original.b.c); // 2 (non modificato)
```

#### `getProperty<T>(obj: any, path: string): T | undefined`
Recupera il valore di una proprietà annidata specificata da un percorso stringa (dot-notation). Restituisce `undefined` se il percorso non esiste.

```typescript
const data = { user: { address: { city: 'Rome' } } };
const city = ObjectUtils.getProperty(data, 'user.address.city'); // 'Rome'
const zip = ObjectUtils.getProperty(data, 'user.address.zip'); // undefined
```

#### `setProperty(obj: any, path: string, value: any): any`
Imposta il valore di una proprietà annidata specificata da un percorso stringa. Crea gli oggetti intermedi se mancanti.

```typescript
const data = {};
ObjectUtils.setProperty(data, 'user.name', 'Mario');
// data diventa { user: { name: 'Mario' } }
```

#### `assign<T>(target: T, ...sources: object[]): T`
Unisce le proprietà di uno o più oggetti sorgente nell'oggetto target (shallow copy). Simile a `Object.assign`.

```typescript
const target = { a: 1 };
ObjectUtils.assign(target, { b: 2 }, { c: 3 });
// target diventa { a: 1, b: 2, c: 3 }
```

---

## <a id="utils-string"></a>StringUtils

`StringUtils` offre metodi statici per la manipolazione e il controllo di stringhe.

#### `isEmpty(str: string): boolean`
Verifica se la stringa è `null`, `undefined` o ha lunghezza 0.

```typescript
StringUtils.isEmpty(null); // true
StringUtils.isEmpty(''); // true
StringUtils.isEmpty(' '); // false
```

#### `isBlank(str: string): boolean`
Verifica se la stringa è `null`, `undefined`, vuota o composta solo da spazi bianchi.

```typescript
StringUtils.isBlank(null); // true
StringUtils.isBlank(''); // true
StringUtils.isBlank('   '); // true
StringUtils.isBlank(' a '); // false
```

#### `isNotEmpty(str: string): boolean`
Opposto di `isEmpty`.

```typescript
StringUtils.isNotEmpty(' '); // true
```

#### `isNotBlank(str: string): boolean`
Opposto di `isBlank`.

```typescript
StringUtils.isNotBlank('   '); // false
StringUtils.isNotBlank(' a '); // true
```

#### `defaultString(str: string): string`
Restituisce la stringa in input oppure una stringa vuota `''` se l'input è `null` o `undefined`.

```typescript
StringUtils.defaultString(null); // ''
StringUtils.defaultString('abc'); // 'abc'
```

#### `defaultIfEmpty(str: string, defaultStr: string): string`
Restituisce `defaultStr` se `str` è vuota (`isEmpty`), altrimenti restituisce `str`.

```typescript
StringUtils.defaultIfEmpty('', 'default'); // 'default'
StringUtils.defaultIfEmpty('val', 'default'); // 'val'
```

#### `defaultIfBlank(str: string, defaultStr: string): string`
Restituisce `defaultStr` se `str` è blank (`isBlank`), altrimenti restituisce `str`.

```typescript
StringUtils.defaultIfBlank('   ', 'default'); // 'default'
```

#### `trim(str: string): string`
Rimuove gli spazi iniziali e finali. Gestisce `null` e `undefined` restituendo stringa vuota.

```typescript
StringUtils.trim('  abc  '); // 'abc'
StringUtils.trim(null); // ''
```

#### `capitalize(str: string): string`
Converte il primo carattere della stringa in maiuscolo.

```typescript
StringUtils.capitalize('ciao'); // 'Ciao'
```

#### `uncapitalize(str: string): string`
Converte il primo carattere della stringa in minuscolo.

```typescript
StringUtils.uncapitalize('Ciao'); // 'ciao'
```

#### `equals(str1: string, str2: string): boolean`
Confronta due stringhe (case-sensitive). Gestisce `null` e `undefined`.

```typescript
StringUtils.equals('abc', 'abc'); // true
StringUtils.equals('abc', 'Abc'); // false
StringUtils.equals(null, null); // true
```

#### `equalsIgnoreCase(str1: string, str2: string): boolean`
Confronta due stringhe ignorando maiuscole/minuscole.

```typescript
StringUtils.equalsIgnoreCase('abc', 'ABC'); // true
```

#### `contains(str: string, searchStr: string): boolean`
Verifica se `str` contiene `searchStr`.

```typescript
StringUtils.contains('hello world', 'world'); // true
```

#### `startsWith(str: string, prefix: string): boolean`
Verifica se `str` inizia con `prefix`.

```typescript
StringUtils.startsWith('hello', 'he'); // true
```

#### `endsWith(str: string, suffix: string): boolean`
Verifica se `str` finisce con `suffix`.

```typescript
StringUtils.endsWith('hello', 'lo'); // true
```

#### `join(elements: any[], separator: string): string`
Unisce elementi di un array in una stringa separati da `separator`.

```typescript
StringUtils.join(['a', 'b', 'c'], '-'); // 'a-b-c'
```

#### `repeat(str: string, count: number): string`
Ripete la stringa `count` volte.

```typescript
StringUtils.repeat('a', 3); // 'aaa'
```

#### `countOccurrences(str: string, sub: string): number`
Conta le occorrenze di `sub` all'interno di `str`.

```typescript
StringUtils.countOccurrences('bananas', 'a'); // 3
```

---

## CollectionUtils <a id="utils-collection"></a>

Utility per lavorare con Array e Collezioni.

### Metodi Principali

#### `isEmpty(collection: any[]): boolean`
Verifica se un array è nullo, indefinito o vuoto.
```typescript
CollectionUtils.isEmpty([]); // true
```

#### `isNotEmpty(collection: any[]): boolean`
Contrario di isEmpty.
```typescript
CollectionUtils.isNotEmpty([1]); // true
```

#### `sort<T>(collection: T[], key: keyof T, order: 'asc' | 'desc' = 'asc')`
Ordina un array di oggetti in base a una chiave specifica.
```typescript
const users = [{ name: 'B' }, { name: 'A' }];
CollectionUtils.sort(users, 'name'); // [{ name: 'A' }, { name: 'B' }]
```

#### `isEmpty(collection: any[]): boolean`
Verifica se un array è `null`, `undefined` o vuoto.

```typescript
CollectionUtils.isEmpty([]); // true
CollectionUtils.isEmpty(null); // true
CollectionUtils.isEmpty([1]); // false
```

#### `isNotEmpty(collection: any[]): boolean`
Opposto di `isEmpty`.

```typescript
CollectionUtils.isNotEmpty([1]); // true
```

#### `getFirst<T>(collection: T[]): T | undefined`
Restituisce il primo elemento o `undefined` se vuoto.

```typescript
CollectionUtils.getFirst([10, 20]); // 10
CollectionUtils.getFirst([]); // undefined
```

#### `getLast<T>(collection: T[]): T | undefined`
Restituisce l'ultimo elemento o `undefined` se vuoto.

```typescript
CollectionUtils.getLast([10, 20]); // 20
```

#### `concat<T>(...arrays: T[][]): T[]`
Concatena più array in un nuovo array.

```typescript
CollectionUtils.concat([1, 2], [3, 4]); // [1, 2, 3, 4]
```

#### `filter<T>(collection: T[], predicate): T[]`
Filtra una collezione. Restituisce array vuoto se input è null.

```typescript
CollectionUtils.filter([1, 2, 3, 4], x => x % 2 === 0); // [2, 4]
```

#### `map<T, U>(collection: T[], mapper): U[]`
Trasforma gli elementi di una collezione. Safe su null.

```typescript
CollectionUtils.map([1, 2], x => x * 2); // [2, 4]
```

#### `reduce<T, U>(collection: T[], reducer, initialValue): U`
Riduce una collezione a un singolo valore.

```typescript
CollectionUtils.reduce([1, 2, 3], (acc, x) => acc + x, 0); // 6
```

#### `forEach<T>(collection: T[], action): void`
Esegue un'azione per ogni elemento. Safe su null.

```typescript
CollectionUtils.forEach([1, 2], x => console.log(x));
```

#### `contains<T>(collection: T[], item: T): boolean`
Verifica se l'elemento è presente nella collezione.

```typescript
CollectionUtils.contains([1, 2, 3], 2); // true
```

#### `exists<T>(collection: T[], predicate): boolean`
Verifica se esiste almeno un elemento che soddisfa il predicato.

```typescript
CollectionUtils.exists([1, 2, 3], x => x > 2); // true
```

#### `removeIf<T>(collection: T[], predicate): T | undefined`
Rimuove il primo elemento che soddisfa il predicato e lo restituisce. Modifica l'array originale.

```typescript
const arr = [1, 2, 3, 4];
const removed = CollectionUtils.removeIf(arr, x => x % 2 === 0); 
// removed = 2, arr = [1, 3, 4]
```

#### `shallowCopy<T>(collection: T[]): T[]`
Crea una copia superficiale dell'array.

```typescript
const copy = CollectionUtils.shallowCopy(originalArray);
```

---

## <a id="utils-console"></a>ConsoleLogsUtility

`ConsoleLogsUtility` fornisce un sistema di logging avanzato e stilizzato per la console del browser. Permette di categorizzare i log per livello (INFO, DEBUG, WARNING, ERROR) e di filtrarli dinamicamente per classe.

### Utilizzo

Per utilizzare la utility, invocare i metodi statici passando l'istanza della classe chiamante (`this`) come primo argomento. Questo permette alla utility di identificare la sorgente del log.

```typescript
import { ConsoleLogsUtility } from '@asor-studio/asor-core';

export class MyService {
    // IMPORTANTE: Definire className per garantire il funzionamento dei log in produzione (build minificata)
    public static readonly className = 'MyService';

    doSomething() {
        // Log Info
        ConsoleLogsUtility.info(this, 'Inizio operazione', {id: 123});
        
        // Log Debug
        ConsoleLogsUtility.debug(this, 'Dettagli oggetto', myObject);
        
        // Log Warning
        ConsoleLogsUtility.warning(this, 'Attenzione: valore mancante');
        
        // Log Error
        ConsoleLogsUtility.error(this, 'Errore critico', errorObj);
    }
}
```

### Configurazione dei Livelli (`ConsoleLogsConfig`)

Per evitare di inquinare la console in produzione, è possibile abilitare o disabilitare i livelli di log per specifiche classi.

La configurazione deve essere effettuata preferibilmente nel file `src/app/app.root.ts` (o nel componente root dell'applicazione), utilizzando `ConsoleLogsConfig.setClassLevels`.

#### Metodo `setClassLevels(className: string, levels: string[])`

*   `className`: Il nome della classe (es. `'MyService'`). **Utilizzare la proprietà statica `className`** (`MyService.className`) per garantire la coerenza nelle build di produzione dove i nomi delle classi vengono minificati.
*   `levels`: Array di livelli abilitati (`'INFO'`, `'DEBUG'`, `'WARNING'`, `'ERROR'`).

> **Nota**: Se una classe non è configurata, di default **NON** mostrerà alcun log (comportamento "silenzioso per default"). È buona norma abilitare esplicitamente ciò che serve in fase di sviluppo.

#### Esempio Configurazione (`app.root.ts`)

```typescript
// src/app/app.root.ts
import { ConsoleLogsConfig, StateService } from '@asor-studio/asor-core';

export class AppRoot {
    constructor() {
        // Abilita solo ERROR e WARNING per StateService usando static className
        ConsoleLogsConfig.setClassLevels(StateService.className || '', ['ERROR', 'WARNING']);
        
        // Abilita TUTTO per MyCustomComponent
        ConsoleLogsConfig.setClassLevels(MyCustomComponent.className || '', ['INFO', 'DEBUG', 'WARNING', 'ERROR']);
    }
}
```

### Modalità Silenziosa (Silent Mode)

Se si desidera mantenere i log attivi nel widget dell'applicazione ma sopprimerli dalla console del browser (es. in produzione), è possibile abilitare la "Silent Mode".

Si consiglia di impostarlo in `AppRoot` o in un inizializzatore principale.

```typescript
// src/app/app.root.ts
import { ConsoleLogsConfig } from '@asor-studio/asor-core';

export class AppRoot {
    constructor() {
        // Nasconde i log dai DevTools del browser
        ConsoleLogsConfig.silent = true;
    }
}
```
