# <a id="storage-mol-intro"></a>Storage Example Molecule

Questa molecola rappresenta un esempio pratico di come implementare un componente che interagisce in lettura e scrittura con lo storage (DataSet).

Nelle sezioni successive verranno illustrati i seguenti aspetti implementativi:
1.  **HTML**: Viene mostrato come collegare gli elementi della UI (input, button) direttamente ai dati dello storage tramite la proprietà `props`. Grazie al *two-way binding* di Angular (`[(ngModel)]`), le modifiche apportate dall'utente vengono riflettute immediatamente nello stato.
2.  **TypeScript**: Viene evidenziata la semplicità della classe, che necessita di pochissima logica grazie all'estensione della classe base.
3.  **Ereditarietà e Props**: Viene approfondito il funzionamento di `BaseStorageMolecule`, spiegando come garantisce l'accesso tipizzato ai dati e la sincronizzazione automatica con il `ConnectDataSet` definito.

## <a id="storage-mol-html"></a>HTML

```html
<div class="macos-window molecule-window">
    <!-- macOS Title Bar -->
    <div class="macos-titlebar">
        <div class="traffic-lights">
            <span class="light red"></span>
            <span class="light yellow"></span>
            <span class="light green"></span>
        </div>
        <span class="titlebar-label">📝 Molecule Example Storage</span>
    </div>

    <!-- Form Content -->
    <div class="macos-content">
        <div class="form-group">
            <label class="macos-label" for="mol-name">Name</label>
            <input class="macos-input" id="mol-name" type="text" [(ngModel)]="props.form.name"
                placeholder="Enter your name..." />
        </div>

        <div class="form-group">
            <label class="macos-label" for="mol-age">Age</label>
            <input class="macos-input" id="mol-age" type="number" [(ngModel)]="props.form.age"
                placeholder="Enter your age..." min="0" max="150" />
        </div>

        <div class="form-group">
            <label class="macos-label" for="mol-age">Job Role</label>
            <div class="job-role-chips">
                <button *ngFor="let role of props.form.jobRole; let i = index" class="macos-chip"
                    [class.active]="role.selected" (click)="onJobRoleChange(role)">
                    <span class="chip-icon">💼</span>
                    <span class="chip-text">{{ role.title }}</span>
                </button>
            </div>
        </div>
    </div>
</div>
```

## <a id="storage-mol-ts"></a>TypeScript

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { BaseStorageMolecule, TranslatePipe } from '@asor-studio/asor-core';
import { IExampleProps } from '../../storages/example.storage.const';

@Component({
    selector: 'asor-example-storage-molecule',
    standalone: true,
    imports: [CommonModule, FormsModule],
    templateUrl: './example-storage.molecule.html',
    styleUrls: ['./example-storage.molecule.scss']
})
export class ExampleStorageMolecule extends BaseStorageMolecule<IExampleProps> {
    public static override readonly className: string = 'ExampleStorageMolecule';

    onJobRoleChange(role: { title: string; description: string; selected: boolean }): void {
        role.selected = !role.selected;
    }
}
```

### <a id="storage-mol-inheritance"></a>Ereditarietà e Props (`BaseStorageMolecule`)

La classe `ExampleStorageMolecule` estende `BaseStorageMolecule<IExampleProps>`. Questa ereditarietà è fondamentale per il funzionamento del sistema di storage.

#### 1. Identificazione del Componente (`className`)
Ogni classe che estende `BaseStorageMolecule` (o `BaseMolecule`) **deve** dichiarare una proprietà statica `className`:

```typescript
public static override readonly className: string = 'ExampleStorageMolecule';
```

Questa proprietà viene utilizzata dal framework per associare la molecola alla sua configurazione nelle rotte (`ConnectDataSet`, `I18nPath`) e per identificare la classe nei log della console. Deve essere `static` affinché sia disponibile sul costruttore della classe prima dell'istanziazione. Ometterla causerà un errore di compilazione TypeScript quando la classe viene utilizzata nella configurazione delle rotte.

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
I dati che popolano `props` vengono definiti nella configurazione `ExampleConnectDataSet` (in `example.storage.const.ts`).
Il `BaseStorageMolecule` si occupa di "ascoltare" le modifiche allo stato e aggiornare automaticamente la variabile `props` quando i dati cambiano.

```typescript
// definition in example.storage.const.ts
export const ExampleConnectDataSet: IConnectDataSet = {
    selectors: {
        title: 'example-component.title', // Maps store 'example-component.title' -> props.title
        form: 'example-component.form'   // Maps store 'example-component.form'  -> props.form
    }
};
```

Grazie a questo meccanismo, non è necessario scrivere logica boilerplate per sottoscriversi agli observable o gestire l'aggiornamento della view: tutto avviene automaticamente tramite l'ereditarietà.

