# Configurazione del Progetto

Questa guida segue il bootstrap reale prodotto da `asor setup`, quindi i passaggi qui sotto corrispondono ai file e alle cartelle che la CLI genera davvero dentro un workspace Angular.

---

## 1. Installa la CLI <a id="config-installation"></a>

Installa la CLI ufficiale:

```bash
npm install -g @asor-studio/asor-cli
```

Se la installi localmente invece che in globale, eseguila con `npx` oppure `npm exec`:

```bash
npx asor help
```

```bash
npm exec asor help
```

`asor setup` deve essere eseguito dalla root del workspace Angular, cioe nella stessa cartella in cui si trova `angular.json`.

---

## 2. Esegui `asor setup` <a id="config-cli-setup"></a>

Se parti da zero:

```bash
ng new my-angular-project
cd my-angular-project
asor setup
```

Durante il setup, la CLI chiede:

- `Prefix`: usato per rinominare classi di configurazione, file, selector e namespace i18n
- `baseHref`: normalizzato con `/` iniziale e finale e riutilizzato sia nella configurazione Angular sia in quella ASOR

Valori di esempio:

- `Prefix`: `App`, `Nexus`, `Portal`
- `baseHref`: `/`, `/portal/`

---

## 3. Cosa genera `asor setup` <a id="config-generated"></a>

Quando il comando termina, esegue queste operazioni:

1. Installa `@asor-studio/asor-core` e `crypto-js`
2. Assicura l'esistenza delle cartelle Atomic Design in `src/app`
3. Copia lo scaffold ASOR dentro l'app Angular
4. Crea i namespace i18n di default per `page-home` e `molecule-sample-widget`
5. Aggiorna `angular.json` con il `baseHref` scelto
6. Inietta `<asor-core-widget mode="on" />` nel componente root quando possibile
7. Importa `AsorWidgetComponent` nel componente standalone root quando serve
8. Salva i valori scelti in `asor-core.json`

La struttura generata include:

- `src/app/app.config.ts`
- `src/app/app.routes.ts`
- `src/app/config/<prefix>.config.ts`
- `src/app/config/<prefix>-cache.config.ts`
- `src/app/config/<prefix>-asor.config.ts`
- `src/app/config/<prefix>-state.config.ts`
- `src/app/config/interfaces/<prefix>-state.interfaces.ts`
- `src/app/pages/page-home/*`
- `src/app/molecules/molecule-sample-widget/*`

---

## 4. Asset generati e i18n <a id="config-assets"></a>

La CLI crea le traduzioni in:

- `public/assets/i18n/<prefix>/...` se il progetto contiene gia una cartella `public`
- `src/assets/i18n/<prefix>/...` negli altri casi

I namespace iniziali sono:

- `page-home`
- `molecule-sample-widget`

Per ogni namespace vengono creati:

- `en.json`
- `it.json`

Inoltre il file generato `<prefix>.config.ts` viene gia aggiornato in modo che:

- `SiteBaseUrl` corrisponda al `baseHref` scelto
- `SiteAssetsUrl` punti a `<baseHref>/assets`
- `TranslationUrl.HOME` punti a `/<prefix>/page-home`
- `TranslationUrl.SAMPLE_WIDGET` punti a `/<prefix>/molecule-sample-widget`
- `Cache.<PREFIX>_BASE_I18N` punti a `i18n/<prefix>`

---

## 5. Setup manuale senza `asor setup` <a id="config-manual"></a>

Se preferisci configurare il progetto a mano, puoi costruire un progetto ASOR minimo e funzionante replicando i blocchi principali che la CLI genera automaticamente.

### Step 1. Installa le dipendenze

```bash
npm install @asor-studio/asor-core crypto-js
```

### Step 2. Crea le cartelle base

Crea almeno:

- `src/app/config`
- `src/app/config/interfaces`
- `src/app/pages/page-home`
- `src/app/molecules/molecule-sample-widget`

