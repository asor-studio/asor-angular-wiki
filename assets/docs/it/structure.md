# Struttura del Progetto e Configurazione

Questa pagina non va letta come un semplice elenco di cartelle. In `asor-core`, la struttura del progetto e` parte dell'architettura: aiuta il team a capire subito dove vive la logica, dove sta la UI riusabile e in che modo una route mette insieme traduzioni, stato e componenti.

L'idea di fondo e` semplice: una codebase ordinata non e` solo piu` bella da vedere, ma diventa anche piu` stabile nel tempo. Per questo ASOR adotta una struttura semantica ispirata ad Atomic Design, affiancata da un routing ricco di metadati.

---

## Scaffolding e Naming Convention <a id="structure-scaffolding"></a>

Quando lavori dentro `src/app`, non stai soltanto creando file Angular: stai dichiarando il ruolo architetturale di ogni blocco.

La suddivisione consigliata e` questa:

| Cartella | Ruolo | Quando usarla |
|---|---|---|
| `pages/` | Viste raggiungibili via route | Quando il blocco rappresenta una destinazione vera dell'app |
| `components/` | Container smart o blocchi di orchestrazione | Quando serve coordinare logica, stato o piu` sezioni senza modellarle come organismo |
| `organisms/` | Sezioni UI ampie e riusabili | Quando una parte di interfaccia compone molecole e/o atomi in una sezione riconoscibile |
| `molecules/` | Blocchi UI funzionali medi | Quando hai un gruppo di elementi che lavorano insieme come unita` |
| `atoms/` | Elementi UI minimi e riusabili | Quando il blocco e` piccolo ma ha identita` autonoma |
| `config/` | Configurazione globale | Quando il file governa route, costanti, cache, stato o bootstrap |

### Struttura delle Cartelle (`src/app`)

```text
src/
└── app/
    ├── pages/
    │   └── page-home/
    │       ├── home.component.ts
    │       └── home.component.html
    ├── components/
    │   └── component-dashboard-container/
    │       └── dashboard-container.component.ts
    ├── organisms/
    │   └── organism-dashboard-shell/
    │       └── dashboard-shell.organism.ts
    ├── molecules/
    │   └── molecule-menu/
    │       └── menu.molecule.ts
    ├── atoms/
    │   └── atom-status-badge/
    │       └── status-badge.atom.ts
    └── config/
        ├── app.config.ts
        └── interfaces/
```

### Convenzioni di nomenclatura

Le convenzioni non sono decorative: servono a rendere immediato il riconoscimento del ruolo di ogni artefatto.

| Tipo | Cartella | File | Classe | Selector |
|---|---|---|---|---|
| Pagina | `page-<name>` | `<name>.component.ts` | `Page<Name>Component` | `asor-<name>` |
| Component | `component-<name>` | `<name>.component.ts` | `<Name>Component` | `asor-<name>` |
| Organism | `organism-<name>` | `<name>.organism.ts` | `<Name>Organism` | `asor-<name>` |
| Molecule | `molecule-<name>` | `<name>.molecule.ts` | `<Name>Molecule` | `asor-<name>` |
| Atom | `atom-<name>` | `<name>.atom.ts` | `<Name>Atom` | `asor-<name>` |

### Components vs Organisms

Questo e` uno dei punti che genera piu` confusione, quindi vale la pena chiarirlo bene.

Un `component` in ASOR non e` semplicemente "un blocco grosso". E` spesso un contenitore smart: coordina flussi, logica di business, relazioni con lo stato o con la route. Può essere visivamente importante, ma il suo valore sta soprattutto nell'orchestrazione.

Un `organism`, invece, e` un blocco di Atomic Design. Rappresenta una sezione UI ampia e riusabile, costruita mettendo insieme molecole e atomi in un pezzo di interfaccia riconoscibile: un header complesso, una sidebar, una dashboard shell, una sezione profilo.

Usa questa regola pratica:

| Se il blocco... | Allora scegli |
|---|---|
| e` una sezione UI riusabile e strutturale | `organism` |
| e` un contenitore smart che coordina logica o piu` blocchi | `component` |
| e` una destinazione di navigazione | `page` |

---

## Configurazione delle Rotte (`IAsorRoute`) <a id="structure-routes"></a>

In ASOR, il file `app.routes.ts` non si limita a collegare un path a un componente. La parte davvero importante e` l'oggetto `data`, perche` e` li` che la route dichiara tutto cio` che serve al runtime:

