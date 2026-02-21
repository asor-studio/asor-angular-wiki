# Struttura del Progetto e Configurazione

In questa sezione analizzeremo come costruire e configurare un progetto che utilizza la libreria `asor-core`. Partiremo dall'analisi della struttura delle cartelle e delle convenzioni di nomenclatura.


## Scaffolding e Naming Convention <a id="structure-scaffolding"></a>

La struttura del progetto `src/app` è organizzata semanticamente per favorire scalabilità e manutenibilità, seguendo rigorose convenzioni di nomenclatura per distinguere chiaramente i ruoli dei vari artefatti (Pagine, Componenti, Molecole).

### Struttura delle Cartelle (`src/app`)

La root dell'applicazione è suddivisa nelle seguenti macro-aree:

*   📂 **`pages/`**: Contiene le pagine dell'applicazione (viste intere raggiungibili via routing).
*   📂 **`components/`**: Contiene i componenti "smart" o organismi complessi riutilizzabili.
*   📂 **`molecules/`**: Contiene le molecole, ovvero componenti UI riutilizzabili e funzionali.
*   📂 **`config/`**: Contiene i file di configurazione globali (rotte, costanti, cache).

```text
src/
└── app/
    ├── pages/                     # Container delle pagine (Viste)
    │   ├── page-home/
    │   │   ├── home.component.ts  # Logica della pagina
    │   │   ├── home.component.html
    │   │   └── ...
    │   └── ...
    ├── components/                # Componenti Smart / Organismi
    │   ├── component-placeholder/
    │   │   ├── placeholder.component.ts
    │   │   └── ...
    │   └── ...
    ├── molecules/                 # Componenti UI Riutilizzabili
    │   ├── molecule-menu/
    │   │   ├── menu.molecule.ts   # Nota il suffisso .molecule.ts
    │   │   └── ...
    │   └── ...
    └── config/                    # Configurazioni Globali
        ├── wiki.config.ts
        └── ...
```

### Convenzioni di Nomenclatura

Per garantire coerenza e immediata riconoscibilità, ogni tipo di artefatto segue uno specifico pattern di naming per cartelle, file e classi.

#### 1. Pagine (`pages/`)

Le pagine rappresentano le viste principali e fungono da contenitori per componenti e molecole.

*   **Cartella**: Prefisso `page-` (es. `page-storage`, `page-home`).
*   **File**: `<name>.component.ts` (es. `storage.component.ts`).
*   **Classe**: Prefisso `Page` + PascalCase (es. `PageStorageComponent`).
*   **Selettore**: Prefisso `asor-` (es. `asor-storage`).

#### 2. Componenti (`components/`)

I componenti sono blocchi costruttivi complessi, spesso contenenti logica di business o coordinando più molecole.

*   **Cartella**: Prefisso `component-` (es. `component-placeholder`).
*   **File**: `<name>.component.ts` (es. `placeholder.component.ts`).
*   **Classe**: PascalCase + Suffix `Component` (es. `PlaceholderComponent`).
*   **Selettore**: Prefisso `asor-` (es. `asor-placeholder`).

#### 3. Molecole (`molecules/`)

Le molecole sono unità funzionali riutilizzabili (es. card, menu, form).

*   **Cartella**: Prefisso `molecule-` (es. `molecule-menu`).
*   **File**: `<name>.molecule.ts` (Nota il suffisso `.molecule.ts`).
*   **Classe**: PascalCase + Suffix `Molecule` (es. `MenuMolecule`).
*   **Selettore**: Prefisso `asor-` (es. `asor-menu`).

---

## Configurazione delle Rotte (`IAsorRoute`) <a id="structure-routes"></a>


Nel file `src/app/app.routes.ts`, le rotte vengono definite utilizzando l'interfaccia `IAsorRoute`. La parte più importante di questa configurazione è l'oggetto `data`, che permette di passare informazioni essenziali per l'inizializzazione dei componenti, la gestione dello stato e l'internazionalizzazione.

