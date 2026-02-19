# Utilities

This section describes the utilities available in `asor-core` to simplify common operations.

## <a id="utils-routing"></a>RoutingUtility

`RoutingUtility` is an injectable service (`providedIn: 'root'`) that provides methods to manage navigation and routing in Angular applications, simplifying operations like retrieving parameters and navigation with state.

### Usage

To use `RoutingUtility`, inject it into the constructor or via the `inject` function.

```typescript
import { RoutingUtility } from '@asor-studio/asor-core';
import { inject } from '@angular/core';

export class MyComponent {
    private routingUtility = inject(RoutingUtility);
    // or
    // constructor(private routingUtility: RoutingUtility) {}
}
```

### Main Methods

#### `getNavParams(key: string): string`
Retrieves the value of a query parameter from the current route.

```typescript
// URL: /page?userId=123
const userId = this.routingUtility.getNavParams('userId'); // Returns '123'
```

#### `getPathParams(key: string): string`
Retrieves the value of a path parameter defined in route configuration.

```typescript
// Route config: { path: 'users/:id', component: UserComponent }
// URL: /users/456
const id = this.routingUtility.getPathParams('id'); // Returns '456'
```

#### `currentNavPass(route: string): boolean`
Checks if the current URL contains a specific string. Useful for highlighting active menu items.

```typescript
// Current URL: /dashboard/settings
const isDashboard = this.routingUtility.currentNavPass('dashboard'); // Returns true
```

#### `navigate(route: string, params?: any): void`
Navigates to a specified route, optionally adding query parameters. Automatically handles `SiteBaseUrl` removal.

```typescript
this.routingUtility.navigate('/users', { page: 1, sort: 'name' });
// Navigates to: /users?page=1&sort=name
```

#### `navigateWithStateObj(route: string, obj?: any, params?: any)`
Navigates to a route passing a state object (not visible in URL) and optionally query parameters.

```typescript
const userData = { name: 'Mario', role: 'Admin' };
this.routingUtility.navigateWithStateObj('/details', { user: userData }, { id: 123 });
// Navigates to: /details?id=123 (with userData in history state)
```

#### `getStateParams(): any`
Retrieves the state object passed during navigation with `navigateWithStateObj`.

```typescript
const stateData = this.routingUtility.getStateParams();
// Returns the object { user: userData } passed in previous navigation
```

---

## <a id="utils-object"></a>ObjectUtils

`ObjectUtils` provides static methods for common object operations.

#### `isNullOrUndefined(obj: any): boolean`
Checks if an object is `null` or `undefined`.

```typescript
ObjectUtils.isNullOrUndefined(null); // true
ObjectUtils.isNullOrUndefined(undefined); // true
ObjectUtils.isNullOrUndefined(0); // false
```

#### `isNotNullOrUndefined(obj: any): boolean`
Opposite of `isNullOrUndefined`.

```typescript
ObjectUtils.isNotNullOrUndefined({}); // true
```

#### `isPrimitive(value: any): boolean`
Checks if the value is a primitive type (string, number, boolean, symbol, bigint, null, undefined).

```typescript
ObjectUtils.isPrimitive(123); // true
ObjectUtils.isPrimitive("test"); // true
ObjectUtils.isPrimitive({}); // false
```

#### `isPlainObject(value: any): boolean`
Checks if the value is a plain JavaScript object (not array, null, or native objects like Date).

```typescript
ObjectUtils.isPlainObject({}); // true
ObjectUtils.isPlainObject(new Date()); // false
ObjectUtils.isPlainObject([]); // false
```

#### `isArray(value: any): boolean`
Checks if the value is an array. Wrapper for `Array.isArray()`.

```typescript
ObjectUtils.isArray([]); // true
ObjectUtils.isArray({}); // false
```

#### `defaultIfNullOrUndefined<T>(value: T, defaultValue: T): T`
Returns `value` if valid, otherwise returns `defaultValue`.

