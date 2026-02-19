# <a id="storage-mol-intro"></a>Storage Example Molecule

This molecule represents a practical example of how to implement a component that interacts in read and write mode with storage (DataSet).

The following implementation aspects will be illustrated in the next sections:
1.  **HTML**: Shows how to bind UI elements (input, button) directly to storage data via the `props` property. Thanks to Angular's *two-way binding* (`[(ngModel)]`), user changes are immediately reflected in the state.
2.  **TypeScript**: Highlights the simplicity of the class, which requires very little logic thanks to base class extension.
3.  **Inheritance and Props**: Delves into how `BaseStorageMolecule` works, explaining how it ensures typed access to data and automatic synchronization with the defined `ConnectDataSet`.

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

### <a id="storage-mol-inheritance"></a>Inheritance and Props (`BaseStorageMolecule`)

The `ExampleStorageMolecule` class extends `BaseStorageMolecule<IExampleProps>`. This inheritance is fundamental for the storage system operation.

#### 1. Component Identification (`className`)
Every class extending `BaseStorageMolecule` (or `BaseMolecule`) **must** declare a static `className` property:

```typescript
public static override readonly className: string = 'ExampleStorageMolecule';
```

This property is used by the framework to match the molecule to its route configuration (`ConnectDataSet`, `I18nPath`) and to identify the class in console logs. It must be `static` so it is available on the class constructor before instantiation. Omitting it will cause a TypeScript compilation error when the class is used in route configuration.

#### 2. Props Typing
By passing the `IExampleProps` interface as a generic type (`<IExampleProps>`), the component automatically inherits a correctly typed `props` property.

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

#### 3. Data Connection (`ConnectDataSet`)
The data populating `props` is defined in the `ExampleConnectDataSet` configuration (in `example.storage.const.ts`).
The `BaseStorageMolecule` takes care of "listening" to state changes and automatically updating the `props` variable when data changes.

```typescript
// definition in example.storage.const.ts
export const ExampleConnectDataSet: IConnectDataSet = {
    selectors: {
        title: 'example-component.title', // Maps store 'example-component.title' -> props.title
        form: 'example-component.form'   // Maps store 'example-component.form'  -> props.form
    }
};
```

Thanks to this mechanism, there is no need to write boilerplate logic to subscribe to observables or manage view updates: everything happens automatically via inheritance.
