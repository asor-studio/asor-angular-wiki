# <a id="storage-comp-intro"></a>Storage Example Component

Questa sezione dimostra un'implementazione pratica di un componente che estende `BaseStorageComponent`. Viene mostrato come connettersi allo stato dell'applicazione utilizzando `IExampleProps`, come utilizzare il data binding e l'interpolazione per visualizzare i dati dallo store e come utilizzare le direttive Angular come `*ngIf` e `*ngFor` per renderizzare liste e visualizzare condizionalmente elementi in base allo stato. Le informazioni vengono recuperate in modalità reactive dalla variabile ereditata `this.props`.

## <a id="storage-comp-html"></a>HTML

```html
<div class="macos-window component-window">
    <!-- macOS Title Bar -->
    <div class="macos-titlebar">
        <div class="traffic-lights">
            <span class="light red"></span>
            <span class="light yellow"></span>
            <span class="light green"></span>
        </div>
        <span class="titlebar-label">📋 Component Example Storage</span>
    </div>

    <!-- Display Content -->
    <div class="macos-content">
        <!-- Interpolated Title -->
        <div class="title-banner">
            <span class="title-icon">👋</span>
            <span class="title-text">{{ interpolatedTitle }}</span>
        </div>

        <!-- Data Table -->
        <div class="data-section">
            <div class="data-row">
                <span class="data-label">Name</span>
                <span class="data-value">{{ props.form.name }}</span>
            </div>
            <div class="data-row">
                <span class="data-label">Age</span>
                <span class="data-value">{{ props.form.age }}</span>
            </div>
            <div class="data-row" *ngIf="selectedJobRole">
                <span class="data-label">Job Role</span>
                <span class="data-badge">{{ selectedJobRole.title }}</span>
            </div>
        </div>

        <!-- All Job Roles -->
        <div class="roles-section" *ngIf="props.form.jobRole.length">
            <span class="section-label">Available Roles</span>
            <div class="roles-list">
                <div class="role-card" *ngFor="let role of props.form.jobRole" [class.selected]="role.selected">
                    <span class="role-title">{{ role.title }}</span>
                    <span class="role-desc">{{ role.description }}</span>
                </div>
            </div>
        </div>

    </div>
</div>
```

## <a id="storage-comp-ts"></a>TypeScript

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { BaseStorageComponent, TranslatePipe } from '@asor-studio/asor-core';
import { IExampleProps } from '../../storages/example.storage.const';

@Component({
    selector: 'asor-example-storage-component',
    standalone: true,
    imports: [CommonModule],
    templateUrl: './example-storage.component.html',
    styleUrls: ['./example-storage.component.scss']
})
export class ExampleStorageComponent extends BaseStorageComponent<IExampleProps> {
    public static override readonly className: string = 'ExampleStorageComponent';

    get interpolatedTitle(): string {
        if (!this.props?.title || !this.props?.form) {
            return '';
        }
        return this.props.title
            .replace('{name}', this.props.form.name || '')
            .replace('{age}', String(this.props.form.age || 0));
    }

    get selectedJobRole(): { title: string; description: string; selected: boolean } | null {
        if (!this.props?.form?.jobRole) return null;
        return this.props.form.jobRole.find(r => r.selected) || this.props.form.jobRole[0] || null;
    }
}
```


### <a id="storage-comp-inheritance"></a>Ereditarietà e Props (`BaseStorageComponent`)

Analogamente a quanto avviene per le molecole, la classe `ExampleStorageComponent` estende `BaseStorageComponent<IExampleProps>`. Questa ereditarietà è fondamentale per ottenere l'accesso tipizzato allo stato applicativo.

#### 1. Identificazione del Componente (`className`)
Ogni classe che estende `BaseStorageComponent` (o `BaseComponent`) **deve** dichiarare una proprietà statica `className`:

```typescript
public static override readonly className: string = 'ExampleStorageComponent';
```

Questa proprietà viene utilizzata dal framework per associare il componente alla sua configurazione nelle rotte (`ConnectDataSet`, `I18nPath`) e per identificare la classe nei log della console. Deve essere `static` affinché sia disponibile sul costruttore della classe prima dell'istanziazione. Ometterla causerà un errore di compilazione TypeScript quando la classe viene utilizzata nella configurazione delle rotte.

#### 2. Tipizzazione delle Props
Passando l'interfaccia `IExampleProps` come tipo generico (`<IExampleProps>`), il componente eredita automaticamente una proprietà `props` tipizzata correttamente.
```typescript
// definition in example.storage.const.ts
export interface IExampleProps {
    title: string;
    form: { ... };
}

// usage in component
this.props.title // Type: string
this.props.form  // Type: { ... }
```

#### 3. Connessione ai Dati (`ConnectDataSet`)
I dati che popolano `props` vengono definiti nella configurazione `ExampleConnectDataSet` (in `example.storage.const.ts`) che viene associata al componente tramite la configurazione del Router.
Il `BaseStorageComponent` si occupa di "ascoltare" le modifiche allo stato e aggiornare automaticamente la variabile `props` quando i dati cambiano.

```typescript
// definition in example.storage.const.ts
export const ExampleConnectDataSet: IConnectDataSet = {
    selectors: {
        title: 'example-component.title', // Maps store 'example-component.title' -> props.title
        form: 'example-component.form'   // Maps store 'example-component.form'  -> props.form
    }
};

// usage in app.routes.ts
{
    Component: ExampleStorageComponent,
    ConnectDataSet: ExampleConnectDataSet // Connects data to component
    ...
}
```

Grazie a questo meccanismo, non è necessario scrivere logica boilerplate per sottoscriversi agli observable o gestire l'aggiornamento della view: tutto avviene automaticamente tramite l'ereditarietà.