Ecco un esempio di configurazione per la rotta `STORAGE`:

```typescript
// src/app/app.routes.ts
{
    path: WikiConfig.Route.STORAGE,
    component: PageStorageComponent,
    data: {
        I18nPath: [WikiConfig.TranslationUrl.STORAGE],
        AuthCheck: WikiConfig.AuthCheck.STORAGE,
        CreateDataSet: ExampleCreateDataSet,
        Components: [ ... ],
        Molecules: [ ... ]
    } as IAsorRoute
},
```

---

## Sistema di Autenticazione (`AuthCheck`) <a id="structure-auth"></a>

La proprietà `AuthCheck` è un parametro opzionale dell'interfaccia `IAsorRoute` che abilita una verifica di sicurezza lato server prima di consentire l'accesso alla rotta.

```typescript
// Esempio configurazione rotta
{
    path: 'admin-panel',
    component: AdminComponent,
    data: {
        AuthCheck: 'ADMIN_ACCESS_CODE', // Codice univoco di controllo
    } as IAsorRoute
}
```

### Flusso di Esecuzione
Quando il Router Angular tenta di navigare verso una rotta protetta da `AuthCheck`, la `AuthGuard` intercetta la navigazione ed esegue i seguenti passaggi:

1.  **Validazione**: Verifica se la proprietà `AuthCheck` è valorizzata.
2.  **Chiamata API**: Esegue una richiesta HTTP GET all'endpoint configurato nel backend, appendendo il codice di autorizzazione.
    *   Endpoint: `auth/flow/{code}`
    *   Esempio URL: `https://api.example.com/api/v1/auth/flow/ADMIN_ACCESS_CODE`
3.  **Analisi Risposta**: Attende un oggetto JSON conforme all'interfaccia `IAuth`.
4.  **Decisione**:
    *   Se l'utente è autorizzato (`ok`), la navigazione procede.
    *   Se non autorizzato o sessione scaduta, viene eseguito un reindirizzamento.

### Interfacce e Definizioni

Il frontend si aspetta una risposta strutturata secondo le seguenti definizioni (presenti in `asor-core`):

**Interfaccia Risposta (`IAuth`)**
```typescript
export interface IAuth {
    /** Stato dell'autorizzazione */
    authorized: AuthStatus;
}
```

**Enum Stati Autorizzazione (`AuthStatus`)**
```typescript
export enum AuthStatus {
    /** Autorizzazione concessa */
    Ok = 'ok',
    
    /** Autorizzazione negata (utente loggato ma senza permessi) */
    NotAllow = 'na',
    
    /** Sessione scaduta o token non valido */
    Expired = 'ex'
}
```

### Comportamento della Guard (`AuthGuard`)

La `AuthGuard` gestisce i vari stati di risposta applicando logiche di reindirizzamento specifiche:

*   **`AuthStatus.Ok` (`'ok'`)**: `return true` (Navigazione consentita).
*   **`AuthStatus.NotAllow` (`'na'`)**: `return false` -> Redirect a **Non Autorizzato** (`ConfigConst.Url.UNAUTHORIZED`).
*   **`AuthStatus.Expired` (`'ex'`)**: `return false` -> Redirect a **Login** (`ConfigConst.Url.LOGIN`).

### Esempio Implementazione Backend (Node.js / Express)

```typescript
// Esempio endpoint Express.js
app.get('/api/v1/auth/flow/:code', (req, res) => {
    const authCode = req.params.code; // es. 'ADMIN_ACCESS_CODE'
    const userSession = req.session.user; 

    // 1. Verifica Sessione
    if (!userSession || isSessionExpired(userSession)) {
        return res.json({ authorized: 'ex' });
    }

    // 2. Verifica Permessi
    const hasPermission = checkUserPermission(userSession, authCode);
    return res.json({ authorized: hasPermission ? 'ok' : 'na' });
});
```

---

## Sistema di Traduzioni (`I18nPath`) <a id="structure-i18n"></a>