```typescript
ObjectUtils.defaultIfNullOrUndefined(null, 'default'); // 'default'
ObjectUtils.defaultIfNullOrUndefined('valid', 'default'); // 'valid'
```

#### `isEmpty(obj: object): boolean`
Checks if an object is "empty" (no enumerable properties), an array is empty, or if it is null/undefined.

```typescript
ObjectUtils.isEmpty({}); // true
ObjectUtils.isEmpty([]); // true
ObjectUtils.isEmpty(null); // true
ObjectUtils.isEmpty({a: 1}); // false
```

#### `isNotEmpty(obj: object): boolean`
Opposite of `isEmpty`.

```typescript
ObjectUtils.isNotEmpty({a: 1}); // true
```

#### `deepClone<T>(obj: T): T`
Performs a deep copy of an object or array.

```typescript
const original = { a: 1, b: { c: 2 } };
const clone = ObjectUtils.deepClone(original);
clone.b.c = 3;
console.log(original.b.c); // 2 (not modified)
```

#### `getProperty<T>(obj: any, path: string): T | undefined`
Retrieves the value of a nested property specified by a string path (dot-notation). Returns `undefined` if the path does not exist.

```typescript
const data = { user: { address: { city: 'Rome' } } };
const city = ObjectUtils.getProperty(data, 'user.address.city'); // 'Rome'
const zip = ObjectUtils.getProperty(data, 'user.address.zip'); // undefined
```

#### `setProperty(obj: any, path: string, value: any): any`
Sets the value of a nested property specified by a string path. Creates intermediate objects if missing.

```typescript
const data = {};
ObjectUtils.setProperty(data, 'user.name', 'Mario');
// data becomes { user: { name: 'Mario' } }
```

#### `assign<T>(target: T, ...sources: object[]): T`
Merges properties of one or more source objects into the target object (shallow copy). Similar to `Object.assign`.

```typescript
const target = { a: 1 };
ObjectUtils.assign(target, { b: 2 }, { c: 3 });
// target becomes { a: 1, b: 2, c: 3 }
```

---

## <a id="utils-string"></a>StringUtils

`StringUtils` offers static methods for string manipulation and checking.

#### `isEmpty(str: string): boolean`
Checks if the string is `null`, `undefined` or has length 0.

```typescript
StringUtils.isEmpty(null); // true
StringUtils.isEmpty(''); // true
StringUtils.isEmpty(' '); // false
```

#### `isBlank(str: string): boolean`
Checks if the string is `null`, `undefined`, empty or composed only of whitespace.

```typescript
StringUtils.isBlank(null); // true
StringUtils.isBlank(''); // true
StringUtils.isBlank('   '); // true
StringUtils.isBlank(' a '); // false
```

#### `isNotEmpty(str: string): boolean`
Opposite of `isEmpty`.

```typescript
StringUtils.isNotEmpty(' '); // true
```

#### `isNotBlank(str: string): boolean`
Opposite of `isBlank`.

```typescript
StringUtils.isNotBlank('   '); // false
StringUtils.isNotBlank(' a '); // true
```

#### `defaultString(str: string): string`
Returns the input string or an empty string `''` if input is `null` or `undefined`.

```typescript
StringUtils.defaultString(null); // ''
StringUtils.defaultString('abc'); // 'abc'
```

#### `defaultIfEmpty(str: string, defaultStr: string): string`
Returns `defaultStr` if `str` is empty (`isEmpty`), otherwise returns `str`.

```typescript
StringUtils.defaultIfEmpty('', 'default'); // 'default'
StringUtils.defaultIfEmpty('val', 'default'); // 'val'
```

#### `defaultIfBlank(str: string, defaultStr: string): string`
Returns `defaultStr` if `str` is blank (`isBlank`), otherwise returns `str`.

```typescript
StringUtils.defaultIfBlank('   ', 'default'); // 'default'
```

#### `trim(str: string): string`
Removes leading and trailing whitespace. Handles `null` and `undefined` by returning empty string.

