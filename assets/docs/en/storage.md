# Storage Management (DataSet)

In this section, we explain how to configure, apply, and initialize storage (DataSet) within the application using `ICreateDataSet` and `IConnectDataSet`.

## 1. Defining Configuration (`ICreateDataSet`) <a id="storage-definition"></a>

To define a new storage (DataSet), you need to create an object that respects the `ICreateDataSet` interface. This object defines the container name, initial data, and configuration options.

### Example (`example.storage.const.ts`)

```typescript
import { AsorStorage, ICreateDataSet } from '@asor-studio/asor-core';

export const ExampleCreateDataSet: ICreateDataSet = {
    name: 'example-component', // Container name
    data: {
        title: 'Hi {name}, you have {age} years.',
        form: {
            name: 'John',
            age: 30,
            jobRole: [
                {
                    title: 'Developer',
                    description: 'Developer',
                    selected: false
                }, {
                    title: 'Designer',
                    description: 'Designer',
                    selected: false
                }]
        }
    } as IExampleProps,
    option: {
        storeType: AsorStorage.StateConst.StoreType.VOLATILE,
        encrypt: true,
        freeze: false
    }
};
```

### Configuration Option Details <a id="storage-config-details"></a>

The `option` property in the `ICreateDataSet` object allows configuring storage behavior for this specific dataset.

#### Storage Types (`storeType`)
There are 3 available storage types:
1. **VOLATILE**: Data is kept only in memory. It is lost upon page refresh. This is the default and most performant option.
2. **SESSION**: Data is saved in the browser's `SessionStorage`. It survives page refresh but is lost when the tab or browser is closed.
3. **LOCAL**: Data is saved in the browser's `LocalStorage`. It persists even after closing and reopening the browser.

#### Encryption (`encrypt`)
By setting `encrypt: true`, the dataset content is encrypted before being saved to storage (useful for Session and Local storage) to ensure greater security for sensitive data.

#### Immutability (`freeze`)
By setting `freeze: true`, the initial content of the dataset is made immutable ("frozen") at creation time. This is useful for preventing accidental modifications to configuration data or constants, ensuring the state remains consistent.

---

## 2. Connect DataSet (`IConnectDataSet`) <a id="storage-connection"></a>

To connect the container to a component or molecule and deliver data to their `props`, use an `IConnectDataSet` object.

This object defines selectors mapping storage properties to component properties. When data in storage changes, the connected properties are automatically updated.

### Example (`example.storage.const.ts`)

```typescript
import { IConnectDataSet } from '@asor-studio/asor-core';

export const ExampleConnectDataSet: IConnectDataSet = {
    selectors: {
        title: 'example-component.title', // Maps component 'title' to storage 'example-component.title'
        form: 'example-component.form'    // Maps component 'form' to storage 'example-component.form'
    }
};
```

---

## 3. Applying Configuration to Routes <a id="storage-application"></a>

Associate storage configurations (`CreateDataSet`) and connections (`ConnectDataSet`) to the routes that need access to them. This is done in the route's `data`.

### Configuration Explanation
How it connects to the route:
- **CreateDataSet**: Creates the container with data.
- **ConnectDataSet**: Connects the container to the molecule or component, providing container data in the `props` variable.
- **Components / Molecules**: Defines the container selectors to which the update event is bound.

### Example (`app.routes.ts`)

```typescript
{
    path: WikiConfig.Route.STORAGE,
    component: PageStorageComponent,
    data: {
        // ... other configurations (I18nPath, AuthCheck, etc.)
        
        // Creates the container with data
        CreateDataSet: ExampleCreateDataSet,
        
        Components: [
            {
                Component: ExampleStorageComponent,
                I18nPath: [WikiConfig.TranslationUrl.STORAGE],
                // Connects the container to this component
                ConnectDataSet: ExampleConnectDataSet
            }
        ],
        Molecules: [
            {
                Molecule: ExampleStorageMolecule,
                I18nPath: [WikiConfig.TranslationUrl.STORAGE],
                // Connects the container to this molecule
                ConnectDataSet: ExampleConnectDataSet
            }
        ],
    } as IAsorRoute
},
```

In this way, when navigating to the `STORAGE` route:
1.  The `example-component` DataSet is created with initial data.
2.  `ExampleStorageComponent` and `ExampleStorageMolecule` automatically receive updated data via `ExampleConnectDataSet`.

> **Important**: For the route matching to work correctly, every component and molecule listed in `Components` or `Molecules` **must** declare a `public static override readonly className` property (see next section).

---

## 4. Component Identification (`className`) <a id="storage-classname"></a>

Every class that extends `BaseComponent`, `BaseMolecule`, `BaseStorageComponent`, or `BaseStorageMolecule` **must** declare a static `className` property:

```typescript
public static override readonly className: string = 'MyComponentName';
```

This property is used by the framework to:
- **Match components to route configuration**: When a route is activated, the system compares `Component.className` (or `Molecule.className`) from the route data with the class being instantiated to deliver the correct `ConnectDataSet` and `I18nPath`.
- **Identify classes in logs**: `ConsoleLogsUtility` uses `className` to prefix log messages, ensuring readable logs even in production builds where JavaScript class names are minified.

### Why `static`?
The property must be **static** because it needs to be available on the class constructor itself (not on an instance), ensuring it can be read during route matching *before* the component is fully constructed. The `ICompStatic` interface enforces this at the type level:

```typescript
export interface ICompStatic {
    readonly className: string;
}
```

The `IComponent` and `IMolecule` interfaces require `Type<BaseComponent> & ICompStatic` and `Type<BaseMolecule> & ICompStatic` respectively, so TypeScript will raise a compilation error if a class used in route configuration does not declare the static `className`.

---

## 5. Service Initialization <a id="storage-init"></a>

The `StateService` represents the core of the state management system. It must be initialized, usually in the application's root component (`src/app/app.root.ts`), to work correctly.

```typescript
inject(StateService).initialize();
```

The `initialize` method accepts an optional object of type `StateHandlerConfig` that allows configuring advanced storage aspects, such as encryption type and key generation.

```typescript
export interface StateHandlerConfig {
    encryptionType?: EncryptType;   // Encryption algorithm type (e.g., AES)
    nameType?: GenerateType;        // Name generation strategy (AUTO or CUSTOM)
    keyType?: GenerateType;         // Key generation strategy (AUTO or CUSTOM)
    asyncEnabled?: boolean;         // Enable async storage (default: false) 
                                    // (define fisical storage command by job or when on change props)
                                    
    callBackStateName?: () => string; // Callback for custom name (if nameType is CUSTOM)
    callBackStateKey?: () => string;  // Callback for custom key (if keyType is CUSTOM)
}

// Example with custom configuration
inject(StateService).initialize({
    encryptionType: AsorStorage.StateConst.EncryptType.AES,
    keyType: AsorStorage.StateConst.Generate.AUTO
});
```

### Key and Name Generation (`AUTO` vs `CUSTOM`) <a id="storage-keys-generation"></a>

The `nameType` and `keyType` properties determine how names and keys for storage are generated.

- **AUTO**: The system automatically generates the value. It is the recommended option for most standard cases.
- **CUSTOM**: Delegates value generation to a custom callback function (`callBackStateName` or `callBackStateKey`). This option is useful when specific control over naming or cryptographic key generation is needed, for example, to integrate with existing systems or specific security policies.
