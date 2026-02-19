# Asor Widget (`AsorWidgetComponent`)

The `AsorWidgetComponent` is a debug and inspection component included in the `asor-core` library. It appears as a **floating button** positioned in the bottom-right corner of the application that, when clicked, opens a **slide-up panel** containing real-time diagnostic tools.

The widget is designed for use during application development and testing, providing immediate access to log console and storage state without leaving the current page context.

![Widget floating button visible in the bottom-right corner](assets/docs/images/widget-storage-page.png)

## Overview <a id="widget-overview"></a>

The `AsorWidgetComponent` is composed of:

*   **Floating Button**: A circular button with the Asor Core logo, positioned in the bottom-right corner (`position: fixed`). Clicking it toggles the panel open/closed.
*   **Slide-up Panel (Slider)**: A panel that occupies 80% of the screen height, animated from bottom to top. It contains a header with navigation tabs and a content area.
*   **Navigation Tabs**: Two main tabs — **Console** and **Storage** — allowing users to switch between available views.

### Technical Details

| Property | Value |
|---|---|
| **Selector** | `asor-core-widget` |
| **Encapsulation** | `ViewEncapsulation.ShadowDom` |
| **Standalone** | `true` |
| **Module** | `asor-core` |

> The use of `ShadowDom` ensures that the widget's styles are completely isolated from the host application, preventing CSS conflicts.

### Source Code

```typescript
// projects/asor-core/src/lib/widget/asor-widget.component.ts
import { Component, HostListener, Input, ViewEncapsulation } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ConsoleTabComponent } from './console-tab/console-tab.component';
import { StorageTabComponent } from './storage-tab/storage-tab.component';

export type AsorWidgetMode = 'off' | 'on' | 'spy';

@Component({
    selector: 'asor-core-widget',
    standalone: true,
    imports: [CommonModule, ConsoleTabComponent, StorageTabComponent],
    templateUrl: './asor-widget.component.html',
    styleUrls: ['./asor-widget.component.scss'],
    encapsulation: ViewEncapsulation.ShadowDom
})
export class AsorWidgetComponent {
    @Input() mode: AsorWidgetMode = 'on';
    isOpen = false;
    activeTab: 'console' | 'storage' | 'other' = 'console';
    // ...
}
```

### Input Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `mode` | `'off' \| 'on' \| 'spy'` | `'on'` | Controls the widget visibility mode |

#### Mode Values

*   **`on`** — The widget is always visible and functional (default behavior).
*   **`off`** — The widget is completely hidden and disabled.
*   **`spy`** — The widget is hidden by default. Press **`CTRL + I`** to toggle its visibility. This is useful in production environments where the widget should remain invisible unless explicitly activated by a developer.

---

## How to Use <a id="widget-usage"></a>

To integrate the widget into your application, simply:

### 1. Import the Component

Add `AsorWidgetComponent` to the imports of your root component (or any component acting as the main layout).

```typescript
import { AsorWidgetComponent } from '@asor-studio/asor-core';

@Component({
    selector: 'app-root',
    standalone: true,
    imports: [
        // ... other imports
        AsorWidgetComponent
    ],
    template: `
        <router-outlet></router-outlet>
        <asor-core-widget mode="on"></asor-core-widget>
    `
})
export class AppRoot { }
```

### 2. Add the Tag to the Template

Insert the `<asor-core-widget>` selector in the HTML template, ideally as the last element of the main layout.

```html
<!-- Always visible -->
<asor-core-widget mode="on"></asor-core-widget>

<!-- Completely hidden -->
<asor-core-widget mode="off"></asor-core-widget>

<!-- Hidden until CTRL + I is pressed -->
<asor-core-widget mode="spy"></asor-core-widget>
```

> **Note**: The widget automatically positions itself in the bottom-right corner thanks to `position: fixed`. No additional styles are needed.
>
> In `spy` mode, pressing **`CTRL + I`** again will hide the widget and close the panel if open.

> [!TIP]
> You can try the `spy` mode right now! Navigate to the **Data Storage** section of this wiki and press **`CTRL + I`** to reveal the hidden widget.

---

## Console Tab <a id="widget-console"></a>

The **Console** tab provides a real-time log stream from the `ConsoleLogsUtility` system of `asor-core`. It allows monitoring application activity without opening browser developer tools.

![Console tab with real-time log stream](assets/docs/images/widget-console-tab.png)

### Features

*   **LOGS STREAM**: Real-time display of all logs emitted by the application, color-coded by level (DEBUG, INFO, WARNING, ERROR).
*   **Level Filters**: Filter badges by type — `INFO`, `DEBUG`, `WARNING` — with counters for each category.
*   **Search Bar**: Text search within logs (`Search logs...`).
*   **Per-Class Configuration**: Left-side panel (**CONFIGURATION**) allowing you to set the log level for each registered class/service:
    *   `StateService`
    *   `AuthGuard`
    *   `CacheInterceptor`
    *   `ErrorInterceptor`
    *   `NotifyErrorService`
    *   `HttpRequestHandler`
*   **Clear Button**: To clear the current log stream (`X Clear`).

---

## Storage Tab <a id="widget-storage"></a>

The **Storage** tab provides an inspectable view of the application's storage state (`StateService`). It displays data contained in registered DataSets and the connected component registry.

![Storage tab with Data Containers and Component Registry](assets/docs/images/widget-storage-tab.png)

### Features

*   **Status Indicators**: Top bar with key storage information:
    *   `INITIALIZED` — Initialization status (YES/NO)
    *   `SYNC INTERVAL` — Synchronizer status (Active/Inactive)
    *   `ENCRYPTION` — Encryption type used (e.g., AES)
    *   `DATA ELEMENTS` — Number of elements in storage
    *   `REGISTRY ENTRIES` — Number of registered components
*   **Data Containers**: Left panel listing registered DataSets with:
    *   Container name and type (e.g., `VOLATILE`)
    *   Encrypted data (displayed as hash)
    *   Last update timestamp
    *   Quick actions: view, copy, export, delete
*   **Component Registry**: Right-side table showing components and molecules connected to storage with:
    *   `Component` — Registered class name
    *   `Prop` — Connected property (e.g., `title`, `form`)
    *   `Path` — DataSet connection path
