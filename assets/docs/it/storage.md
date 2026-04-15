# Gestione dello Storage (DataSet)

Lo storage in `asor-core` non e` un semplice contenitore di dati. E` il meccanismo che permette a una pagina, una molecola, un organismo o un atom storage-aware di leggere e aggiornare uno stato condiviso senza dover riscrivere continuamente la stessa logica di sincronizzazione.

Questa pagina prova a leggerlo in modo pratico: prima capiamo i concetti, poi vediamo come si collegano tra loro.

---

## 1. Definizione del DataSet (`ICreateDataSet`) <a id="storage-definition"></a>

Un dataset descrive tre cose:

- come si chiama il contenitore
- quali dati iniziali contiene
- come deve essere conservato

In ASOR il dataset e` dichiarativo: non si costruisce "a mano" in giro per l'app, ma si definisce una volta e poi lo si registra nel punto corretto.

### Struttura essenziale

| Campo | Significato |
|---|---|
| `name` | Nome logico del contenitore |
| `data` | Oggetto iniziale del dataset |
| `option` | Regole di persistenza e protezione |

### Esempio

```typescript
import { AsorStorage, ICreateDataSet } from '@asor-studio/asor-core';

export const ExampleCreateDataSet: ICreateDataSet = {
    name: 'example-component',
    data: {
        title: 'Hi {name}, you have {age} years.',
        form: {
            name: 'John',
            age: 30,
            jobRole: [
                { title: 'Developer', description: 'Developer', selected: false },
                { title: 'Designer', description: 'Designer', selected: false }
            ]
        }
    } as IExampleProps,
    option: {
        storeType: AsorStorage.StateConst.StoreType.VOLATILE,
        encrypt: true,
        freeze: false
    }
};
```

### Dettagli delle opzioni di configurazione <a id="storage-config-details"></a>

| Opzione | Significato pratico |
|---|---|
| `storeType` | Decide dove e per quanto tempo vivono i dati |
| `encrypt` | Protegge il contenuto prima della persistenza |
| `freeze` | Rende immutabile il valore iniziale |

### Tipi di storage

| Tipo | Quando usarlo |
|---|---|
| `VOLATILE` | Quando i dati possono essere persi al refresh e vuoi la soluzione piu` leggera |
| `SESSION` | Quando i dati devono sopravvivere al reload ma non oltre la sessione del browser |
| `LOCAL` | Quando i dati devono restare disponibili anche dopo chiusura e riapertura |

---

## 2. Connessione del DataSet (`IConnectDataSet`) <a id="storage-connection"></a>

Definire un dataset non basta. Bisogna anche dire come i suoi valori vengono esposti dentro i blocchi UI.

Questo compito spetta a `IConnectDataSet`, che crea una mappa tra i path dello storage e le proprieta` che il componente vedra` dentro `props`.

### Struttura

| Campo | Ruolo |
|---|---|
| `name` | Nome logico della connessione |
| `selectors` | Mappa tra path del dataset e props del blocco |

### Esempio

```typescript
import { IConnectDataSet } from '@asor-studio/asor-core';

export const ExampleConnectDataSet: IConnectDataSet = {
    name: 'ExampleConnection',
    selectors: {
        title: 'example-component.title',
        form: 'example-component.form'
    }
};
```

### Come leggerlo

| Selector | Significa |
|---|---|
| `title: 'example-component.title'` | `props.title` leggerebbe `example-component.title` |
| `form: 'example-component.form'` | `props.form` leggerebbe `example-component.form` |

---

## 3. Come si collega alla route <a id="storage-application"></a>

In ASOR e` la route a mettere insieme i pezzi. La pagina dichiara:

- se deve creare un dataset
- se deve connettersi a un dataset
- quali blocchi UI useranno quella connessione

Questo rende il comportamento della pagina leggibile direttamente nel routing.

### Esempio

```typescript
{
    path: WikiConfig.Route.STORAGE,
    component: PageStorageComponent,
    data: {
        I18nPath: [WikiConfig.TranslationUrl.STORAGE],
        CreateDataSet: ExampleCreateDataSet,
        Components: [
            {
                Component: ExampleStorageComponent,
                I18nPath: [WikiConfig.TranslationUrl.STORAGE],
                ConnectDataSet: ExampleConnectDataSet
            }
        ],
        Molecules: [
            {
                Molecule: ExampleStorageMolecule,
                I18nPath: [WikiConfig.TranslationUrl.STORAGE],
                ConnectDataSet: ExampleConnectDataSet
            }
        ],
    } as IAsorRoute
}
```

### Cosa succede in pratica

| Passo | Effetto |
|---|---|
| 1. Entri nella route | La pagina viene attivata |
| 2. `CreateDataSet` viene applicato | Il container viene creato |
| 3. `ConnectDataSet` viene letto | I blocchi UI ricevono le props collegate |
| 4. Il dataset cambia | Le `props` si aggiornano in modo reattivo |

### Nota importante

Il matching della route si basa sul `className` statico delle classi registrate. Se la classe non espone questo valore, la route non potra` agganciare correttamente dataset e traduzioni.