Il sistema di traduzioni di `asor-core` è basato sul caricamento dinamico di file JSON in base alla rotta e alla lingua corrente.

### Configurazione (`I18nPath`)

La proprietà `I18nPath` in `IAsorRoute` è un array di stringhe che definisce da quali moduli caricare le traduzioni per la pagina corrente.

```typescript
I18nPath: [WikiConfig.TranslationUrl.STORAGE, WikiConfig.TranslationUrl.SHARED]
```

#### Definizione dei Percorsi
I valori (es. `/asor/storage`) sono definiti nella classe di configurazione del progetto (`WikiConfig`), estendendo `ConfigConst`.

```typescript
// src/app/config/wiki.config.ts
export class WikiConfig extends ConfigConst {
    protected static override _translationUrlExtensions = {
        STORAGE: '/asor/storage', 
        SHARED: '/asor/shared',
    };
    // ...
}
```

### Struttura e Contenuto dei File

I file di traduzione devono trovarsi in `public/assets/i18n/<path>/[lang].json`.

*   **Path**: Corrisponde al valore definito in `WikiConfig` (es. `/asor/storage`).
*   **Lang**: Codice lingua (es. `it`, `en`).
*   **Contenuto**: **Mappa piatta** (Flat Key-Value). Oggetti annidati **NON** sono supportati.

Esempio `public/assets/i18n/asor/storage/it.json`:
```json
{
    "TITLE": "Benvenuto",
    "SUBTITLE": "Gestione Storage",
    "BUTTON_SAVE": "Salva"
}
```

### Configurazione della Lingua

La lingua di default (`DefaultLanguage`) è definita in `ConfigConst` (default: 'it'). Per modificarla, fare override in `WikiConfig`:

```typescript
// src/app/config/wiki.config.ts
export class WikiConfig extends ConfigConst {
    protected static override _translationUrlExtensions = {
        DefaultLanguage: 'en', // Override
        // ...
    };

    static override get TranslationUrl() {
        return { ...super.TranslationUrl, ...this._translationUrlExtensions };
    }
}
```

### Utilizzo delle Traduzioni

#### 1. Nei Template HTML (`TranslatePipe`)
Importare `TranslatePipe` nel componente standalone e usare la pipe `| translate`.

```html
<h1>{{ 'TITLE' | translate }}</h1>
<p>{{ 'WELCOME' | translate : params }}</p> <!-- params = { name: 'Mario' } -->
```

#### 2. Nel Codice TypeScript
Iniettare `TranslatePipe` e usare il metodo `transform()`.

```typescript
import { TranslatePipe } from '@asor-studio/asor-core';

@Component({ ... })
export class MyComponent {
    private translate = inject(TranslatePipe);

    showAlert() {
        const msg = this.translate.transform('ERROR_MSG');
        alert(msg);
    }
}
```

---

## Sistema Componenti e Molecole <a id="structure-components"></a>

In `asor-core`, l'architettura UI segue il pattern **Atomic Design**, distinguendo chiaramente tra **Components** (spesso corrispondenti a Organismi o Intere Pagine) e **Molecules** (componenti riutilizzabili più piccoli).

Tutti i componenti e le molecole utilizzati in una pagina **DEVONO** essere registrati nella configurazione della rotta per funzionare correttamente (es. per il caricamento delle traduzioni o la gestione dello stato).

### Differenze Concettuali

#### 1. Components (`BaseComponent`)
Rappresentano i blocchi di alto livello dell'applicazione, come le Pagine (`PageStorageComponent`) o macro-sezioni complesse (`ExampleStorageComponent`).
*   **Ruolo**: Gestire la logica di business principale, l'inizializzazione della pagina e coordinare le molecole.
*   **Classe Base**: Estendono `BaseComponent`.
*   **Lifecycle**: Gestiscono automaticamente i ciclo di vita Angular (`ngOnInit`, `ngOnDestroy`) e Ionic (`ionViewWillEnter`, `ionViewWillLeave`).

