# Project Configuration

In this guide, we will cover the fundamental steps to configure a new Angular project using the `asor-core` library.

---

## 1. Installation <a id="config-installation"></a>

To get started, install the library in your project via npm:

```bash
npm install @asor-studio/asor-core
```

Ensure you also have the required peer dependencies installed (e.g., `@angular/common`, `@angular/core`, `rxjs`).

---

## 2. Assets Configuration (`angular.json`) <a id="config-assets"></a>

`asor-core` relies on dynamic loading of translation files (i18n). You need to configure `angular.json` to correctly serve public assets.

```json
"assets": [
  {
    "glob": "**/*",
    "input": "public",
    "output": "/"
  }
],
```

Ensure that the `public/assets/i18n` folder exists and contains the JSON translation files.

---

## 3. Global Configuration (`app.config.ts`) <a id="config-global"></a>

The `app.config.ts` file is the entry point for configuring global providers. Here we enable routing, the HTTP client, and `asor-core` interceptors.

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { HTTP_INTERCEPTORS, provideHttpClient, withInterceptorsFromDi } from '@angular/common/http';
import { CacheInterceptor, ErrorInterceptor } from '@asor-studio/asor-core';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    // 1. Enable Change Detection (Zone.js or Zoneless)
    provideZoneChangeDetection({ eventCoalescing: true }),
    
    // 2. Enable Routing
    provideRouter(routes),
    
    // 3. Enable HTTP Client with Interceptor DI support
    provideHttpClient(withInterceptorsFromDi()),

    // 4. Register Cache Interceptor (Cache System)
    {
      provide: HTTP_INTERCEPTORS,
      useClass: CacheInterceptor,
      multi: true,
    },

    // 5. Register Error Interceptor (Error Notification System)
    {
      provide: HTTP_INTERCEPTORS,
      useClass: ErrorInterceptor,
      multi: true,
    },
  ]
};
```

---

## 4. Application Initialization (`app.root.ts`) <a id="config-init"></a>

The root component (`AppComponent` or `AppRoot`) is responsible for initializing core services and configuring logs at application bootstrap.

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
        // 1. Log Level Configuration
        // Enable logs for specific services (e.g., StateService, MyComponent)
        ConsoleLogsConfig.setClassLevels(StateService.name || '', ['ERROR', 'WARNING']);
        // ConsoleLogsConfig.setClassLevels('MyComponent', ['INFO', 'DEBUG', 'ERROR']);

        // 2. State Service Initialization
        // Crucial for application state restoration
        inject(StateService).initialize();
    }
}
```

---

## 5. Extending Base Configurations <a id="config-extension"></a>

`asor-core` provides base configuration classes that **MUST** be extended to adapt them to project specifications.

### Constants and Routes (`ConfigConst`)

Create a class (e.g., `WikiConfig`) that extends `ConfigConst` to define routes and translation paths.

```typescript
// src/app/config/wiki.config.ts
import { ConfigConst } from '@asor-studio/asor-core';

export class WikiConfig extends ConfigConst {
    // Route Definitions
    protected static override _routeExtensions = {
        HOME: 'home',
        DASHBOARD: 'dashboard'
    };

    // Translation Path Definitions
    protected static override _translationUrlExtensions = {
        HOME: '/i18n/home',
        DASHBOARD: '/i18n/dashboard'
    };

    // Exposing typed getters (optional but recommended)
    static override get Route() { return { ...super.Route, ...this._routeExtensions }; }
    static override get TranslationUrl() { return { ...super.TranslationUrl, ...this._translationUrlExtensions }; }

    // Mandatory for initializing asor-core library constants
    static {
        ConfigConst.SetConfiguration(WikiConfig);
    }
}
```

### Cache Configuration (`ConfigCache`)

If necessary, extend `ConfigCache` to define custom caching rules.

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

    // Mandatory for initializing asor-core library constants
    static {
        ConfigCache.SetConfiguration(WikiCacheConfig);
    }
}
```
