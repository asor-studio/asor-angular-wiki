# Asor Widget (`AsorWidgetComponent`)

L'`AsorWidgetComponent` e` il punto in cui `asor-core` mostra il suo lato piu` pratico durante sviluppo e debugging. Non e` solo un pulsante flottante: e` una finestra rapida sullo stato interno dell'applicazione.

L'idea e` semplice: mentre lavori su route, dataset, log e cache, non dovresti essere costretto ogni volta ad abbandonare la pagina per aprire strumenti esterni o ricostruire manualmente cosa sta succedendo.

![Pulsante floating del widget visibile nell'angolo inferiore destro](assets/docs/images/widget-storage-page.png)

---

## Panoramica <a id="widget-overview"></a>

Il widget e` composto da tre elementi principali:

| Parte | Ruolo |
|---|---|
| Floating button | Apre e chiude il pannello |
| Slider panel | Contiene strumenti e viste di diagnostica |
| Tab interne | Separano log e storage |

### Dettagli tecnici

| Proprietà | Valore |
|---|---|
| Selector | `asor-core-widget` |
| Encapsulation | `ViewEncapsulation.ShadowDom` |
| Standalone | `true` |
| Modulo | `asor-core` |

L'uso di `ShadowDom` e` importante perche` isola gli stili del widget dal resto dell'applicazione host.

### Modalita` disponibili

| `mode` | Comportamento |
|---|---|
| `on` | Widget sempre visibile |
| `off` | Widget disabilitato e nascosto |
| `spy` | Widget nascosto finche` non premi `CTRL + I` |

---

## Come usarlo <a id="widget-usage"></a>

Il modo corretto di pensare il widget e` questo: va inserito nel layout principale, come strumento trasversale all'applicazione.

### Dove inserirlo

| Punto | Suggerimento |
|---|---|
| Root component | Posizione ideale |
| Layout principale | Ottima alternativa |
| Ultimo elemento del template | Consigliato per chiarezza strutturale |

### Esempio

```typescript
import { AsorWidgetComponent } from '@asor-studio/asor-core';

@Component({
    selector: 'app-root',
    standalone: true,
    imports: [AsorWidgetComponent],
    template: `
        <router-outlet></router-outlet>
        <asor-core-widget mode="on"></asor-core-widget>
    `
})
export class AppRoot { }
```

### Esempi di utilizzo nel template

```html
<asor-core-widget mode="on"></asor-core-widget>
<asor-core-widget mode="off"></asor-core-widget>
<asor-core-widget mode="spy"></asor-core-widget>
```

### Nota pratica

Non serve aggiungere CSS custom per posizionarlo: il widget e` gia` progettato per stare in basso a destra con `position: fixed`.

---

## Tab Console <a id="widget-console"></a>

La tab **Console** e` utile quando vuoi capire cosa sta accadendo senza aprire ogni volta i DevTools del browser.

Mostra i log emessi dal sistema ASOR e dalle classi registrate, con livelli e filtri leggibili.

![Tab Console con flusso di log in tempo reale](assets/docs/images/widget-console-tab.png)

### Funzionalità principali

| Funzionalita` | Descrizione |
|---|---|
| Log stream | Flusso live dei log |
| Filtri per livello | Selezione per `INFO`, `DEBUG`, `WARNING`, `ERROR` |
| Ricerca | Cerca testo nei log |
| Configurazione per classe | Regola i livelli per classi e servizi registrati |
| Clear | Pulisce il flusso corrente |

### Perche` e` utile

E` particolarmente comoda quando stai lavorando su:

- bootstrap dell'app
- route guard
- cache interceptor
- gestione errori
- dataset e collegamenti storage-aware

---

## Tab Storage <a id="widget-storage"></a>

La tab **Storage** e` la parte piu` preziosa quando lavori con `StateService`. Ti permette di vedere se i dataset sono stati creati, se i componenti sono registrati e se la sincronizzazione sta avvenendo come previsto.

![Tab Storage con Data Containers e Component Registry](assets/docs/images/widget-storage-tab.png)

### Cosa mostra

| Sezione | Contenuto |
|---|---|
| Indicatori di stato | Stato globale dello storage |
| Data Containers | Elenco dei dataset presenti |
| Component Registry | Blocchi UI collegati allo storage |

### Indicatori principali

| Indicatore | Significato |
|---|---|
| `INITIALIZED` | Dice se `StateService` e` stato inizializzato |
| `SYNC INTERVAL` | Mostra lo stato del sincronizzatore |
| `ENCRYPTION` | Tipo di cifratura usata |
| `DATA ELEMENTS` | Numero di dataset o elementi presenti |
| `REGISTRY ENTRIES` | Numero di componenti registrati |

### Data Containers

Qui puoi vedere per ogni dataset:

| Campo | Significato |
|---|---|
| Nome container | Nome del dataset |
| Tipo | `VOLATILE`, `SESSION`, `LOCAL` |
| Dati | Valore o rappresentazione cifrata |
| Timestamp | Ultimo aggiornamento |
| Azioni rapide | Visualizza, copia, esporta, elimina |

### Component Registry

Questa tabella aiuta a capire quali blocchi stanno leggendo lo stato e come sono collegati.

| Colonna | Significato |
|---|---|
| `Component` | Nome della classe registrata |
| `Prop` | Proprietà collegata |
| `Path` | Path del dataset usato dal binding |

---

## Quando usarlo davvero

Il widget da` il meglio quando stai controllando problemi come:

| Problema sospetto | Cosa guardare |
|---|---|
| Traduzioni che non compaiono | Bootstrap e log in Console |
| Dataset che non si aggiornano | Tab Storage e registry |
| Route che sembrano corrette ma non iniettano props | Component Registry |
| Bootstrap incompleto | `INITIALIZED`, log e presenza dei dataset |

In pratica, il widget e` un buon ponte tra documentazione, runtime e debugging reale: ti permette di vedere se l'applicazione si sta comportando come `asor-core` si aspetta.
