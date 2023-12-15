# Angular reactive form validation with FormBuilder

Reactive forms in Angular enable you to build clean forms without using too many directives. This is critical because:
- Javascript frameworks typically caution against using clustered templates
- Form logic now lies in the component class

### Form controls and form groups in Angular
Form controls are classes that can hold both the data values and the validation information of any form element, which means every form input you have in a reactive form should be bound by a form control. There are the basic units that make up reactive forms.

`FormControl` is a class in Angular that tracks the value and validation status of an individual form control. One of the three essential building blocks in Angular forms — along with `FormGroup` and `FormArray` — `FormControl` extends the `AbstractControl` class, which enables it to access the value, validation status, user interactions, and events.

Form groups are construct that basically wrap a collection of form controls. Just as control give you access to the state of an element, the group gives the same access, but to the state of the wrapped controls. Every single form control in the form group is identified by name when initializing.

FormGroup is used with FormControl to track the value and validate the state of form control. In practice, FormGroup aggregates the values of each child FormControl into a single object, using each control name as the key. It calculates its status by reducing the status values of its children so that if one control in a group is invalid, the entire group is rendered invalid.

### What is form validation in Angular?
Form validation in Angular enables you to verify that the input is accurate and complete. You can validate user input from the UI and display helpful validation messages in both template-driven and reactive forms.

When validating reactive forms in Angular, validator functions are added directly to the form control model in the component class. Angular calls these functions whenever the value of the control changes.

Validator functions can be either synchronous or asynchronous:
- Synchronous validators take a control instance and return either a set of errors or null. When you create a FormControl, you can pass sync functions in as the second argument.
- Asynchronous validators take a control instance and return a Promise or an Observable that later issues either a set of errors or null. You can pass async functions in as the third argument when you instantiate a FormControl.

Depending on the unique needs and goals of your project, you may choose to either write custom validator functions or use any of Angular’s built-in validators.

### What is FormBuilder?
Setting up a form controls, especially for very long forms, can quickly become both monotonous and stressful. FormBuilder in Angular helps you streamline the process of building complex forms while avoiding repetition.

Put simply, FormBuilder provides syntactic sugar that eases the burden of creating instances of FormControl, FormGroup, or FormArray and reduces the amount of boilerplate required to build complex forms.

### How to use FormBuilder
employee.component.ts:
```typescript
import { Component, OnInit } from '@angular/core';
import { FormControl, FormGroup } from '@angular/form';

@Component({
    selector: 'app-employee',
    templateUrl: './employee.component.html',
    styleUrls: ['./employee.component.css']
})
export class EmployeeComponent implements OnInit {
    bioSection = new FormGroup({
        firstName: new FormControl(''),
        lastName: new FormControl(''),
        age: new FormControl(''),
        stackDetails: new FormGroup({
            stack: new FormControl(''),
            experience: new FormControl('')
        }),
        address: new FormGroup({
            country: new FormControl(''),
            city: new FormControl('')
        })
    })

    constructor() {}

    ngOnInit() {}

    callingFunction() {
        console.log(this.bioSection.value);
    }
}
```
You can see that every single form control — and even the form group that partitions it — is spelled out, so over time, you end up repeating yourself. FormBuilder helps solve this efficiency problem.

### Registering FormBuilder
To use FormBuilder, you must first register it. To register FormBuilder in a component, import it from Angular forms:
```typescript
import { FormBuilder } from '@angular/forms';
```

The next step is to inject the form builder service, which is an injectable provider that comes with the reactive forms module. You can then use the form builder after injecting it. Navigate to the employee.component.ts:
```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder } from '@angular/forms';

@Component({
  selector: 'app-employee',
  templateUrl: './employee.component.html',
  styleUrls: ['./employee.component.css']
})
export class EmployeeComponent implements OnInit {
    bioSection = this.fb.group({
        firstName: [''],
        lastName: [''],
        age: [''],
        stackDetails: this.fb.group({
            stack: [''],
            experience: ['']
        }),
        address: this.fb.group({
            country: [''],
            city: ['']
        })
    })

    constructor(private fb: FormBuilder) {}

    ngOnInit() {}

    callingFunction() {
        console.log(this.bioSection.value);
    }
}
```
This does exactly the same thing as the previous code block you saw at the start, but you can see there is a lot less code and more structure — and, thus, optimal usage of resources. Form builders not only help to make your reactive forms’ code efficient, but they are also important for form validation.

### How to validate forms in Angular
The first thing to do, as with all elements of reactive forms, is to import it from Angular forms.
```typescript
import { Validators } from '@angular/forms';
```

You can now play around with the validators by specifying the form controls that must be filled in order for the submit button to be active.
```typescript
// ...
bioSection = this.fb.group({
    firstName: ['', Validators.required],
    lastName: [''],
    age: [''],
    stackDetails: this.fb.group({
      stack: [''],
      experience: ['']
    }),
    address: this.fb.group({
        country: [''],
        city: ['']
    })
});
```

The last thing to do is to make sure the submit button’s active settings are set accordingly. Navigate to the employee.component.html
```html
<button type="submit" [disabled]="!bioSection.valid">Submit Application</button>
```

### Displaying input values and status
employee.component.html:
```html
<form [formGroup]="bioSection" (ngSubmit)="callingFunction()">
    <h3>Bio Details</h3>

    <label>First Name: 
        <input type="text" formControlName="firstName">
    </label>
    <br>
    <label>Last Name: 
        <input type="text" formControlName="lastName">
    </label>
    <label>Age 
        <input type="text" formControlName="age">
    </label>
    
    <div formGroupName="stackDetails">
        <h3>Stack Details</h3>
        <label>
            Stack:
            <input type="text" formControlName="stack">
        </label> <br>

        <label>
            Experience:
            <input type="text" formControlName="experience">
        </label>
    </div>

    <div formGroupName="address">
        <h3>Address</h3>
        <label>
            Country:
            <input type="text" formControlName="country">
        </label>
        <label>
            City:
            <input type="text" formControlName="city">
        </label>
    </div>
    <button type="submit" [disabled]="!bioSection.valid">Submit Application</button>
    <p>
        Real-time data: {{ bioSection.value | json }}
    </p>
    <p>
        Your form status is : {{ bioSection.status }}
    </p>
</form>
```