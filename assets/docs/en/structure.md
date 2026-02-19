# Project Structure and Configuration

In this section, we will analyze how to build and configure a project using the `asor-core` library. We will start by analyzing the folder structure and naming conventions.

## Scaffolding and Naming Convention <a id="structure-scaffolding"></a>

The `src/app` structure is semantically organized to favor scalability and maintainability, following strict naming conventions to clearly distinguish the roles of various artifacts (Pages, Components, Molecules).

### Folder Structure (`src/app`)

The application root is divided into the following macro-areas:

*   📂 **`pages/`**: Contains application pages (entire views reachable via routing).
*   📂 **`components/`**: Contains "smart" components or reusable complex organisms.
*   📂 **`molecules/`**: Contains molecules, reusable functional UI components.
*   📂 **`config/`**: Contains global configuration files (routes, constants, cache).

```text
src/
└── app/
    ├── pages/                     # Page Containers (Views)
    │   ├── page-home/
    │   │   ├── home.component.ts  # Page Logic
    │   │   ├── home.component.html
    │   │   └── ...
    │   └── ...
    ├── components/                # Smart Components / Organisms
    │   ├── component-placeholder/
    │   │   ├── placeholder.component.ts
    │   │   └── ...
    │   └── ...
    ├── molecules/                 # Reusable UI Components
    │   ├── molecule-menu/
    │   │   ├── menu.molecule.ts   # Note the .molecule.ts suffix
    │   │   └── ...
    │   └── ...
    └── config/                    # Global Configurations
        ├── wiki.config.ts
        └── ...
```

### Naming Conventions

To ensure consistency and immediate recognition, each artifact type follows a specific naming pattern for folders, files, and classes.

#### 1. Pages (`pages/`)

Pages represent main views and act as containers for components and molecules.

*   **Folder**: Prefix `page-` (e.g., `page-storage`, `page-home`).
*   **File**: `<name>.component.ts` (e.g., `storage.component.ts`).
*   **Class**: Prefix `Page` + PascalCase (e.g., `PageStorageComponent`).
*   **Selector**: Prefix `asor-` (e.g., `asor-storage`).

#### 2. Components (`components/`)

Components are complex building blocks, often containing business logic or coordinating multiple molecules.

*   **Folder**: Prefix `component-` (e.g., `component-placeholder`).
*   **File**: `<name>.component.ts` (e.g., `placeholder.component.ts`).
*   **Class**: PascalCase + Suffix `Component` (e.g., `PlaceholderComponent`).
*   **Selector**: Prefix `asor-` (e.g., `asor-placeholder`).

#### 3. Molecules (`molecules/`)

Molecules are reusable functional units (e.g., card, menu, form).

*   **Folder**: Prefix `molecule-` (e.g., `molecule-menu`).
*   **File**: `<name>.molecule.ts` (Note the `.molecule.ts` suffix).
*   **Class**: PascalCase + Suffix `Molecule` (e.g., `MenuMolecule`).
*   **Selector**: Prefix `asor-` (e.g., `asor-menu`).

---

## Route Configuration (`IAsorRoute`) <a id="structure-routes"></a>

In the `src/app/app.routes.ts` file, routes are defined using the `IAsorRoute` interface. The most important part of this configuration is the `data` object, which allows passing essential information for component initialization, state management, and internationalization.

Here is an example configuration for the `STORAGE` route:

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

## Authentication System (`AuthCheck`) <a id="structure-auth"></a>

The `AuthCheck` property is an optional parameter of the `IAsorRoute` interface that enables a server-side security check before allowing access to the route.

```typescript
// Route configuration example
{
    path: 'admin-panel',
    component: AdminComponent,
    data: {
        AuthCheck: 'ADMIN_ACCESS_CODE', // Unique control code
    } as IAsorRoute
}
```

### Execution Flow
When the Angular Router attempts to navigate to a route protected by `AuthCheck`, the `AuthGuard` intercepts the navigation and performs the following steps:

1.  **Validation**: Checks if the `AuthCheck` property is set.
2.  **API Call**: Executes an HTTP GET request to the configured backend endpoint, appending the authorization code.
    *   Endpoint: `auth/flow/{code}`
    *   Example URL: `https://api.example.com/api/v1/auth/flow/ADMIN_ACCESS_CODE`