Crea anche la cartella i18n in una di queste posizioni:

- `public/assets/i18n/app`
- `src/assets/i18n/app`

### Step 3. Aggiungi le costanti principali ASOR

Crea `src/app/config/app.config.ts`:

```ts
import { ConfigConst } from '@asor-studio/asor-core';

export class AppConfig extends ConfigConst {
  static override SiteBaseUrl = '/';
  static override SiteAssetsUrl = '/assets';

  protected static override _routeExtensions = {
    HOME: 'home',
  };

  protected static override _translationUrlExtensions = {
    DefaultLanguage: 'en',
    HOME: '/page-home',
    SAMPLE_WIDGET: '/molecule-sample-widget',
  };

  protected static override _urlExtensions = {
    HOME: '/home',
  };

  protected static override _cacheExtensions = {
    APP_BASE_I18N: 'i18n/app',
  };

  static override get Route() { return { ...super.Route, ...this._routeExtensions }; }
  static override get TranslationUrl() { return { ...super.TranslationUrl, ...this._translationUrlExtensions }; }
  static override get Url() { return { ...super.Url, ...this._urlExtensions }; }
  static override get Cache() { return { ...super.Cache, ...this._cacheExtensions }; }

  static {
    ConfigConst.SetConfiguration(AppConfig);
  }
}
```

### Step 4. Definisci interfaccia, dataset e cache

Crea `src/app/config/interfaces/app-state.interfaces.ts`:

```ts
export interface IAppSystemStatus {
  isMenuOpen: boolean;
  appVersion: string;
  currentUser: string | null;
}
```

Crea `src/app/config/app-state.config.ts`:

```ts
import { AsorStorage, ICreateDataSet, IConnectDataSet } from '@asor-studio/asor-core';
import { IAppSystemStatus } from './interfaces/app-state.interfaces';

export const AppStateCreateSystemDataSet: ICreateDataSet = {
  name: 'app-system-status',
  data: {
    isMenuOpen: false,
    appVersion: '1.0.0',
    currentUser: null,
  } as IAppSystemStatus,
  option: {
    storeType: AsorStorage.StateConst.StoreType.VOLATILE,
    encrypt: false,
  },
};

export const AppStateConnectionSystem: IConnectDataSet = {
  name: 'AppSystemConnection',
  selectors: {
    isMenuOpen: 'app-system-status.isMenuOpen',
    appVersion: 'app-system-status.appVersion',
    currentUser: 'app-system-status.currentUser',
  },
};
```

Crea `src/app/config/app-cache.config.ts`:

```ts
import { AsorGlobalEnum, ConfigCache } from '@asor-studio/asor-core';
import { AppConfig } from './app.config';

export class AppCacheConfig extends ConfigCache {
  public static override globalStateName = 'app';

  protected static override _pathsExtensions = [
    {
      pathValue: AppConfig.Cache.APP_BASE_I18N,
      persistenceType: AsorGlobalEnum.CacheType.VOLATILE,
      encriptType: AsorGlobalEnum.CacheEncriptType.NONE,
      reqPayloadCache: false,
      resctrictRoute: AppConfig.Route.NONE,
      clearOn: [],
    },
  ];

  static {
    ConfigCache.SetConfiguration(AppCacheConfig);
  }
}
```

### Step 5. Inizializza ASOR in `app.config.ts`

Crea `src/app/config/app-asor.config.ts`:

```ts
import { ConsoleLogsConfig, StateService, AsorStorage } from '@asor-studio/asor-core';
import { AppConfig } from './app.config';
import { AppCacheConfig } from './app-cache.config';
import { AppStateCreateSystemDataSet } from './app-state.config';

export function initializeAsorCoreApp(stateService: StateService) {
  return () => {
    ConsoleLogsConfig.silent = false;

    AppConfig.init?.();
    AppCacheConfig.init?.();

    stateService.initialize({
      globalStateName: 'app',
      encryptionType: AsorStorage.StateConst.EncryptType.AES,
      nameType: AsorStorage.StateConst.Generate.AUTO,
      keyType: AsorStorage.StateConst.Generate.AUTO,
      asyncEnabled: true,
    });

    stateService.createDataSet(
      AppStateCreateSystemDataSet.name,
      AppStateCreateSystemDataSet.data,
      AppStateCreateSystemDataSet.option!
    );

    return true;
  };
}
```

