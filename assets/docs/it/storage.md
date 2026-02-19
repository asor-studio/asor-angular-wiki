# Gestione dello Storage (DataSet)

In questa sezione viene spiegato come configurare, applicare e inizializzare lo storage (DataSet) all'interno dell'applicazione utilizzando `ICreateDataSet` e `IConnectDataSet`.

## 1. Definizione della Configurazione (`ICreateDataSet`) <a id="storage-definition"></a>

Per definire un nuovo storage (DataSet), è necessario creare un oggetto che rispetti l'interfaccia `ICreateDataSet`. Questo oggetto definisce il nome del contenitore, i dati iniziali e le opzioni di configurazione.

### Esempio (`example.storage.const.ts`)

```typescript
import { AsorStorage, ICreateDataSet } from '@asor-studio/asor-core';

export const ExampleCreateDataSet: ICreateDataSet = {
    name: 'example-component', // Nome del contenitore
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

### Dettagli Opzioni di Configurazione <a id="storage-config-details"></a>

La proprietà `option` nell'oggetto `ICreateDataSet` permette di configurare il comportamento dello storage per questo specifico dataset.

#### Tipologie di Storage (`storeType`)
Esistono 3 tipologie di storage disponibili:
1. **VOLATILE**: I dati vengono mantenuti solo in memoria. Vengono persi al refresh della pagina. È l'opzione di default e la più performante.
2. **SESSION**: I dati vengono salvati nel `SessionStorage` del browser. Sopravvivono al refresh della pagina ma vengono persi alla chiusura del tab o del browser.
3. **LOCAL**: I dati vengono salvati nel `LocalStorage` del browser. Persistono anche dopo la chiusura e riapertura del browser.

#### Criptazione (`encrypt`)
Impostando `encrypt: true`, il contenuto del dataset viene criptato prima di essere salvato nello storage (utile per Session e Local storage) per garantire una maggiore sicurezza dei dati sensibili.

#### Immutabilità (`freeze`)
Impostando `freeze: true`, il contenuto iniziale del dataset viene reso immutabile ("congelato") al momento della creazione. Questo è utile per prevenire modifiche accidentali ai dati di configurazione o costanti, assicurando che lo stato rimanga consistente.

---

## 2. Connessione del DataSet (`IConnectDataSet`) <a id="storage-connection"></a>

Per collegare il contenitore a un componente o a una molecola ed erogare i dati nelle loro proprietà (`props`), si utilizza un oggetto `IConnectDataSet`.

Questo oggetto definisce i selettori che mappano le proprietà dello storage alle proprietà del componente. Quando i dati nello storage cambiano, le proprietà collegate vengono aggiornate automaticamente.

### Esempio (`example.storage.const.ts`)

```typescript
import { IConnectDataSet } from '@asor-studio/asor-core';