3.  **Response Analysis**: Awaits a JSON object conforming to the `IAuth` interface.
4.  **Decision**:
    *   If the user is authorized (`ok`), navigation proceeds.
    *   If unauthorized or session expired, a redirect acts.

### Interfaces and Definitions

The frontend expects a structured response according to the following definitions (present in `asor-core`):

**Response Interface (`IAuth`)**
```typescript
export interface IAuth {
    /** Authorization status */
    authorized: AuthStatus;
}
```

**Authorization Status Enum (`AuthStatus`)**
```typescript
export enum AuthStatus {
    /** Authorization granted */
    Ok = 'ok',
    
    /** Authorization denied (user logged in but without permissions) */
    NotAllow = 'na',
    
    /** Session expired or invalid token */
    Expired = 'ex'
}
```

### Guard Behavior (`AuthGuard`)

The `AuthGuard` handles various response states by applying specific redirect logic:

*   **`AuthStatus.Ok` (`'ok'`)**: `return true` (Navigation allowed).
*   **`AuthStatus.NotAllow` (`'na'`)**: `return false` -> Redirect to **Unauthorized** (`ConfigConst.Url.UNAUTHORIZED`).
*   **`AuthStatus.Expired` (`'ex'`)**: `return false` -> Redirect to **Login** (`ConfigConst.Url.LOGIN`).

### Backend Implementation Example (Node.js / Express)

```typescript
// Express.js endpoint example
app.get('/api/v1/auth/flow/:code', (req, res) => {
    const authCode = req.params.code; // e.g., 'ADMIN_ACCESS_CODE'
    const userSession = req.session.user; 

    // 1. Check Session
    if (!userSession || isSessionExpired(userSession)) {
        return res.json({ authorized: 'ex' });
    }

    // 2. Check Permissions
    const hasPermission = checkUserPermission(userSession, authCode);
    return res.json({ authorized: hasPermission ? 'ok' : 'na' });
});
```

---

## Translation System (`I18nPath`) <a id="structure-i18n"></a>

The `asor-core` translation system is based on dynamic loading of JSON files based on the route and current language.

### Configuration (`I18nPath`)

The `I18nPath` property in `IAsorRoute` is an array of strings defining which modules to load translations from for the current page.

```typescript
I18nPath: [WikiConfig.TranslationUrl.STORAGE, WikiConfig.TranslationUrl.SHARED]
```

#### Path Definition
Values (e.g., `/asor/storage`) are defined in the project configuration class (`WikiConfig`), extending `ConfigConst`.

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

### File Structure and Content

Translation files must be located in `public/assets/i18n/<path>/[lang].json`.

*   **Path**: Corresponds to the value defined in `WikiConfig` (e.g., `/asor/storage`).
*   **Lang**: Language code (e.g., `it`, `en`).
*   **Content**: **Flat Map** (Flat Key-Value). Nested objects are **NOT** supported.

Example `public/assets/i18n/asor/storage/it.json`:
```json
{
    "TITLE": "Welcome",
    "SUBTITLE": "Storage Management",
    "BUTTON_SAVE": "Save"
}
```

### Language Configuration

The default language (`DefaultLanguage`) is defined in `ConfigConst` (default: 'it'). To modify it, override in `WikiConfig`:

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

### Usage of Translations

#### 1. In HTML Templates (`TranslatePipe`)
Import `TranslatePipe` into the standalone component and use the `| translate` pipe.

```html
<h1>{{ 'TITLE' | translate }}</h1>
<p>{{ 'WELCOME' | translate : { name: 'Mario' } }}</p>
```

#### 2. In TypeScript Code
Inject `TranslatePipe` and use the `transform()` method.

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

## Components and Molecules System <a id="structure-components"></a>

In `asor-core`, the UI architecture follows the **Atomic Design** pattern, clearly distinguishing between **Components** (often corresponding to Organisms or Entire Pages) and **Molecules** (smaller reusable components).

All components and molecules used in a page **MUST** be registered in the route configuration to function correctly (e.g., for translation loading or state management).

### Conceptual Differences

#### 1. Components (`BaseComponent`)
Represent high-level application blocks, such as Pages (`PageStorageComponent`) or complex macro-sections (`ExampleStorageComponent`).
*   **Role**: Manage main business logic, page initialization, and coordinate molecules.
*   **Base Class**: Extend `BaseComponent`.
*   **Lifecycle**: Automatically handle Angular (`ngOnInit`, `ngOnDestroy`) and Ionic (`ionViewWillEnter`, `ionViewWillLeave`) lifecycles.