Poi collegalo in `src/app/app.config.ts`:

```ts
import { ApplicationConfig, APP_INITIALIZER, provideZonelessChangeDetection } from '@angular/core';
import { APP_BASE_HREF } from '@angular/common';
import { provideRouter, withComponentInputBinding } from '@angular/router';
import { HTTP_INTERCEPTORS, provideHttpClient, withInterceptorsFromDi } from '@angular/common/http';
import { CacheInterceptor, ErrorInterceptor, MockHttpInterceptor, MockOrchestratorService, StateService } from '@asor-studio/asor-core';
import { routes } from './app.routes';
import { initializeAsorCoreApp } from './config/app-asor.config';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: initializeAsorCoreApp,
      deps: [StateService, MockOrchestratorService],
      multi: true,
    },
    provideZonelessChangeDetection(),
    provideRouter(routes, withComponentInputBinding()),
    provideHttpClient(withInterceptorsFromDi()),
    { provide: HTTP_INTERCEPTORS, useClass: MockHttpInterceptor, multi: true },
    { provide: HTTP_INTERCEPTORS, useClass: CacheInterceptor, multi: true },
    { provide: HTTP_INTERCEPTORS, useClass: ErrorInterceptor, multi: true },
    { provide: APP_BASE_HREF, useValue: '/' },
  ],
};
```

### Step 6. Crea una pagina, una molecule e la route

Versione minima di `src/app/molecules/molecule-sample-widget/sample-widget.molecule.ts`:

```ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { BaseStorageMolecule, TranslatePipe } from '@asor-studio/asor-core';
import { IAppSystemStatus } from '../../config/interfaces/app-state.interfaces';

@Component({
  selector: 'app-sample-widget',
  standalone: true,
  imports: [CommonModule, TranslatePipe],
  template: `<section>{{ 'TITLE' | translate }}</section>`,
})
export class SampleWidgetMolecule extends BaseStorageMolecule<IAppSystemStatus> {
  static override readonly className = 'SampleWidgetMolecule';
}
```

Versione minima di `src/app/pages/page-home/home.component.ts`:

```ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { BaseStorageComponent, TranslatePipe } from '@asor-studio/asor-core';
import { IAppSystemStatus } from '../../config/interfaces/app-state.interfaces';
import { SampleWidgetMolecule } from '../../molecules/molecule-sample-widget/sample-widget.molecule';

@Component({
  selector: 'app-home-page',
  standalone: true,
  imports: [CommonModule, TranslatePipe, SampleWidgetMolecule],
  template: `
    <h1>{{ 'TITLE' | translate }}</h1>
    <app-sample-widget />
  `,
})
export class PageHomeComponent extends BaseStorageComponent<IAppSystemStatus> {
  static override readonly className = 'PageHomeComponent';
}
```

Crea `src/app/app.routes.ts`:

```ts
import { Routes } from '@angular/router';
import { IAsorRoute } from '@asor-studio/asor-core';
import { AppConfig } from './config/app.config';
import { AppStateConnectionSystem } from './config/app-state.config';
import { PageHomeComponent } from './pages/page-home/home.component';
import { SampleWidgetMolecule } from './molecules/molecule-sample-widget/sample-widget.molecule';

export const routes: Routes = [
  { path: '', redirectTo: AppConfig.Route.HOME, pathMatch: 'full' },
  {
    path: AppConfig.Route.HOME,
    component: PageHomeComponent,
    data: {
      I18nPath: [AppConfig.TranslationUrl.HOME],
      ConnectDataSet: AppStateConnectionSystem,
      Molecules: [
        {
          Molecule: SampleWidgetMolecule,
          I18nPath: [AppConfig.TranslationUrl.SAMPLE_WIDGET],
        },
      ],
    } as IAsorRoute,
  },
];
```