#### 2. Molecules (`BaseMolecule`)
Rappresentano componenti UI funzionali e riutilizzabili (es. una card utente, un form di ricerca, una finestra di dialogo).
*   **Ruolo**: Fornire funzionalità specifiche e isolate. Sono spesso contenute all'interno di un Component.
*   **Classe Base**: Estendono `BaseMolecule`.
*   **Lifecycle**: Ereditano il ciclo di vita dal Component padre (se collegati correttamente tramite `@ViewChild`).

### Configurazione (`Components` e `Molecules`)

Nel file `app.routes.ts`, ogni rotta deve dichiarare quali Componenti e Molecole utilizza nell'oggetto `data`.

```typescript
// src/app/app.routes.ts
data: {
    // ...
    Components: [
        {
            Component: ExampleStorageComponent, // Classe del componente
            I18nPath: [WikiConfig.TranslationUrl.STORAGE], // Traduzioni specifiche
            ConnectDataSet: ExampleConnectDataSet // Connessione stato (opzionale)
        }
    ],
    Molecules: [
        {
            Molecule: ExampleStorageMolecule, // Classe della molecola
            I18nPath: [WikiConfig.TranslationUrl.STORAGE], // Traduzioni specifiche
            ConnectDataSet: ExampleConnectDataSet // Connessione stato (opzionale)
        }
    ]
} as IAsorRoute
```

### Implementazione nelle Classi

Per beneficiare di queste funzionalità, le classi devono estendere le rispettive classi base.

#### Esempio Componente
```typescript
import { BaseComponent } from '@asor-studio/asor-core';

@Component({ ... })
export class MyPage extends BaseComponent {
    // I18nPath viene popolato automaticamente dal routing
    
    // Implementazione Obbligatoria
    override baseCompViewEnter() { }
    override baseCompViewLeave() { }
}
```

#### Esempio Molecola
```typescript
import { BaseMolecule } from '@asor-studio/asor-core';

@Component({ ... })
export class MyCard extends BaseMolecule {
    // Recupera la configurazione specifica per 'MyCard' dal routing
    
    // Implementazione Obbligatoria
    override baseCompViewEnter() { }
    override baseCompViewLeave() { }
}
```

> **Nota Fondamentale**: Se un Componente o una Molecola non viene registrato nell'array `Components` o `Molecules` della rotta, `asor-core` non sarà in grado di iniettare le configurazioni corrette (come `I18nPath`), causando errori nel caricamento delle traduzioni o dello stato.

### Gestione Automatica del Ciclo di Vita (`#ChildRef`)

`BaseComponent` offre un meccanismo potente per gestire automaticamente il ciclo di vita delle molecole figlie, propagando gli eventi `baseCompViewEnter` e `baseCompViewLeave`.

Per attivare questo automatismo, è sufficiente assegnare la template reference variable `#ChildRef` alle molecole nel template HTML del componente.

#### Esempio Utilizzo Singolo o Multiplo
È possibile marcare una o più molecole con `#ChildRef`. Il componente padre le rileverà automaticamente tutte e invocherà i loro metodi di inizializzazione/distruzione.

```html
<!-- Master Page Template -->

<!-- ... -->

<!-- Molecole Multiple: Tutte riceveranno gli eventi del ciclo di vita -->
<asor-header #ChildRef></asor-header>
<asor-sidebar #ChildRef></asor-sidebar>

<!-- ... -->
```

In questo scenario:
1.  Quando `MyPage` (BaseComponent) entra in view (`ngOnInit` / `ionViewWillEnter`), chiama automaticamente `baseCompViewEnter` su **TUTTE** le istanze marcate con `#ChildRef`.
2.  Quando `MyPage` esce (`ngOnDestroy` / `ionViewWillLeave`), chiama `baseCompViewLeave` su tutte le istanze.

> Questo elimina la necessità di gestire manualmente `ViewChild` e chiamare i metodi di init/destroy per ogni sotto-componente.



---

