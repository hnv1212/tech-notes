# (change) vs (ngModelChange) in angular

- `(change)` event bound to classical input change event.
- You can use (change) event even if you don't have a model at your input as 
```html
<input (change)="somethingChanged()">
```

- `(ngModelChange)` is the `@Output` of ngModel directive. It fires when the model changes. You cannot use this event without ngModel directive.
- As you discover more in the source code, (ngModelChange) emits the new values. (https://github.com/angular/angular/blob/master/packages/forms/src/directives/ng_model.ts#L169)
- So it means you have ability of such usage:
```html
<input (ngModelChange)="modelChanged($event)">
``` 
```typescript
modelChanged(newObj) {
    // do something with the new value
}
```

Basically, it seems like there is no big difference between two, but `ngModel` events gains the power when you use `[ngValue]`.
```html
<select [(ngModel)]="data" (ngModelChange)="dataChanged($event)" name="data">
    <option *ngFor="let currentData of allData" [ngValue]="currentData">{{data.name}}</option>
</select>
```
```typescript
dataChanged(newObj) {
    // here comes the object as parameter
}
```

assume you try the same thing without "ngModel"
```html
<select (change)="changed($event)">
    <option *ngFor="let currentData of allData" [value]="currentData.id">
        {{data.name}}
    </option>
</select>
```
```typescript
changed(e){
    // event comes as parameter, you'll have to find selectedData manually
    // by using e.target.data
}
```