### Step 7. Aggiungi il widget root e le traduzioni

Nel template del componente root mantieni entrambi:

```html
<router-outlet />
<asor-core-widget mode="on" />
```

Se il componente root e standalone, importa `AsorWidgetComponent` da `@asor-studio/asor-core`.

Poi crea:

- `assets/i18n/app/page-home/en.json`
- `assets/i18n/app/page-home/it.json`
- `assets/i18n/app/molecule-sample-widget/en.json`
- `assets/i18n/app/molecule-sample-widget/it.json`

Esempio di `en.json`:

```json
{
  "TITLE": "Home"
}
```

Con questi file il progetto ha gli stessi pezzi minimi usati da `asor setup`: registrazione config, inizializzazione dello stato, metadata delle route, namespace i18n e widget ASOR globale.

---

## 6. File da rivedere per primi <a id="config-review"></a>

Dopo il setup, questi sono i file principali da toccare.

### `src/app/app.config.ts`

Questo file e gia scaffoldato con i provider runtime di ASOR. Include:

- `APP_INITIALIZER` collegato a `initializeAsorCoreApp`
- `provideRouter(routes, withComponentInputBinding())`
- `provideHttpClient(withInterceptorsFromDi())`
- `MockHttpInterceptor`, `CacheInterceptor` ed `ErrorInterceptor`
- `APP_BASE_HREF` impostato al `baseHref` selezionato

### `src/app/config/<prefix>.config.ts`

E il file principale delle costanti generate a partire dal prefix scelto. Usalo per verificare o estendere:

- base path API
- route
- namespace i18n
- auth check
- namespace cache
- URL del sito

### `src/app/config/<prefix>-asor.config.ts`

Questo file contiene il bootstrap eseguito da `APP_INITIALIZER`. Di default:

- configura i livelli di log ASOR
- inizializza `<prefix>.config.ts` e `<prefix>-cache.config.ts`
- registra l'esempio `PageHomeComponent`
- inizializza `StateService`
- crea il dataset globale di default definito in `<prefix>-state.config.ts`

### `src/app/app.routes.ts`

Il file delle route generato include gia:

- redirect da `''` a `Route.HOME`
- la route della pagina `home`
- il `data` della route con i18n, connessione dataset e slot componenti
- il sample widget registrato dentro `Molecules`

---

## 7. Prime modifiche consigliate dopo il setup <a id="config-next-steps"></a>

Questi sono di solito i primi interventi richiesti in un progetto reale:

1. Modifica `src/app/config/<prefix>-asor.config.ts` sostituendo il bootstrap di esempio con dataset, controller e scelte di logging reali
2. Rivedi `src/app/config/<prefix>.config.ts` e aggiorna route, path API e namespace di traduzione
3. Sostituisci o amplia `src/app/config/<prefix>-state.config.ts` e `src/app/config/interfaces/<prefix>-state.interfaces.ts`
4. Ripulisci `src/app/app.routes.ts`, rimuovendo i placeholder di esempio che non vuoi mantenere
5. Aggiorna lo scaffold `page-home` e `molecule-sample-widget`, oppure sostituiscilo con componenti generati da te

---

## 8. Continua con i generatori <a id="config-generators"></a>

Una volta completato il setup, puoi generare altri building block:

```bash
asor -g -page user-profile
asor -g -molecule menu-bar
asor -g -atom status-badge
asor -g -organism dashboard-shell
```

Flag utili:

- `-full`: abilita injection di route, i18n e configurazione
- `-storage`: genera la variante con classe base storage-aware
- `-in <page>`: inietta atom, molecule o organism in una pagina specifica quando usato insieme a `-full`

Per vedere l'elenco completo dei comandi:

```bash
asor help
```