- traduzioni da caricare
- eventuale controllo di accesso
- dataset da creare
- dataset da collegare
- blocchi UI da registrare

In altre parole, la route diventa il punto di incontro tra navigazione, stato e UI.

### Cosa puo` contenere `IAsorRoute`

| Campo | Significato | Quando serve |
|---|---|---|
| `I18nPath` | Namespace di traduzione da caricare | Sempre, per la pagina corrente |
| `AuthCheck` | Codice di autorizzazione lato backend | Se la route e` protetta |
| `CreateDataSet` | Dataset creato all'ingresso della route | Se la pagina apre uno stato contestuale |
| `ConnectDataSet` | Connessione dataset di pagina | Se la pagina e` storage-aware |
| `Components` | Blocchi `BaseComponent` / `BaseStorageComponent` | Se la route usa componenti registrabili ASOR |
| `Organisms` | Blocchi `BaseOrganism` / `BaseStorageOrganism` | Se la route usa organismi registrabili |
| `Molecules` | Blocchi `BaseMolecule` / `BaseStorageMolecule` | Se la route usa molecole registrabili |
| `Atoms` | Blocchi `BaseAtom` / `BaseStorageAtom` | Se la route usa atomi registrabili |

### Esempio

```typescript
{
    path: WikiConfig.Route.STORAGE,
    component: PageStorageComponent,
    data: {
        I18nPath: [WikiConfig.TranslationUrl.STORAGE],
        AuthCheck: WikiConfig.AuthCheck.STORAGE,
        CreateDataSet: ExampleCreateDataSet,
        Components: [ ... ],
        Organisms: [ ... ],
        Molecules: [ ... ],
        Atoms: [ ... ]
    } as IAsorRoute
}
```

### Regola importante

Ogni classe registrata nella route deve stare nell'array coerente con la sua classe base:

| Classe base | Array route corretto |
|---|---|
| `BaseComponent` / `BaseStorageComponent` | `Components` |
| `BaseOrganism` / `BaseStorageOrganism` | `Organisms` |
| `BaseMolecule` / `BaseStorageMolecule` | `Molecules` |
| `BaseAtom` / `BaseStorageAtom` | `Atoms` |

Registrare un organismo dentro `Components`, o un atom dentro `Molecules`, rende piu` difficile capire il progetto e puo` compromettere il matching corretto del runtime.

---

## Sistema di Autenticazione (`AuthCheck`) <a id="structure-auth"></a>

`AuthCheck` e` la chiave che permette a una route di dichiarare: "prima di entrare qui, verifica che il backend confermi questo permesso".

Dal punto di vista di chi sviluppa la pagina, e` un approccio molto pulito: la route espone l'intenzione, mentre la guard si occupa della parte tecnica.

### Flusso di esecuzione