```typescript
StringUtils.trim('  abc  '); // 'abc'
StringUtils.trim(null); // ''
```

#### `capitalize(str: string): string`
Converts the first character of the string to uppercase.

```typescript
StringUtils.capitalize('hello'); // 'Hello'
```

#### `uncapitalize(str: string): string`
Converts the first character of the string to lowercase.

```typescript
StringUtils.uncapitalize('Hello'); // 'hello'
```

#### `equals(str1: string, str2: string): boolean`
Compares two strings (case-sensitive). Handles `null` and `undefined`.

```typescript
StringUtils.equals('abc', 'abc'); // true
StringUtils.equals('abc', 'Abc'); // false
StringUtils.equals(null, null); // true
```

#### `equalsIgnoreCase(str1: string, str2: string): boolean`
Compares two strings ignoring case.

```typescript
StringUtils.equalsIgnoreCase('abc', 'ABC'); // true
```

#### `contains(str: string, searchStr: string): boolean`
Checks if `str` contains `searchStr`.

```typescript
StringUtils.contains('hello world', 'world'); // true
```

#### `startsWith(str: string, prefix: string): boolean`
Checks if `str` starts with `prefix`.

```typescript
StringUtils.startsWith('hello', 'he'); // true
```

#### `endsWith(str: string, suffix: string): boolean`
Checks if `str` ends with `suffix`.

```typescript
StringUtils.endsWith('hello', 'lo'); // true
```

#### `join(elements: any[], separator: string): string`
Joins elements of an array into a string separated by `separator`.

```typescript
StringUtils.join(['a', 'b', 'c'], '-'); // 'a-b-c'
```

#### `repeat(str: string, count: number): string`
Repeats the string `count` times.

```typescript
StringUtils.repeat('a', 3); // 'aaa'
```

#### `countOccurrences(str: string, sub: string): number`
Counts occurrences of `sub` within `str`.

```typescript
StringUtils.countOccurrences('bananas', 'a'); // 3
```

---

## CollectionUtils <a id="utils-collection"></a>

Utility for working with Arrays and Collections.

### Main Methods

#### `isEmpty(collection: any[]): boolean`
Checks if an array is null, undefined or empty.
```typescript
CollectionUtils.isEmpty([]); // true
```

#### `isNotEmpty(collection: any[]): boolean`
Opposite of isEmpty.
```typescript
CollectionUtils.isNotEmpty([1]); // true
```

#### `sort<T>(collection: T[], key: keyof T, order: 'asc' | 'desc' = 'asc')`
Sorts an array of objects based on a specific key.
```typescript
const users = [{ name: 'B' }, { name: 'A' }];
CollectionUtils.sort(users, 'name'); // [{ name: 'A' }, { name: 'B' }]
```

#### `getFirst<T>(collection: T[]): T | undefined`
Returns the first element or `undefined` if empty.

```typescript
CollectionUtils.getFirst([10, 20]); // 10
CollectionUtils.getFirst([]); // undefined
```

#### `getLast<T>(collection: T[]): T | undefined`
Returns the last element or `undefined` if empty.

```typescript
CollectionUtils.getLast([10, 20]); // 20
```

#### `concat<T>(...arrays: T[][]): T[]`
Concatenates multiple arrays into a new array.

```typescript
CollectionUtils.concat([1, 2], [3, 4]); // [1, 2, 3, 4]
```

#### `filter<T>(collection: T[], predicate): T[]`
Filters a collection. Returns empty array if input is null.

```typescript
CollectionUtils.filter([1, 2, 3, 4], x => x % 2 === 0); // [2, 4]
```

#### `map<T, U>(collection: T[], mapper): U[]`
Transforms elements of a collection. Safe on null.

```typescript
CollectionUtils.map([1, 2], x => x * 2); // [2, 4]
```

#### `reduce<T, U>(collection: T[], reducer, initialValue): U`
Reduces a collection to a single value.

```typescript
CollectionUtils.reduce([1, 2, 3], (acc, x) => acc + x, 0); // 6
```