## Sistema di Cache <a id="structure-cache"></a>

`asor-core` include un sistema di caching HTTP configurabile e granulare, gestito tramite un `HTTP_INTERCEPTOR` e una classe di configurazione estendibile.

### Configurazione (`ConfigCache`)

Il comportamento della cache è definito da regole (`IPath`) specificate nella classe `ConfigCache`. Ogni regola determina come e quando una richiesta HTTP deve essere cachata.

#### Proprietà di Configurazione (`IPath`)

Le regole sono oggetti che rispettano l'interfaccia `IPath`. Ecco il significato di ogni proprietà:

*   **`persistenceType`** (`AsorGlobalEnum.CacheType`): Il tipo di persistenza della cache.
    *   `AsorGlobalEnum.CacheType.VOLATILE` (`'volatile'`): La cache è in memoria (RAM) e viene persa al refresh o alla chiusura del tab.
    *   `AsorGlobalEnum.CacheType.SESSION` (`'session'`): La cache sopravvive al reload della pagina (salvata nel `sessionStorage` del browser).
    *   `AsorGlobalEnum.CacheType.LOCAL` (`'local'`): La cache viene salvata nel `localStorage` e persiste tra le sessioni.
*   **`encriptType`** (`AsorGlobalEnum.CacheEncriptType`): Il tipo di crittografia da utilizzare.
    *   `AsorGlobalEnum.CacheEncriptType.NONE` (`'NONE'`): Nessuna crittografia.
    *   `AsorGlobalEnum.CacheEncriptType.AUTO` (`'AUTO'`): Crittografia rilevata automaticamente.
    *   `AsorGlobalEnum.CacheEncriptType.CUSTOM` (`'CUSTOM'`): Tipo di crittografia personalizzato.
*   **`pathValue`** (`string`): L'URL (o parte di esso) della richiesta HTTP da intercettare. Se la richiesta contiene questa stringa, la regola viene applicata.
*   **`resctrictRoute`** (`string`): Limita la regola a una specifica rotta dell'applicazione.
    *   `ConfigConst.Route.NONE` o `''`: La regola vale per tutte le rotte.
    *   Specificando una rotta (es. `/dashboard`), la cache si attiva solo se l'utente si trova in quella pagina.
*   **`reqPayloadCache`** (`boolean`):
    *   `true`: Include il body della richiesta (payload) nella chiave di cache. Utile per le `POST` di ricerca dove lo stesso endpoint restituisce dati diversi in base ai filtri.
    *   `false`: Ignora il payload. La chiave di cache è basata solo sull'URL.
*   **`clearOn`** (`string[]`): Elenco di endpoint che, se chiamati (solitamente con metodi non-GET come `POST`, `PUT`, `DELETE`), invalidano questa cache.
    *   Esempio: Una chiamata a `/api/users/update` può essere configurata per pulire la cache di `/api/users/list`.
*   **`callBackCacheKey`** (opzionale `() => string`): Una funzione di callback per generare una chiave di cache personalizzata per la richiesta.

### Abilitazione e Personalizzazione

Per attivare e configurare il sistema nel proprio progetto, seguire questi due passaggi:

#### 1. Estensione della Configurazione

Creare una classe di configurazione (es. `WikiCacheConfig`) che estende `ConfigCache` di `asor-core`. Definire le proprie regole nell'array protetto `_pathsExtensions`.

```typescript
// src/app/config/wiki.cache.config.ts
import { ConfigCache, IPath, ConfigConst, AsorGlobalEnum } from '@asor-studio/asor-core';

export class WikiCacheConfig extends ConfigCache {
    protected static override _pathsExtensions: IPath[] = [
        {
            persistenceType: AsorGlobalEnum.CacheType.SESSION,
            encriptType: AsorGlobalEnum.CacheEncriptType.AUTO,
            reqPayloadCache: false,
            resctrictRoute: ConfigConst.Route.NONE,
            pathValue: '/api/v1/users', // Cacha le chiamate utenti
            clearOn: ['/api/v1/users/save', '/api/v1/users/delete'] // Pulisce su salva/elimina
        },
        // ... altre regole
    ];

    // è obbligatorio per inizializzare le costanti della libreria asor-core
    static {
        ConfigCache.SetConfiguration(WikiCacheConfig);
    }
}
```