| Passo | Cosa succede |
|---|---|
| 1. Validazione | La guard controlla se `AuthCheck` e` presente |
| 2. Chiamata API | Viene invocato `auth/flow/{code}` |
| 3. Risposta | Il backend restituisce un oggetto `IAuth` |
| 4. Decisione | La guard consente o blocca la navigazione |

### Contratto atteso dal frontend

| Tipo | Struttura |
|---|---|
| `IAuth` | `{ authorized: AuthStatus }` |
| `AuthStatus.Ok` | Accesso consentito |
| `AuthStatus.NotAllow` | Accesso negato |
| `AuthStatus.Expired` | Sessione scaduta |

### Comportamento della guard

| Stato | Esito |
|---|---|
| `ok` | Entra nella route |
| `na` | Redirect a `ConfigConst.Url.UNAUTHORIZED` |
| `ex` | Redirect a `ConfigConst.Url.LOGIN` |

Esempio di route:

```typescript
{
    path: 'admin-panel',
    component: AdminComponent,
    data: {
        AuthCheck: 'ADMIN_ACCESS_CODE',
    } as IAsorRoute
}
```

---

## Sistema di Traduzioni (`I18nPath`) <a id="structure-i18n"></a>

Le traduzioni in `asor-core` si basano su un principio semplice: ogni route dichiara quali namespace deve caricare, e ogni componente registrato eredita i percorsi che gli servono.

Questo evita due problemi frequenti:

- traduzioni sparse in punti difficili da rintracciare
- componenti che dipendono da file JSON caricati "a mano"

### Struttura del sistema

| Elemento | Ruolo |
|---|---|
| `I18nPath` | Elenca i namespace necessari alla route |
| `TranslationUrl` | Centralizza i percorsi nella config di progetto |
| File JSON | Contengono le chiavi tradotte |
| `TranslatePipe` | Recupera le traduzioni nel template o nel TS |

### Dove vivono i file

I file di traduzione devono stare in:

`public/assets/i18n/<path>/<lang>.json`

oppure, nei progetti che non usano `public`, in:

`src/assets/i18n/<path>/<lang>.json`

### Regola sul contenuto

I file JSON devono essere piatti.

| Corretto | Non supportato |
|---|---|
| `{ "TITLE": "Benvenuto" }` | `{ "page": { "title": "Benvenuto" } }` |

### Utilizzo

| Contesto | Strumento |
|---|---|
| Template HTML | `TranslatePipe` |
| Codice TypeScript | `TranslatePipe.transform()` |

Esempio:

```html
<h1>{{ 'TITLE' | translate }}</h1>
```

```typescript
const msg = this.translate.transform('ERROR_MSG');
```

### Nota importante sui binding verso componenti figli

Quando la traduzione viene usata direttamente nel template del componente corrente, `| translate` puo` ricavare il namespace dal contesto ASOR associato alla view.

Nei binding padre-figlio, invece, questo contesto non e` sempre sufficiente. Un caso tipico e` il passaggio di una label gia tradotta a un componente figlio:

```html
<app-settings-color-swatch
    [label]="'SETTINGS_APPEARANCE_PANEL.COLOR_1' | translate:I18nPath"
    [previewColor]="'#f5dece'"
    [active]="true"
></app-settings-color-swatch>
```

In questo scenario e` consigliato passare esplicitamente `I18nPath` alla pipe. In questo modo la traduzione resta agganciata al namespace del padre e non dipende dal contesto runtime con cui Angular risolve il binding del figlio.

---

## Sistema UI: Components, Organisms, Molecules e Atoms <a id="structure-components"></a>

ASOR usa Atomic Design ma lascia spazio anche a blocchi smart di alto livello. Per questo conviene pensare ai quattro livelli UI come a una gerarchia con ruoli diversi.

| Livello | Classe base | Scopo |
|---|---|---|
| Component | `BaseComponent` | Container smart, pagina, coordinamento di logica e lifecycle |
| Organism | `BaseOrganism` | Sezione UI ampia e riusabile |
| Molecule | `BaseMolecule` | Blocco funzionale medio |
| Atom | `BaseAtom` | Elemento UI minimo riusabile |

Le versioni storage-aware seguono la stessa logica:

| Livello | Variante storage-aware |
|---|---|
| Component | `BaseStorageComponent<T>` |
| Organism | `BaseStorageOrganism<T>` |
| Molecule | `BaseStorageMolecule<T>` |
| Atom | `BaseStorageAtom<T>` |

### `className`: perche` e` obbligatorio

Ogni classe registrata deve dichiarare:

```typescript
public static override readonly className = 'MyClassName';
```

Questo valore serve per due motivi fondamentali:

| Motivo | Perche` conta |
|---|---|
| Matching di route | Il runtime trova la configurazione giusta confrontando il `className` |
| Logging | I log restano leggibili anche in build minificate |

### Lifecycle

Ogni livello ha hook coerenti con il proprio ruolo:

| Classe base | Hook principali |
|---|---|
| `BaseComponent` | `baseCompViewEnter`, `baseCompViewLeave` |
| `BaseOrganism` | `baseOrganismViewWillEnter`, `baseOrganismViewWillLeave` |
| `BaseMolecule` | `baseMoleculeViewWillEnter`, `baseMoleculeViewWillLeave` |
| `BaseAtom` | `baseAtomViewWillEnter`, `baseAtomViewWillLeave` |

### Gestione automatica dei figli con `#ChildRef`

`BaseComponent` puo` propagare automaticamente il proprio lifecycle alle molecole figlie. Questo rende i template piu` puliti e riduce il bisogno di `ViewChild` manuali.

Esempio:

```html
<asor-header #ChildRef></asor-header>
<asor-sidebar #ChildRef></asor-sidebar>
```

Quando il componente padre entra o esce, tutte le molecole marcate con `#ChildRef` ricevono automaticamente il relativo hook.

---

## Sistema di Cache <a id="structure-cache"></a>

La cache in `asor-core` non e` un accessorio: e` uno strumento di prestazione e di stabilita` del traffico HTTP.

Il punto chiave e` `ConfigCache`, cioe` la configurazione centralizzata che dice al runtime:

- cosa tenere in cache
- per quanto tempo
- in quale storage
- quando invalidare una voce
- se la route corrente deve influenzare il caching

### Come leggere una regola di cache

| Proprietà | Significato pratico |
|---|---|
| `pathValue` | Quale endpoint o pattern URL intercettare |
| `persistenceType` | Dove e per quanto tempo tenere la cache |
| `encriptType` | Se e come cifrare i dati cacheati |
| `reqPayloadCache` | Se il body deve entrare nella chiave di cache |
| `resctrictRoute` | Se la cache vale solo dentro una route specifica |
| `clearOn` | Quali chiamate invalidano quella cache |
| `callBackCacheKey` | Chiave personalizzata per casi avanzati |

### Tipi di persistenza

| Tipo | Comportamento |
|---|---|
| `VOLATILE` | Cache in memoria, persa a refresh o chiusura tab |
| `SESSION` | Sopravvive al refresh ma non oltre la sessione |
| `LOCAL` | Resta disponibile anche tra sessioni diverse |

### Perche` il file cache e` importante

Un buon `app-cache.config.ts` aiuta a:

- ridurre richieste ripetute
- velocizzare il caricamento di asset frequenti come le traduzioni
- controllare l'invalidazione dopo operazioni di scrittura
- evitare logiche cache improvvisate dentro i service

### Esempio

```typescript
export class WikiCacheConfig extends ConfigCache {
    public static override globalStateName = 'wiki';

    protected static override _pathsExtensions = [
        {
            persistenceType: AsorGlobalEnum.CacheType.SESSION,
            encriptType: AsorGlobalEnum.CacheEncriptType.AUTO,
            reqPayloadCache: false,
            resctrictRoute: ConfigConst.Route.NONE,
            pathValue: '/api/v1/users',
            clearOn: ['/api/v1/users/save', '/api/v1/users/delete']
        },
    ];

    static {
        ConfigCache.SetConfiguration(WikiCacheConfig);
    }
}
```

---

## Sistema di Notifica di Errore <a id="structure-error"></a>

Il sistema errori di ASOR permette di evitare `catchError` distribuiti in ogni servizio. L'idea e` centralizzare l'intercettazione e lasciare ai componenti solo la reazione.

### Flusso

| Fase | Cosa accade |
|---|---|
| Intercettazione | `ErrorInterceptor` cattura l'errore HTTP |
| Notifica | L'errore viene inoltrato a `NotifyErrorService` |
| Reazione | I componenti registrati ricevono lo status e il payload errore |

### Registrazione nei componenti

Per usare il sistema, il componente si registra di solito in `baseCompViewEnter()` usando un identificatore unico, in genere il proprio `className`.

| Punto chiave | Motivo |
|---|---|
| ID univoco | Evita sottoscrizioni duplicate |
| Registrazione nel lifecycle | Tiene il componente allineato alla sua presenza reale in pagina |
| Gestione centralizzata | Riduce boilerplate nei service |

Esempio:

```typescript
this.notifyErrorService.registry(
    this.constructor.className,
    (status, error) => {
        this.handleError(status, error);
    }
);
```