#### `forEach<T>(collection: T[], action): void`
Executes an action for each element. Safe on null.

```typescript
CollectionUtils.forEach([1, 2], x => console.log(x));
```

#### `contains<T>(collection: T[], item: T): boolean`
Checks if the element is present in the collection.

```typescript
CollectionUtils.contains([1, 2, 3], 2); // true
```

#### `exists<T>(collection: T[], predicate): boolean`
Checks if at least one element satisfies the predicate.

```typescript
CollectionUtils.exists([1, 2, 3], x => x > 2); // true
```

#### `removeIf<T>(collection: T[], predicate): T | undefined`
Removes the first element satisfying the predicate and returns it. Modifies the original array.

```typescript
const arr = [1, 2, 3, 4];
const removed = CollectionUtils.removeIf(arr, x => x % 2 === 0); 
// removed = 2, arr = [1, 3, 4]
```

#### `shallowCopy<T>(collection: T[]): T[]`
Creates a shallow copy of the array.

```typescript
const copy = CollectionUtils.shallowCopy(originalArray);
```

---

## <a id="utils-console"></a>ConsoleLogsUtility

`ConsoleLogsUtility` provides an advanced and styled logging system for the browser console. It allows categorizing logs by level (INFO, DEBUG, WARNING, ERROR) and filtering them dynamically by class.

### Usage

To use the utility, invoke static methods passing the calling class instance (`this`) as the first argument. This allows the utility to identify the log source.

```typescript
import { ConsoleLogsUtility } from '@asor-studio/asor-core';

export class MyService {
    // IMPORTANT: Define className to ensure logs work correctly in production (minified build)
    public static readonly className = 'MyService';

    doSomething() {
        // Log Info
        ConsoleLogsUtility.info(this, 'Start operation', {id: 123});
        
        // Log Debug
        ConsoleLogsUtility.debug(this, 'Object details', myObject);
        
        // Log Warning
        ConsoleLogsUtility.warning(this, 'Warning: missing value');
        
        // Log Error
        ConsoleLogsUtility.error(this, 'Critical error', errorObj);
    }
}
```

### Level Configuration (`ConsoleLogsConfig`)

To avoid polluting the console in production, you can enable or disable log levels for specific classes.

Configuration must be done preferably in `src/app/app.root.ts` (or in the application root component), using `ConsoleLogsConfig.setClassLevels`.

#### Method `setClassLevels(className: string, levels: string[])`

*   `className`: Class name (e.g., `'MyService'`). **Use the static `className` property** (`MyService.className`) to ensure consistency in production builds where class names are minified.
*   `levels`: Array of enabled levels (`'INFO'`, `'DEBUG'`, `'WARNING'`, `'ERROR'`).

> **Note**: If a class is not configured, it will show **NO** logs by default ("silent by default" behavior). It is good practice to explicitly enable what is needed during development.

#### Configuration Example (`app.root.ts`)

```typescript
// src/app/app.root.ts
import { ConsoleLogsConfig, StateService } from '@asor-studio/asor-core';

export class AppRoot {
    constructor() {
        // Enable only ERROR and WARNING for StateService using static className
        ConsoleLogsConfig.setClassLevels(StateService.className || '', ['ERROR', 'WARNING']);
        
        // Enable ALL for MyCustomComponent
        ConsoleLogsConfig.setClassLevels(MyCustomComponent.className || '', ['INFO', 'DEBUG', 'WARNING', 'ERROR']);
    }
}
```

### Silent Mode

If you want to keep logs active in the application widget but suppress them from the browser console (e.g., in production), you can enable "Silent Mode".

It is recommended to set this in the `AppRoot` or a core initializer.

```typescript
// src/app/app.root.ts
import { ConsoleLogsConfig } from '@asor-studio/asor-core';

export class AppRoot {
    constructor() {
        // Suppress logs from browser DevTools
        ConsoleLogsConfig.silent = true;
    }
}
```