export const ExampleConnectDataSet: IConnectDataSet = {
    selectors: {
        title: 'example-component.title', // Mappa 'title' del componente a 'example-component.title' dello storage
        form: 'example-component.form'    // Mappa 'form' del componente a 'example-component.form' dello storage
    }
};
```

---

## 3. Applicazione della Configurazione alle Rotte <a id="storage-application"></a>

Associare le configurazioni di storage (`CreateDataSet`) e le connessioni (`ConnectDataSet`) alle rotte che necessitano di accedervi. Questo avviene nel `data` della rotta.

### Spiegazione Configurazione
Come si collega alla rotta:
- **CreateDataSet**: Crea il contenitore con i dati.
- **ConnectDataSet**: Connette il contenitore alla molecola o al componente erogando nella variabile `props` i dati del contenitore.
- **Components / Molecules**: Definisce i selettori del contenitore a cui poi è legato l'evento di aggiornamento.

### Esempio (`app.routes.ts`)

```typescript
{
    path: WikiConfig.Route.STORAGE,
    component: PageStorageComponent,
    data: {
        // ... altre configurazioni (I18nPath, AuthCheck, ecc.)
        
        // Crea il contenitore con i dati
        CreateDataSet: ExampleCreateDataSet,
        
        Components: [
            {
                Component: ExampleStorageComponent,
                I18nPath: [WikiConfig.TranslationUrl.STORAGE],
                // Connette il contenitore a questo componente
                ConnectDataSet: ExampleConnectDataSet
            }
        ],
        Molecules: [
            {
                Molecule: ExampleStorageMolecule,
                I18nPath: [WikiConfig.TranslationUrl.STORAGE],
                // Connette il contenitore a questa molecola
                ConnectDataSet: ExampleConnectDataSet
            }
        ],
    } as IAsorRoute
},
```

In questo modo, quando si naviga sulla rotta `STORAGE`:
1.  Viene creato il DataSet `example-component` con i dati iniziali.
2.  `ExampleStorageComponent` e `ExampleStorageMolecule` ricevono automaticamente i dati aggiornati tramite `ExampleConnectDataSet`.

> **Importante**: Affinché il matching delle rotte funzioni correttamente, ogni componente e molecola elencato in `Components` o `Molecules` **deve** dichiarare una proprietà `public static override readonly className` (vedi sezione successiva).

---

## 4. Identificazione del Componente (`className`) <a id="storage-classname"></a>

Ogni classe che estende `BaseComponent`, `BaseMolecule`, `BaseStorageComponent` o `BaseStorageMolecule` **deve** dichiarare una proprietà statica `className`:

```typescript
public static override readonly className: string = 'NomeDellaClasse';
```

Questa proprietà viene utilizzata dal framework per:
- **Associare i componenti alla configurazione delle rotte**: Quando una rotta viene attivata, il sistema confronta `Component.className` (o `Molecule.className`) dai dati della rotta con la classe in fase di istanziazione per erogare il corretto `ConnectDataSet` e `I18nPath`.
- **Identificare le classi nei log**: `ConsoleLogsUtility` utilizza `className` come prefisso nei messaggi di log, garantendo log leggibili anche in build di produzione dove i nomi delle classi JavaScript vengono minificati.

### Perché `static`?
La proprietà deve essere **static** perché deve essere disponibile sul costruttore della classe stessa (non su un'istanza), assicurando che possa essere letta durante il matching delle rotte *prima* che il componente sia completamente costruito. L'interfaccia `ICompStatic` lo impone a livello di tipi:

```typescript
export interface ICompStatic {
    readonly className: string;
}
```

Le interfacce `IComponent` e `IMolecule` richiedono rispettivamente `Type<BaseComponent> & ICompStatic` e `Type<BaseMolecule> & ICompStatic`, quindi TypeScript genererà un errore di compilazione se una classe utilizzata nella configurazione delle rotte non dichiara il `className` statico.

---

## 5. Inizializzazione del Service <a id="storage-init"></a>

Il `StateService` rappresenta il core del sistema di gestione dello stato. Esso deve essere inizializzato, solitamente nel componente root dell'applicazione (`src/app/app.root.ts`), per poter funzionare correttamente.

```typescript
inject(StateService).initialize();
```

Il metodo `initialize` accetta un oggetto opzionale di tipo `StateHandlerConfig` che permette di configurare aspetti avanzati dello storage, come il tipo di crittografia e la generazione delle chiavi.

```typescript
export interface StateHandlerConfig {
    encryptionType?: EncryptType;   // Encryption algorithm type (e.g., AES)
    nameType?: GenerateType;        // Name generation strategy (AUTO or CUSTOM)
    keyType?: GenerateType;         // Key generation strategy (AUTO or CUSTOM)
    callBackStateName?: () => string; // Callback for custom name (if nameType is CUSTOM)
    callBackStateKey?: () => string;  // Callback for custom key (if keyType is CUSTOM)
}

// Example with custom configuration
inject(StateService).initialize({
    encryptionType: AsorStorage.StateConst.EncryptType.AES,
    keyType: AsorStorage.StateConst.Generate.AUTO
});
```

### Generazione Chiavi e Nomi (`AUTO` vs `CUSTOM`) <a id="storage-keys-generation"></a>

Le proprietà `nameType` e `keyType` determinano come vengono generati i nomi e le chiavi per lo storage.

- **AUTO**: Il sistema genera automaticamente il valore. È l'opzione consigliata per la maggior parte dei casi standard.
- **CUSTOM**: Delega la generazione del valore a una funzione di callback personalizzata (`callBackStateName` o `callBackStateKey`). Questa opzione è utile quando si ha bisogno di un controllo specifico sulla nomenclatura o sulla generazione delle chiavi crittografiche, ad esempio per integrarsi con sistemi esistenti o policy di sicurezza specifiche.