#### 2. Registrazione dell'Interceptor

Nel file di configurazione dell'app (`app.config.ts`), fornire il `CacheInterceptor` e inizializzare il servizio di configurazione.

> **Nota**: È necessario passare la configurazione estesa al servizio di utilità, solitamente nell'interceptor stesso o nell'inizializzazione dell'app, oppure assicurarsi che le costanti puntino alla classe estesa se il design lo prevede. (In `asor-core` l'interceptor usa `CachePathUtility` che legge la configurazione globale).

Registrazione in `app.config.ts`:

```typescript
// src/app/app.config.ts
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { CacheInterceptor } from '@asor-studio/asor-core'; // O dal percorso corretto della lib

export const appConfig: ApplicationConfig = {
  providers: [
    // ... altri provider
    {
      provide: HTTP_INTERCEPTORS,
      useClass: CacheInterceptor,
      multi: true,
    },
  ]
};
```

---

## Sistema di Notifica di Errore <a id="structure-error"></a>

Il sistema di gestione degli errori centralizzato permette ai componenti e alle molecole di reagire agli errori HTTP (es. 404, 500, 401) intercettati globalmente, senza dover gestire il `catchError` in ogni singola chiamata di servizio.

Il cuore del sistema è il `NotifyErrorService`, che agisce come un bus di eventi per gli errori.

### Funzionamento

1.  **Intercettazione**: L'`ErrorInterceptor` cattura ogni errore HTTP.
2.  **Notifica**: L'interceptor invia l'errore al `NotifyErrorService`.
3.  **Reazione**: I componenti registrati al servizio ricevono la notifica ed eseguono la logica appropriata (es. mostrare un toast, redirect, alert).

### Abilitazione

Per abilitare il sistema, è necessario registrare l'`ErrorInterceptor` nel file di configurazione dell'app (`app.config.ts`), similmente al sistema di cache.

```typescript
// src/app/app.config.ts
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { ErrorInterceptor } from '@asor-studio/asor-core';

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    {
      provide: HTTP_INTERCEPTORS,
      useClass: ErrorInterceptor,
      multi: true,
    },
  ]
};
```

### Utilizzo nei Componenti (`NotifyErrorService`)

Per gestire gli errori in un componente o molecola, iniettare `NotifyErrorService` e usare il metodo `registry`.

> **Importante**: Il metodo `registry` accetta come primo argomento un identificativo univoco (solitamente il nome della classe). Questo garantisce che ci sia una sola sottoscrizione attiva per componente, prevenendo memory leaks (la sottoscrizione precedente con lo stesso ID viene cancellata automaticamente).

```typescript
import { BaseComponent, NotifyErrorService, IHttpStatusCode, IHttpResponseError } from '@asor-studio/asor-core';

@Component({ ... })
export class MyComponent extends BaseComponent {
    private notifyErrorService = inject(NotifyErrorService);

    override baseCompViewEnter() {
        // Registrazione alla ricezione degli errori
        this.notifyErrorService.registry(
            this.constructor.name, // ID Univoco (Nome Classe)
            (status: HttpStatusCode, error: IHttpResponseError) => {
                this.handleError(status, error);
            }
        );
    }
    
    override baseCompViewLeave() {
         // Logica di cleanup se necessaria
    }

    private handleError(status: HttpStatusCode, error: IHttpResponseError) {
        switch (status) {
            case IHttpStatusCode.NotFound: // 404
                console.error('Risorsa non trovata');
                break;
            case IHttpStatusCode.InternalServerError: // 500
                alert('Errore del server: ' + error.message);
                break;
            default:
                console.log('Altro errore:', status);
        }
    }
}
```
