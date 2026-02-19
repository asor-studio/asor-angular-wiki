# Asor Widget (`AsorWidgetComponent`)

L'`AsorWidgetComponent` è un componente di debug e ispezione incluso nella libreria `asor-core`. Si presenta come un **pulsante floating** posizionato nell'angolo inferiore destro dell'applicazione che, una volta cliccato, apre un **pannello a scomparsa** contenente strumenti di diagnostica in tempo reale.

Il widget è progettato per essere utilizzato durante lo sviluppo e il testing dell'applicazione, offrendo accesso immediato a console di log e stato dello storage senza lasciare il contesto della pagina corrente.

![Pulsante floating del widget visibile nell'angolo inferiore destro](/assets/docs/images/widget-storage-page.png)

## Panoramica <a id="widget-overview"></a>

Il componente `AsorWidgetComponent` è composto da:

*   **Floating Button**: Un pulsante circolare con il logo Asor Core, posizionato in basso a destra (`position: fixed`). Cliccandolo si apre/chiude il pannello.
*   **Pannello a Scomparsa (Slider)**: Un pannello che occupa l'80% dell'altezza dello schermo, animato dal basso verso l'alto. Contiene un header con tab di navigazione e un'area di contenuto.
*   **Tab di Navigazione**: Due tab principali — **Console** e **Storage** — che permettono di alternare tra le viste disponibili.

### Dettagli Tecnici

| Proprietà | Valore |
|---|---|
| **Selector** | `asor-core-widget` |
| **Encapsulation** | `ViewEncapsulation.ShadowDom` |
| **Standalone** | `true` |
| **Modulo** | `asor-core` |

> L'utilizzo di `ShadowDom` garantisce che gli stili del widget siano completamente isolati dall'applicazione host, evitando conflitti CSS.

### Codice Sorgente

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

### Proprietà di Input

| Proprietà | Tipo | Default | Descrizione |
|---|---|---|---|
| `mode` | `'off' \| 'on' \| 'spy'` | `'on'` | Controlla la modalità di visibilità del widget |

#### Valori di Mode

*   **`on`** — Il widget è sempre visibile e funzionante (comportamento predefinito).
*   **`off`** — Il widget è completamente nascosto e disabilitato.
*   **`spy`** — Il widget è nascosto di default. Premere **`CTRL + I`** per attivare/disattivare la sua visibilità. Utile in ambienti di produzione dove il widget deve rimanere invisibile a meno che non venga attivato esplicitamente dallo sviluppatore.

---

## Come Usarlo <a id="widget-usage"></a>

Per integrare il widget nella propria applicazione, è sufficiente:

### 1. Importare il Componente

Aggiungere `AsorWidgetComponent` negli imports del componente root (o di qualsiasi componente che funga da layout principale).

```typescript
import { AsorWidgetComponent } from '@asor-studio/asor-core';

@Component({
    selector: 'app-root',
    standalone: true,
    imports: [
        // ... altri imports
        AsorWidgetComponent
    ],
    template: `
        <router-outlet></router-outlet>
        <asor-core-widget mode="on"></asor-core-widget>
    `
})
export class AppRoot { }
```

### 2. Aggiungere il Tag nel Template

Inserire il selettore `<asor-core-widget>` nel template HTML, idealmente come ultimo elemento del layout principale.

```html
<!-- Sempre visibile -->
<asor-core-widget mode="on"></asor-core-widget>

<!-- Completamente nascosto -->
<asor-core-widget mode="off"></asor-core-widget>

<!-- Nascosto fino alla pressione di CTRL + I -->
<asor-core-widget mode="spy"></asor-core-widget>
```

> **Nota**: Il widget si posiziona automaticamente in basso a destra grazie al `position: fixed`. Non è necessario aggiungere stili aggiuntivi.
>
> In modalità `spy`, premendo nuovamente **`CTRL + I`** il widget verrà nascosto e il pannello chiuso se aperto.

> [!TIP]
> Puoi provare la modalità `spy` adesso! Vai alla sezione **Archiviazione Dati** di questa wiki e premi **`CTRL + I`** per rivelare il widget nascosto.

---

## Tab Console <a id="widget-console"></a>

Il tab **Console** fornisce un flusso di log in tempo reale proveniente dal sistema `ConsoleLogsUtility` di `asor-core`. Permette di monitorare l'attività dell'applicazione senza aprire gli strumenti di sviluppo del browser.

![Tab Console con flusso di log in tempo reale](/assets/docs/images/widget-console-tab.png)

### Funzionalità

*   **LOGS STREAM**: Visualizzazione in tempo reale di tutti i log emessi dall'applicazione, colorati per livello (DEBUG, INFO, WARNING, ERROR).
*   **Filtri per Livello**: Badge filtro per tipo — `INFO`, `DEBUG`, `WARNING` — con contatori per ogni categoria.
*   **Barra di Ricerca**: Ricerca testuale nei log (`Search logs...`).
*   **Configurazione per Classe**: Pannello laterale sinistro (**CONFIGURATION**) che permette di impostare il livello di log per ogni classe/servizio registrato:
    *   `StateService`
    *   `AuthGuard`
    *   `CacheInterceptor`
    *   `ErrorInterceptor`
    *   `NotifyErrorService`
    *   `HttpRequestHandler`
*   **Pulsante Clear**: Per svuotare il flusso dei log correnti (`X Clear`).

---

## Tab Storage <a id="widget-storage"></a>

Il tab **Storage** offre una vista ispezionabile dello stato dello storage (`StateService`) dell'applicazione. Mostra i dati contenuti nei DataSet registrati e il registro dei componenti collegati.

![Tab Storage con Data Containers e Component Registry](/assets/docs/images/widget-storage-tab.png)

### Funzionalità

*   **Indicatori di Stato**: Barra superiore con informazioni chiave sullo storage:
    *   `INITIALIZED` — Stato di inizializzazione (YES/NO)
    *   `SYNC INTERVAL` — Stato del sincronizzatore (Active/Inactive)
    *   `ENCRYPTION` — Tipo di crittografia utilizzata (es. AES)
    *   `DATA ELEMENTS` — Numero di elementi nello storage
    *   `REGISTRY ENTRIES` — Numero di componenti registrati
*   **Data Containers**: Pannello sinistro che elenca i DataSet registrati con:
    *   Nome del container e tipo (es. `VOLATILE`)
    *   Dati crittografati (visualizzati in forma hash)
    *   Timestamp ultimo aggiornamento
    *   Azioni rapide: visualizza, copia, esporta, elimina
*   **Component Registry**: Tabella a destra che mostra i componenti e le molecole collegati allo storage con:
    *   `Component` — Nome della classe registrata
    *   `Prop` — Proprietà collegata (es. `title`, `form`)
    *   `Path` — Percorso di collegamento al DataSet