#### 2. Molecules (`BaseMolecule`)
Represent functional and reusable UI components (e.g., a user card, a search form, a dialog window).
*   **Role**: Provide specific and isolated functionality. Often contained within a Component.
*   **Base Class**: Extend `BaseMolecule`.
*   **Lifecycle**: Inherit lifecycle from the parent Component (if correctly connected via `@ViewChild`).

### Configuration (`Components` and `Molecules`)

In the `app.routes.ts` file, each route must declare which Components and Molecules it uses in the `data` object.

```typescript
// src/app/app.routes.ts
data: {
    // ...
    Components: [
        {
            Component: ExampleStorageComponent, // Component Class
            I18nPath: [WikiConfig.TranslationUrl.STORAGE], // Specific translations
            ConnectDataSet: ExampleConnectDataSet // State connection (optional)
        }
    ],
    Molecules: [
        {
            Molecule: ExampleStorageMolecule, // Molecule Class
            I18nPath: [WikiConfig.TranslationUrl.STORAGE], // Specific translations
            ConnectDataSet: ExampleConnectDataSet // State connection (optional)
        }
    ]
} as IAsorRoute
```

### Implementation in Classes

To benefit from these features, classes must extend their respective base classes.

#### Component Example
```typescript
import { BaseComponent } from '@asor-studio/asor-core';

@Component({ ... })
export class MyPage extends BaseComponent {
    // I18nPath is automatically populated by routing
    
    // Mandatory Implementation
    override baseCompViewEnter() { }
    override baseCompViewLeave() { }
}
```

#### Molecule Example
```typescript
import { BaseMolecule } from '@asor-studio/asor-core';

@Component({ ... })
export class MyCard extends BaseMolecule {
    // Retrieves specific configuration for 'MyCard' from routing
    
    // Mandatory Implementation
    override baseCompViewEnter() { }
    override baseCompViewLeave() { }
}
```

> **Fundamental Note**: If a Component or Molecule is not registered in the `Components` or `Molecules` array of the route, `asor-core` will not be able to inject the correct configurations (like `I18nPath`), causing errors in translation or state loading.

### Automatic Lifecycle Management (`#ChildRef`)

`BaseComponent` offers a powerful mechanism to automatically manage child molecule lifecycles, propagating `baseCompViewEnter` and `baseCompViewLeave` events.

To activate this automation, simply assign the template reference variable `#ChildRef` to molecules in the component's HTML template.

#### Single or Multiple Usage Example
One or more molecules can be marked with `#ChildRef`. The parent component will automatically detect all of them and invoke their initialization/destruction methods.

```html
<!-- Master Page Template -->

<!-- ... -->

<!-- Multiple Molecules: All will receive lifecycle events -->
<asor-header #ChildRef></asor-header>
<asor-sidebar #ChildRef></asor-sidebar>

<!-- ... -->
```

In this scenario:
1.  When `MyPage` (BaseComponent) enters view (`ngOnInit` / `ionViewWillEnter`), it automatically calls `baseCompViewEnter` on **ALL** instances marked with `#ChildRef`.
2.  When `MyPage` leaves (`ngOnDestroy` / `ionViewWillLeave`), it calls `baseCompViewLeave` on all instances.

> This eliminates the need to manually manage `ViewChild` and call init/destroy methods for each sub-component.

---

## Cache System <a id="structure-cache"></a>

`asor-core` includes a configurable and granular HTTP caching system, managed via an `HTTP_INTERCEPTOR` and an extensible configuration class.

### Configuration (`ConfigCache`)

Cache behavior is defined by rules (`IPath`) specified in the `ConfigCache` class. Each rule determines how and when an HTTP request should be cached.

#### Configuration Properties (`IPath`)

Rules are objects respecting the `IPath` interface. Here is the meaning of each property:

*   **`pathValue`** (`string`): The URL (or part of it) of the HTTP request to intercept. If the request contains this string, the rule applies.
*   **`resctrictRoute`** (`string`): Limits the rule to a specific application route.
    *   `ConfigConst.Route.NONE` or `''`: The rule applies to all routes.
    *   Specifying a route (e.g., `/dashboard`), the cache activates only if the user is on that page.
