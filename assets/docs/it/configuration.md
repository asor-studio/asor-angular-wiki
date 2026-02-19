# Configurazione del Progetto

In questa guida vedremo i passaggi fondamentali per configurare un nuovo progetto Angular che utilizza la libreria `asor-core`.


---

## 1. Installazione <a id="config-installation"></a>

Per iniziare, installa la libreria nel tuo progetto tramite npm:

```bash
npm install @asor-studio/asor-core
```

Assicurati di avere installate anche le dipendenze peer richieste (es. `@angular/common`, `@angular/core`, `rxjs`).

---

## 2. Configurazione Assets (`angular.json`) <a id="config-assets"></a>

`asor-core` si basa sul caricamento dinamico di file di traduzione (i18n). È necessario configurare `angular.json` per servire correttamente gli asset pubblici.

```json
"assets": [
  {
    "glob": "**/*",
    "input": "public",
    "output": "/"
  }
],
```

Assicurati che la cartella `public/assets/i18n` esista e contenga i file JSON delle traduzioni.

---

## 3. Configurazione Globale (`app.config.ts`) <a id="config-global"></a>

Il file `app.config.ts` è il punto di ingresso per la configurazione dei provider globali. Qui abilitiamo il routing, il client HTTP e gli interceptor di `asor-core`.

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { HTTP_INTERCEPTORS, provideHttpClient, withInterceptorsFromDi } from '@angular/common/http';
import { CacheInterceptor, ErrorInterceptor } from '@asor-studio/asor-core';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    // 1. Abilita Change Detection (Zone.js o Zoneless)
    provideZoneChangeDetection({ eventCoalescing: true }),
    
    // 2. Abilita Routing
    provideRouter(routes),
    
    // 3. Abilita HTTP Client con supporto Interceptor DI
    provideHttpClient(withInterceptorsFromDi()),

    // 4. Registra Cache Interceptor (Sistema di Cache)
    {
      provide: HTTP_INTERCEPTORS,
      useClass: CacheInterceptor,
      multi: true,
    },

    // 5. Registra Error Interceptor (Sistema Notifica Errori)
    {
      provide: HTTP_INTERCEPTORS,
      useClass: ErrorInterceptor,
      multi: true,
    },
  ]
};
```

---

## 4. Inizializzazione Applicazione (`app.root.ts`) <a id="config-init"></a>

Il componente root (`AppComponent` o `AppRoot`) è responsabile dell'inizializzazione dei servizi core e della configurazione dei log al bootstrap dell'applicazione.

```typescript
import { Component, inject } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { ConsoleLogsConfig, StateService } from '@asor-studio/asor-core';

@Component({
    selector: 'app-root',
    standalone: true,
    imports: [RouterOutlet],
    template: `<router-outlet></router-outlet>`
})
export class AppComponent {
    constructor() {
        // 1. Configurazione Livelli di Log
        // Abilita i log per servizi specifici (es. StateService, MyComponent)
        ConsoleLogsConfig.setClassLevels(StateService.name || '', ['ERROR', 'WARNING']);
        // ConsoleLogsConfig.setClassLevels('MyComponent', ['INFO', 'DEBUG', 'ERROR']);

        // 2. Inizializzazione State Service
        // Fondamentale per il ripristino dello stato applicativo
        inject(StateService).initialize();
    }
}
```

---

## 5. Estensione Configurazioni Base <a id="config-extension"></a>

`asor-core` fornisce classi di configurazione base che **DEVONO** essere estese per adattarle alle specifiche del progetto.

### Costanti e Rotte (`ConfigConst`)

Crea una classe (es. `WikiConfig`) che estende `ConfigConst` per definire le rotte e i path delle traduzioni.

```typescript
// src/app/config/wiki.config.ts
import { ConfigConst } from '@asor-studio/asor-core';

export class WikiConfig extends ConfigConst {
    // Definizione Rotte
    protected static override _routeExtensions = {
        HOME: 'home',
        DASHBOARD: 'dashboard'
    };

    // Definizione Path Traduzioni
    protected static override _translationUrlExtensions = {
        HOME: '/i18n/home',
        DASHBOARD: '/i18n/dashboard'
    };

    // Esposizione getter tipizzati (opzionale ma consigliato)
    static override get Route() { return { ...super.Route, ...this._routeExtensions }; }
    static override get TranslationUrl() { return { ...super.TranslationUrl, ...this._translationUrlExtensions }; }

    // è obbligatorio per inizializzare le costanti della libreria asor-core
    static {
        ConfigConst.SetConfiguration(WikiConfig);
    }
}
```

### Configurazione Cache (`ConfigCache`)

Se necessario, estendi `ConfigCache` per definire regole di caching personalizzate.

```typescript
// src/app/config/wiki.cache.config.ts
import { ConfigCache, IPath, ConfigConst } from '@asor-studio/asor-core';

export class WikiCacheConfig extends ConfigCache {
    protected static override _pathsExtensions: IPath[] = [
        {
            pathValue: '/api/v1/secure-data',
            isPersistent: true,
            resctrictRoute: ConfigConst.Route.NONE,
            reqPayloadCache: false,
            clearOn: []
        }
    ];

    // è obbligatorio per inizializzare le costanti della libreria asor-core
    static {
        ConfigCache.SetConfiguration(WikiCacheConfig);
    }
}
```