---

## 4. `className`: perche` e` obbligatorio <a id="storage-classname"></a>

Ogni classe che estende una delle classi base ASOR registrabili deve dichiarare:

```typescript
public static override readonly className: string = 'NomeDellaClasse';
```

### Perche` serve

| Motivo | Effetto |
|---|---|
| Matching con la route | Il runtime riconosce quale configurazione applicare |
| Logging | I log restano leggibili anche in produzione |
| Tipizzazione | TypeScript verifica la compatibilita` con `ICompStatic` |

### A quali classi si applica

| Livello | Versione base | Versione storage-aware |
|---|---|---|
| Component | `BaseComponent` | `BaseStorageComponent` |
| Organism | `BaseOrganism` | `BaseStorageOrganism` |
| Molecule | `BaseMolecule` | `BaseStorageMolecule` |
| Atom | `BaseAtom` | `BaseStorageAtom` |

---

## 5. Inizializzazione di `StateService` <a id="storage-init"></a>

`StateService` e` il motore del sistema di stato. Se non viene inizializzato correttamente, i blocchi storage-aware non hanno una base affidabile su cui lavorare.

La regola pratica e` questa: inizializzalo una sola volta nel bootstrap applicativo.

### Esempio minimo

```typescript
inject(StateService).initialize();
```

### Configurazione avanzata

| Campo | Significato |
|---|---|
| `globalStateName` | Namespace dello storage applicativo |
| `encryptionType` | Algoritmo usato per la cifratura |
| `nameType` | Strategia di generazione nome |
| `keyType` | Strategia di generazione chiave |
| `asyncEnabled` | Modalita` di sincronizzazione e persistenza |
| `callBackStateName` | Callback per naming custom |
| `callBackStateKey` | Callback per key generation custom |

### Esempio

```typescript
inject(StateService).initialize({
    globalStateName: 'wiki',
    encryptionType: AsorStorage.StateConst.EncryptType.AES,
    nameType: AsorStorage.StateConst.Generate.AUTO,
    keyType: AsorStorage.StateConst.Generate.AUTO,
    asyncEnabled: true,
});
```

### AUTO vs CUSTOM

| Modalita` | Quando usarla |
|---|---|
| `AUTO` | Quasi sempre, quando vuoi un comportamento standard e solido |
| `CUSTOM` | Quando devi allinearti a convenzioni esterne o requisiti di sicurezza particolari |

### Generazione di nomi e chiavi <a id="storage-keys-generation"></a>

Se il progetto usa la modalita` `AUTO`, il framework gestisce naming e key generation in modo trasparente.
Se usi `CUSTOM`, allora stai prendendo una decisione architetturale e devi fornire callback coerenti per mantenere stabile il comportamento dello storage.

---

## 6. Esempio Molecola storage-aware <a id="storage-mol-intro"></a>

Se vuoi vedere lo storage "in azione", la forma piu` didattica e` osservare una molecola che legge e scrive direttamente dentro `props`.

Le tre parti da guardare sono:

| Punto | Perche` e` utile |
|---|---|
| HTML | Mostra il binding diretto verso `props` |
| TypeScript | Mostra quanto poco boilerplate serve |
| Inheritance | Spiega perche` `BaseStorageMolecule<T>` fa il grosso del lavoro |

### HTML della Molecola <a id="storage-mol-html"></a>

Per l'esempio completo e il markup reale, consulta la pagina dedicata all'esempio molecule storage.

### TypeScript della Molecola <a id="storage-mol-ts"></a>

Per l'esempio completo e la logica reale, consulta la pagina dedicata all'esempio molecule storage.

### Ereditarieta` della Molecola <a id="storage-mol-inheritance"></a>

Il punto chiave e` che `BaseStorageMolecule<T>` espone `props` in modo tipizzato e reattivo, quindi la molecola puo` concentrarsi sulla UI invece che sulla gestione manuale della sincronizzazione.

---

## 7. Esempio Component storage-aware <a id="storage-comp-intro"></a>

Un component storage-aware e` utile quando vuoi visualizzare, derivare o orchestrare dati di stato a un livello piu` alto rispetto alla molecola.

Anche qui conviene leggere l'esempio in tre parti:

| Punto | Perche` e` utile |
|---|---|
| HTML | Mostra come presentare i dati provenienti dallo stato |
| TypeScript | Mostra getter e logica derivata |
| Inheritance | Spiega il ruolo di `BaseStorageComponent<T>` |

### HTML del Component <a id="storage-comp-html"></a>

Per l'esempio completo e il template reale, consulta la pagina dedicata all'esempio component storage.

### TypeScript del Component <a id="storage-comp-ts"></a>

Per l'esempio completo e la logica reale, consulta la pagina dedicata all'esempio component storage.

### Ereditarieta` del Component <a id="storage-comp-inheritance"></a>

Il valore di `BaseStorageComponent<T>` sta nel fatto che collega route, dataset e lifecycle della pagina, lasciando al component solo la responsabilita` di coordinare la UI.