*   **`isPersistent`** (`boolean`):
    *   `true`: The cache survives page reload (saved in browser `sessionStorage`).
    *   `false`: The cache is in-memory (RAM) and is lost upon refresh or tab closure.
*   **`reqPayloadCache`** (`boolean`):
    *   `true`: Include request body (payload) in cache key. Useful for `POST` search requests where the same endpoint returns different data based on filters.
    *   `false`: Ignore payload. Cache key is based only on URL.
*   **`clearOn`** (`string[]`): List of endpoints that, if called (usually with non-GET methods like `POST`, `PUT`, `DELETE`), invalidate this cache.
    *   Example: A call to `/api/users/update` can be configured to clear the cache of `/api/users/list`.

### Enabling and Customization

To activate and configure the system in your project, follow these two steps:

#### 1. Extending Configuration

Create a configuration class (e.g., `WikiCacheConfig`) extending `ConfigCache` from `asor-core`. Define your rules in the protected `_pathsExtensions` array.

```typescript
// src/app/config/wiki.cache.config.ts
import { ConfigCache, IPath, ConfigConst } from '@asor-studio/asor-core';

export class WikiCacheConfig extends ConfigCache {
    protected static override _pathsExtensions: IPath[] = [
        {
            isPersistent: true,
            reqPayloadCache: false,
            resctrictRoute: ConfigConst.Route.NONE,
            pathValue: '/api/v1/users', // Caches user calls
            clearOn: ['/api/v1/users/save', '/api/v1/users/delete'] // Clears on save/delete
        },
        // ... other rules
    ];

    // Mandatory for initializing asor-core library constants
    static {
        ConfigCache.SetConfiguration(WikiCacheConfig);
    }
}
```

#### 2. Interceptor Registration

In the app configuration file (`app.config.ts`), provide `CacheInterceptor` and initialize the configuration service.

> **Note**: You need to pass the extended configuration to the utility service, usually in the interceptor itself or app initialization, or ensure constants point to the extended class if the design allows. (In `asor-core`, the interceptor uses `CachePathUtility` which reads global configuration).

Registration in `app.config.ts`:

```typescript
// src/app/app.config.ts
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { CacheInterceptor } from '@asor-studio/asor-core'; // Or from the correct lib path

export const appConfig: ApplicationConfig = {
  providers: [
    // ... other providers
    {
      provide: HTTP_INTERCEPTORS,
      useClass: CacheInterceptor,
      multi: true,
    },
  ]
};
```

---

## Error Notification System <a id="structure-error"></a>

The centralized error management system allows components and molecules to react to HTTP errors (e.g., 404, 500, 401) intercepted globally, without handling `catchError` in every single service call.

The core of the system is `NotifyErrorService`, acting as an error event bus.

### Functioning

1.  **Interception**: `ErrorInterceptor` captures every HTTP error.
2.  **Notification**: The interceptor sends the error to `NotifyErrorService`.
3.  **Reaction**: Components registered to the service receive the notification and execute appropriate logic (e.g., show toast, redirect, alert).

### Enabling

To enable the system, register `ErrorInterceptor` in the app configuration file (`app.config.ts`), similar to the cache system.

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

### Usage in Components (`NotifyErrorService`)

To handle errors in a component or molecule, inject `NotifyErrorService` and use the `registry` method.

> **Important**: The `registry` method accepts a unique identifier as the first argument (usually the class name). This ensures only one active subscription per component, preventing memory leaks (the previous subscription with the same ID is automatically cancelled).

```typescript
import { BaseComponent, NotifyErrorService, IHttpStatusCode, IHttpResponseError } from '@asor-studio/asor-core';

@Component({ ... })
export class MyComponent extends BaseComponent {
    private notifyErrorService = inject(NotifyErrorService);

    override baseCompViewEnter() {
        // Registering for error reception
        this.notifyErrorService.registry(
            this.constructor.name, // Unique ID (Class Name)
            (status: HttpStatusCode, error: IHttpResponseError) => {
                this.handleError(status, error);
            }
        );
    }
    
    override baseCompViewLeave() {
         // Cleanup logic if needed
    }

    private handleError(status: HttpStatusCode, error: IHttpResponseError) {
        switch (status) {
            case IHttpStatusCode.NotFound: // 404
                console.error('Resource not found');
                break;
            case IHttpStatusCode.InternalServerError: // 500
                alert('Server error: ' + error.message);
                break;
            default:
                console.log('Other error:', status);
        }
    }
}
```
