# <a id="storage-comp-intro"></a>Storage Example Component

This section demonstrates a practical implementation of a component that extends `BaseStorageComponent`. It showcases how to connect to the application state using `IExampleProps`, how to use data binding and interpolation to display data from the store, and how to utilize Angular directives like `*ngIf` and `*ngFor` to render lists and conditionally display elements based on the state. Importantly, all information is retrieved reactively from the inherited `this.props` variable.

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


### <a id="storage-comp-inheritance"></a>Inheritance and Props (`BaseStorageComponent`)

Similar to molecules, the `ExampleStorageComponent` class extends `BaseStorageComponent<IExampleProps>`. This inheritance is fundamental to gain typed access to the application state.

#### 1. Component Identification (`className`)
Every class extending `BaseStorageComponent` (or `BaseComponent`) **must** declare a static `className` property:

```typescript
public static override readonly className: string = 'ExampleStorageComponent';
```

This property is used by the framework to match the component to its route configuration (`ConnectDataSet`, `I18nPath`) and to identify the class in console logs. It must be `static` so it is available on the class constructor before instantiation. Omitting it will cause a TypeScript compilation error when the class is used in route configuration.

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
The data populating `props` is defined in the `ExampleConnectDataSet` configuration (in `example.storage.const.ts`) which is associated with the component via Router configuration.
The `BaseStorageComponent` takes care of "listening" to state changes and automatically updating the `props` variable when data changes.

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

Thanks to this mechanism, there is no need to write boilerplate logic to subscribe to observables or manage view updates: everything happens automatically via inheritance.
