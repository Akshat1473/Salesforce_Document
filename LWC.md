# ⚡ LWC Complete Cheatsheet — Lightning Web Components

> A deep-dive reference for learning and practicing LWC. Covers structure, templates, JS, data communication, lifecycle, wire, Apex, forms, tables, and more — with HTML parallels.

---

## 📚 Table of Contents

1. [LWC Component Structure](#1-lwc-component-structure)
2. [HTML Templates — Basics](#2-html-templates--basics)
3. [Template Directives — Conditionals & Loops](#3-template-directives--conditionals--loops)
4. [JavaScript — Component Class](#4-javascript--component-class)
5. [Decorators — @api, @track, @wire](#5-decorators)
6. [Lifecycle Hooks](#6-lifecycle-hooks)
7. [Event Handling](#7-event-handling)
8. [Data Communication Patterns](#8-data-communication-patterns)
   - [Parent → Child via @api](#pattern-1-parent--child-via-api)
   - [Child → Parent via Custom Events](#pattern-2-child--parent-via-custom-events)
   - [Sibling via LMS](#pattern-3-sibling-communication-via-lightning-message-service-lms)
   - [Parent calling Child methods](#pattern-4-parent-calling-child-methods)
   - [Bubbling Events](#pattern-5-bubbling-events)
   - [Pub/Sub Service](#pattern-6-pubsub-utility-service)
9. [Wire Service & Apex](#9-wire-service--apex)
10. [Forms & Validation](#10-forms--validation)
11. [Tables in LWC](#11-tables-in-lwc)
12. [CSS & Styling](#12-css--styling)
13. [Slots — Content Projection](#13-slots--content-projection)
14. [Navigation Service](#14-navigation-service)
15. [Salesforce-Specific Features](#15-salesforce-specific-features)
16. [Best Practices](#16-best-practices)
17. [Practice Projects](#17-practice-projects)

---

## 1. LWC Component Structure

Every LWC component lives in its own folder. The folder name = component name (camelCase → kebab-case when used in HTML).

```
force-app/
└── main/
    └── default/
        └── lwc/
            └── myComponent/
                ├── myComponent.html          ← Template
                ├── myComponent.js            ← Controller
                ├── myComponent.css           ← Scoped styles
                ├── myComponent.js-meta.xml   ← Metadata config
                └── myComponent.svg           ← Custom icon (optional)
```

### myComponent.js-meta.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>59.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__AppPage</target>
        <target>lightning__RecordPage</target>
        <target>lightning__HomePage</target>
        <target>lightning__Tab</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordPage">
            <property name="recordId" type="String" label="Record ID"/>
            <property name="title" type="String" label="Component Title" default="My Component"/>
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

> **HTML Parallel**: The `.html` file = HTML `<body>`, `.js` = JavaScript, `.css` = CSS, `.js-meta.xml` = configuration (like `<meta>` tags in `<head>`).

---

## 2. HTML Templates — Basics

Every LWC template must have a **single root `<template>` tag** — similar to how HTML has a single `<body>`.

### Text Binding (like HTML text nodes)

```html
<!-- myComponent.html -->
<template>
    <!-- Headings equivalent: h1-h6 -->
    <h1>{mainTitle}</h1>
    <h2>{subTitle}</h2>

    <!-- Paragraph equivalent -->
    <p>{description}</p>

    <!-- Bold / Italic (use CSS or lightning components) -->
    <p><strong>{importantText}</strong></p>
    <p><em>{emphasizedText}</em></p>

    <!-- Using a computed getter -->
    <p>{fullName}</p>

    <!-- Data attribute equivalent -->
    <div data-id={item.id} data-type={item.type}>
        {item.name}
    </div>
</template>
```

```javascript
// myComponent.js
import { LightningElement } from 'lwc';

export default class MyComponent extends LightningElement {
    mainTitle = 'Welcome to LWC';
    subTitle = 'Salesforce Lightning Web Components';
    description = 'A modern framework for building Salesforce UIs';
    importantText = 'Read this carefully';
    emphasizedText = 'This is emphasized';
    firstName = 'John';
    lastName = 'Doe';

    item = { id: '001', type: 'contact', name: 'Alice' };

    // Getter = computed property (like a derived HTML text value)
    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }
}
```

### Links (Anchor equivalent)

```html
<template>
    <!-- External link -->
    <a href="https://developer.salesforce.com" target="_blank" rel="noopener noreferrer">
        Salesforce Docs
    </a>

    <!-- Internal navigation — use NavigationMixin instead of href -->
    <button onclick={navigateToRecord}>Go to Record</button>
</template>
```

### Images

```html
<template>
    <!-- Static resource image -->
    <img src={logoUrl} alt="Company Logo" />

    <!-- With lightning-icon (Salesforce icons) -->
    <lightning-icon icon-name="standard:account" size="medium" alternative-text="Account"></lightning-icon>
</template>
```

```javascript
import { LightningElement } from 'lwc';
import LOGO from '@salesforce/resourceUrl/companyLogo';

export default class MyComponent extends LightningElement {
    logoUrl = LOGO;
}
```

### Horizontal Rule / Dividers

```html
<template>
    <p>Section One</p>
    <hr />
    <p>Section Two</p>

    <!-- Or use lightning-divider -->
    <lightning-layout>
        <lightning-layout-item>Content A</lightning-layout-item>
    </lightning-layout>
</template>
```

---

## 3. Template Directives — Conditionals & Loops

### Conditionals — lwc:if / lwc:elseif / lwc:else (API 56+) ✅ Recommended

```html
<template>
    <template lwc:if={isAdmin}>
        <p>Admin Panel — Full Access</p>
        <button onclick={deleteRecord}>Delete</button>
    </template>

    <template lwc:elseif={isEditor}>
        <p>Editor Panel — Edit Access</p>
    </template>

    <template lwc:else>
        <p>Viewer — Read Only</p>
    </template>
</template>
```

```javascript
import { LightningElement } from 'lwc';

export default class ConditionalDemo extends LightningElement {
    userRole = 'admin';

    get isAdmin()  { return this.userRole === 'admin'; }
    get isEditor() { return this.userRole === 'editor'; }
}
```

### Conditionals — if:true / if:false (Legacy, API < 56)

```html
<template>
    <template if:true={isLoading}>
        <lightning-spinner alternative-text="Loading..." size="medium"></lightning-spinner>
    </template>

    <template if:false={isLoading}>
        <p>Content loaded!</p>
    </template>
</template>
```

### for:each Loop (like HTML `<ul><li>` with data)

```html
<template>
    <!-- Unordered list equivalent -->
    <ul>
        <template for:each={fruits} for:item="fruit" for:index="index">
            <li key={fruit.id}>
                {index}: {fruit.name} — {fruit.color}
            </li>
        </template>
    </ul>

    <!-- Card list -->
    <div class="card-container">
        <template for:each={products} for:item="product">
            <div key={product.id} class="card">
                <h3>{product.name}</h3>
                <p>{product.price}</p>
            </div>
        </template>
    </div>
</template>
```

```javascript
import { LightningElement } from 'lwc';

export default class ListDemo extends LightningElement {
    fruits = [
        { id: 1, name: 'Apple',  color: 'Red' },
        { id: 2, name: 'Banana', color: 'Yellow' },
        { id: 3, name: 'Cherry', color: 'Dark Red' }
    ];

    products = [
        { id: 'p1', name: 'Laptop', price: '$999' },
        { id: 'p2', name: 'Phone',  price: '$699' },
        { id: 'p3', name: 'Watch',  price: '$299' }
    ];
}
```

> ⚠️ `key` is **REQUIRED** in `for:each` — must be unique and a primitive (string/number). Never use array index as key in production!

### iterator:it (Advanced Loop — First/Last Access)

```html
<template>
    <!-- Like HTML ordered list with first/last awareness -->
    <ol>
        <template iterator:it={steps}>
            <li key={it.value.id}>
                <template lwc:if={it.first}>
                    <strong>START: </strong>
                </template>

                {it.value.label}

                <template lwc:if={it.last}>
                    <span> ← FINAL STEP</span>
                </template>
            </li>
        </template>
    </ol>
</template>
```

```javascript
steps = [
    { id: 1, label: 'Initialize' },
    { id: 2, label: 'Process' },
    { id: 3, label: 'Complete' }
];
```

### lwc:ref — Direct DOM Reference (API 59+)

```html
<template>
    <!-- Like getElementById but scoped to this component -->
    <input lwc:ref="nameInput" type="text" placeholder="Enter name" />
    <textarea lwc:ref="descArea" placeholder="Description"></textarea>
    <button onclick={focusInput}>Focus Name Field</button>
    <button onclick={clearAll}>Clear All</button>
</template>
```

```javascript
focusInput() {
    this.refs.nameInput.focus();
}

clearAll() {
    this.refs.nameInput.value = '';
    this.refs.descArea.value = '';
}
```

### Nested Loops

```html
<template>
    <template for:each={categories} for:item="category">
        <div key={category.id}>
            <h2>{category.name}</h2>
            <ul>
                <template for:each={category.items} for:item="item">
                    <li key={item.id}>{item.name}</li>
                </template>
            </ul>
        </div>
    </template>
</template>
```

```javascript
categories = [
    {
        id: 'c1', name: 'Electronics',
        items: [{ id: 'i1', name: 'Laptop' }, { id: 'i2', name: 'Phone' }]
    },
    {
        id: 'c2', name: 'Clothing',
        items: [{ id: 'i3', name: 'Shirt' }, { id: 'i4', name: 'Pants' }]
    }
];
```

---

## 4. JavaScript — Component Class

### Properties, Getters, Setters

```javascript
import { LightningElement } from 'lwc';

export default class JsBasics extends LightningElement {

    // ── Reactive Public Properties (settable by parent via HTML) ──
    // Declared with @api (see Decorators section)

    // ── Reactive Private Properties ──
    count = 0;
    name = 'LWC';
    isVisible = true;
    items = [];

    // ── Non-Reactive (won't trigger re-render on change) ──
    _internalCounter = 0;
    _subscription = null;

    // ── Getter (computed, like a derived HTML text value) ──
    get countLabel() {
        return `Count: ${this.count}`;
    }

    get isListEmpty() {
        return this.items.length === 0;
    }

    get formattedCurrency() {
        return new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: 'USD'
        }).format(this.amount);
    }

    // ── Getter + Setter pair ──
    _title = '';
    get title() {
        return this._title;
    }
    set title(val) {
        this._title = val ? val.toUpperCase() : '';
    }

    // ── Methods ──
    increment() {
        this.count++;
    }

    addItem(label) {
        // Always reassign array/object for reactivity without @track
        this.items = [...this.items, { id: Date.now(), label }];
    }

    removeItem(id) {
        this.items = this.items.filter(item => item.id !== id);
    }

    // ── Async Method ──
    async fetchData() {
        try {
            const result = await someApexMethod();
            this.items = result;
        } catch (error) {
            console.error('Error fetching data:', error.body?.message);
        }
    }
}
```

### DOM Querying

```javascript
// Single element (like document.querySelector but scoped to shadow DOM)
const inputEl = this.template.querySelector('input');
const myDiv   = this.template.querySelector('.my-class');
const byData  = this.template.querySelector('[data-id="item1"]');
const byTag   = this.template.querySelector('c-child-component');

// Multiple elements (like querySelectorAll)
const allInputs = this.template.querySelectorAll('input');
const allCards  = this.template.querySelectorAll('.card');

allCards.forEach(card => {
    card.classList.add('highlight');
});
```

---

## 5. Decorators

### @api — Public Property / Method

```javascript
// childComponent.js
import { LightningElement, api } from 'lwc';

export default class ChildComponent extends LightningElement {

    // Public properties — parent sets via HTML attributes
    @api firstName;
    @api lastName;
    @api userAge = 18;       // Default value
    @api isActive = false;

    // ⚠️ NEVER mutate @api props directly inside child — read only!
    // ❌ this.firstName = 'Bob';  — throws error

    // Public getter
    @api get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }

    // Public method — parent can call this
    @api resetForm() {
        this.localValue = '';
        // reset internal state
    }

    @api focusFirstField() {
        const input = this.template.querySelector('input');
        if (input) input.focus();
    }
}
```

```html
<!-- parentComponent.html -->
<template>
    <!-- Static values -->
    <c-child-component
        first-name="Alice"
        last-name="Smith"
        user-age="25"
        is-active>
    </c-child-component>

    <!-- Dynamic bindings -->
    <c-child-component
        first-name={user.firstName}
        last-name={user.lastName}
        user-age={user.age}
        is-active={user.isActive}>
    </c-child-component>
</template>
```

> **camelCase → kebab-case rule**: `firstName` → `first-name`, `isActive` → `is-active`, `userId` → `user-id`

### @track — Deep Reactivity

```javascript
import { LightningElement, track } from 'lwc';

export default class TrackDemo extends LightningElement {

    // Without @track — full reassignment triggers re-render ✅
    address = { street: '123 Main St', city: 'SF', zip: '94101' };

    updateCityReactive() {
        // Reassign entire object — works without @track ✅
        this.address = { ...this.address, city: 'LA' };
    }

    // With @track — nested mutation triggers re-render ✅
    @track config = {
        theme: 'dark',
        layout: { columns: 2, gap: '16px' },
        filters: []
    };

    updateNestedWithTrack() {
        // Direct nested mutation works WITH @track
        this.config.layout.columns = 3;
        this.config.filters.push('active');  // Array push works too!
    }

    // @track on arrays — push/splice trigger re-render
    @track notifications = [];

    addNotification(msg) {
        this.notifications.push({ id: Date.now(), msg });  // Reactive!
    }
}
```

### @wire — Reactive Data Binding

```javascript
import { LightningElement, api, wire } from 'lwc';
import { getRecord, getFieldValue } from 'lightning/uiRecordApi';
import CONTACT_NAME  from '@salesforce/schema/Contact.Name';
import CONTACT_EMAIL from '@salesforce/schema/Contact.Email';
import CONTACT_PHONE from '@salesforce/schema/Contact.Phone';

export default class WireDecorator extends LightningElement {

    @api recordId;

    // Wire as property (auto-populates .data / .error)
    @wire(getRecord, {
        recordId: '$recordId',   // $ = reactive, re-runs when recordId changes
        fields: [CONTACT_NAME, CONTACT_EMAIL, CONTACT_PHONE]
    })
    contact;

    // Wire as function (more control)
    @wire(getRecord, { recordId: '$recordId', fields: [CONTACT_NAME] })
    wiredContact({ data, error }) {
        if (data) {
            this.contactName = getFieldValue(data, CONTACT_NAME);
            this.isLoading = false;
        } else if (error) {
            console.error('Wire error:', error);
            this.isLoading = false;
        }
    }

    get name()  { return getFieldValue(this.contact.data, CONTACT_NAME);  }
    get email() { return getFieldValue(this.contact.data, CONTACT_EMAIL); }
    get phone() { return getFieldValue(this.contact.data, CONTACT_PHONE); }
    get hasError()   { return !!this.contact.error; }
    get isLoading()  { return !this.contact.data && !this.contact.error; }
}
```

---

## 6. Lifecycle Hooks

```javascript
import { LightningElement, api } from 'lwc';

export default class LifecycleDemo extends LightningElement {

    @api componentTitle;
    _rendered = false;
    _timerInterval = null;

    // 1️⃣ CONSTRUCTOR — Very first hook
    constructor() {
        super();  // Always call super() first!
        // ✅ Good for: initializing private variables
        // ❌ Don't: access child components, DOM, @api props, or this.template
        console.log('1. constructor');
        this._startTime = Date.now();
    }

    // 2️⃣ CONNECTED CALLBACK — Component inserted into DOM
    connectedCallback() {
        // ✅ Good for: subscribing to events, starting timers, fetching data
        // ✅ @api properties are now available
        // ✅ Can access this.template (but children may not be rendered yet)
        console.log('2. connectedCallback — title:', this.componentTitle);

        this._timerInterval = setInterval(() => {
            this.tick();
        }, 1000);

        // Subscribe to platform events, LMS, etc.
        this.subscribeToMessages();
    }

    // 3️⃣ RENDERED CALLBACK — After every render (initial + updates)
    renderedCallback() {
        // ✅ Good for: working with DOM, third-party library initialization
        // ⚠️ Runs after EVERY re-render — always use a guard for one-time logic!

        console.log('3. renderedCallback');

        if (!this._rendered) {
            this._rendered = true;
            // One-time DOM manipulation (e.g., init a chart library)
            this.initializeChart();
        }
    }

    // 4️⃣ DISCONNECTED CALLBACK — Component removed from DOM
    disconnectedCallback() {
        // ✅ Good for: cleanup — unsubscribe, clear timers, release resources
        console.log('4. disconnectedCallback — cleaning up');

        if (this._timerInterval) {
            clearInterval(this._timerInterval);
        }
        this.unsubscribeFromMessages();
    }

    // 5️⃣ ERROR CALLBACK — Child component throws an error
    errorCallback(error, stack) {
        // ✅ Acts as an error boundary for child errors
        console.error('5. errorCallback:', error.message);
        console.error('Stack:', stack);
        this.childError = error.message;
    }

    tick()                  { this.elapsedSeconds++; }
    initializeChart()       { /* init chart library */ }
    subscribeToMessages()   { /* LMS subscribe */ }
    unsubscribeFromMessages() { /* LMS unsubscribe */ }
}
```

### Lifecycle Order Diagram

```
Parent constructor
Parent connectedCallback
  ↓ Child constructor
  ↓ Child connectedCallback
  ↓ Child renderedCallback  ← Child renders first
Parent renderedCallback     ← Then parent
  ── (state change) ──
  ↓ Child renderedCallback  ← Child re-renders
Parent renderedCallback
  ── (parent removed) ──
  ↓ Child disconnectedCallback
Parent disconnectedCallback
```

---

## 7. Event Handling

### Native DOM Events

```html
<!-- eventDemo.html -->
<template>
    <!-- Mouse events -->
    <button onclick={handleClick} ondblclick={handleDoubleClick}>Click Me</button>
    <div onmouseenter={handleMouseEnter} onmouseleave={handleMouseLeave}>Hover Zone</div>

    <!-- Keyboard events -->
    <input
        type="text"
        onkeyup={handleKeyUp}
        onkeydown={handleKeyDown}
        onchange={handleChange}
        oninput={handleInput}
        placeholder="Type something..."
    />

    <!-- Form events -->
    <form onsubmit={handleSubmit}>
        <input type="text" name="username" />
        <button type="submit">Submit</button>
    </form>

    <!-- Focus events -->
    <input onfocus={handleFocus} onblur={handleBlur} />

    <!-- data-* attributes for event context -->
    <template for:each={items} for:item="item">
        <button
            key={item.id}
            data-id={item.id}
            data-name={item.name}
            onclick={handleItemClick}>
            {item.name}
        </button>
    </template>
</template>
```

```javascript
// eventDemo.js
import { LightningElement } from 'lwc';

export default class EventDemo extends LightningElement {

    items = [
        { id: '1', name: 'Alpha' },
        { id: '2', name: 'Beta' },
        { id: '3', name: 'Gamma' }
    ];

    // event.target = element that fired event
    // event.currentTarget = element with the listener
    handleClick(event) {
        console.log('Clicked:', event.target.tagName);
        event.preventDefault();  // Prevent default browser behavior
        event.stopPropagation(); // Stop event bubbling
    }

    handleDoubleClick() {
        console.log('Double clicked!');
    }

    handleKeyUp(event) {
        if (event.key === 'Enter') {
            this.submitValue(event.target.value);
        }
        if (event.key === 'Escape') {
            event.target.value = '';
        }
    }

    handleKeyDown(event) {
        // Block non-numeric input
        if (!/[0-9]/.test(event.key) && event.key !== 'Backspace') {
            event.preventDefault();
        }
    }

    handleChange(event) {
        this.inputValue = event.target.value;
    }

    handleInput(event) {
        // Fires on every keystroke (vs onChange which fires on blur)
        this.liveValue = event.target.value;
    }

    handleSubmit(event) {
        event.preventDefault();
        const form = event.target;
        const username = form.querySelector('[name="username"]').value;
        console.log('Submitted username:', username);
    }

    handleMouseEnter() { this.isHovered = true; }
    handleMouseLeave() { this.isHovered = false; }
    handleFocus()      { this.isFocused = true; }
    handleBlur()       { this.isFocused = false; }

    // Reading data-* attributes from event target
    handleItemClick(event) {
        const itemId   = event.currentTarget.dataset.id;
        const itemName = event.currentTarget.dataset.name;
        console.log(`Clicked item: ${itemId} — ${itemName}`);
    }
}
```

### Lightning Component Events

```html
<template>
    <!-- lightning-input fires 'change' event with event.detail.value -->
    <lightning-input
        label="Full Name"
        value={name}
        onchange={handleNameChange}>
    </lightning-input>

    <!-- lightning-combobox -->
    <lightning-combobox
        label="Status"
        value={selectedStatus}
        options={statusOptions}
        onchange={handleStatusChange}>
    </lightning-combobox>

    <!-- lightning-checkbox-group -->
    <lightning-checkbox-group
        label="Permissions"
        options={permissionOptions}
        value={selectedPermissions}
        onchange={handlePermissionsChange}>
    </lightning-checkbox-group>
</template>
```

```javascript
handleNameChange(event) {
    this.name = event.detail.value;  // lightning components use event.detail.value
}

handleStatusChange(event) {
    this.selectedStatus = event.detail.value;
}

handlePermissionsChange(event) {
    this.selectedPermissions = event.detail.value;  // Array of selected values
}

statusOptions = [
    { label: 'Active',   value: 'active' },
    { label: 'Inactive', value: 'inactive' },
    { label: 'Pending',  value: 'pending' }
];
```

---

## 8. Data Communication Patterns

> **The 6 patterns every LWC developer must master**

---

### Pattern 1: Parent → Child via @api

**Use when**: Parent needs to pass data **down** to a child component.

```
Parent ──(data)──→ Child
```

```javascript
// childCard.js
import { LightningElement, api } from 'lwc';

export default class ChildCard extends LightningElement {
    @api productName;
    @api productPrice;
    @api isAvailable = true;
    @api tags = [];  // Array prop

    get priceFormatted() {
        return `$${Number(this.productPrice).toFixed(2)}`;
    }

    get availabilityClass() {
        return this.isAvailable ? 'badge-green' : 'badge-red';
    }

    get availabilityLabel() {
        return this.isAvailable ? 'In Stock' : 'Out of Stock';
    }
}
```

```html
<!-- childCard.html -->
<template>
    <div class="product-card">
        <h2>{productName}</h2>
        <p class="price">{priceFormatted}</p>
        <span class={availabilityClass}>{availabilityLabel}</span>

        <div class="tags">
            <template for:each={tags} for:item="tag">
                <span key={tag} class="tag">{tag}</span>
            </template>
        </div>
    </div>
</template>
```

```html
<!-- parentPage.html -->
<template>
    <!-- Static values -->
    <c-child-card
        product-name="MacBook Pro"
        product-price="1999.99"
        is-available>
    </c-child-card>

    <!-- Dynamic binding from JS -->
    <c-child-card
        product-name={selectedProduct.name}
        product-price={selectedProduct.price}
        is-available={selectedProduct.inStock}
        tags={selectedProduct.tags}>
    </c-child-card>

    <!-- Loop of children -->
    <template for:each={productList} for:item="product">
        <c-child-card
            key={product.id}
            product-name={product.name}
            product-price={product.price}
            is-available={product.inStock}>
        </c-child-card>
    </template>
</template>
```

```javascript
// parentPage.js
import { LightningElement } from 'lwc';

export default class ParentPage extends LightningElement {
    selectedProduct = {
        name: 'iPhone 15',
        price: 999,
        inStock: true,
        tags: ['Apple', 'Mobile', '5G']
    };

    productList = [
        { id: '1', name: 'Laptop', price: 1299, inStock: true  },
        { id: '2', name: 'Tablet', price: 599,  inStock: false },
        { id: '3', name: 'Watch',  price: 399,  inStock: true  }
    ];
}
```

---

### Pattern 2: Child → Parent via Custom Events

**Use when**: Child needs to **notify parent** about something (click, selection, form submit).

```
Child ──(event)──→ Parent
```

```javascript
// productList.js (child)
import { LightningElement, api } from 'lwc';

export default class ProductList extends LightningElement {
    @api products = [];

    handleSelect(event) {
        const productId = event.currentTarget.dataset.id;
        const product   = this.products.find(p => p.id === productId);

        // 🔥 Fire a custom event with data in detail
        this.dispatchEvent(new CustomEvent('productselected', {
            detail: {
                productId:   product.id,
                productName: product.name,
                price:       product.price
            }
        }));
    }

    handleDelete(event) {
        const productId = event.currentTarget.dataset.id;
        this.dispatchEvent(new CustomEvent('productdeleted', {
            detail: { productId }
        }));
    }

    handleQuantityChange(event) {
        const productId = event.currentTarget.dataset.id;
        const quantity  = parseInt(event.target.value, 10);
        this.dispatchEvent(new CustomEvent('quantitychanged', {
            detail: { productId, quantity }
        }));
    }
}
```

```html
<!-- productList.html (child) -->
<template>
    <ul>
        <template for:each={products} for:item="product">
            <li key={product.id}>
                <span>{product.name}</span>
                <input
                    type="number"
                    min="1"
                    value="1"
                    data-id={product.id}
                    onchange={handleQuantityChange}
                />
                <button data-id={product.id} onclick={handleSelect}>Select</button>
                <button data-id={product.id} onclick={handleDelete}>Remove</button>
            </li>
        </template>
    </ul>
</template>
```

```html
<!-- shoppingCart.html (parent) -->
<template>
    <h1>Shopping Cart</h1>

    <template lwc:if={selectedProduct}>
        <p>Selected: {selectedProduct.productName} — {selectedProduct.price}</p>
    </template>

    <c-product-list
        products={allProducts}
        onproductselected={handleProductSelected}
        onproductdeleted={handleProductDeleted}
        onquantitychanged={handleQuantityChanged}>
    </c-product-list>

    <p>Total Items: {allProducts.length}</p>
</template>
```

```javascript
// shoppingCart.js (parent)
import { LightningElement } from 'lwc';

export default class ShoppingCart extends LightningElement {
    selectedProduct = null;

    allProducts = [
        { id: '1', name: 'Phone',  price: '$699' },
        { id: '2', name: 'Tablet', price: '$499' },
        { id: '3', name: 'Watch',  price: '$299' }
    ];

    // event.detail contains what child put in new CustomEvent('...', { detail: ... })
    handleProductSelected(event) {
        this.selectedProduct = event.detail;
        console.log('Selected:', event.detail.productName);
    }

    handleProductDeleted(event) {
        const { productId } = event.detail;
        this.allProducts = this.allProducts.filter(p => p.id !== productId);
    }

    handleQuantityChanged(event) {
        const { productId, quantity } = event.detail;
        this.allProducts = this.allProducts.map(p =>
            p.id === productId ? { ...p, quantity } : p
        );
    }
}
```

> **Naming rule**: Event name in `new CustomEvent('productselected')` → listener in HTML is `onproductselected`. **Lowercase only**, no hyphens.

---

### Pattern 3: Sibling Communication via Lightning Message Service (LMS)

**Use when**: Components are **not parent-child** — they're siblings, or in different regions of a Salesforce page (flexipage), or unrelated.

```
ComponentA ──(publish)──→ MessageChannel ──(subscribe)──→ ComponentB
```

**Step 1: Create Message Channel**

```xml
<!-- force-app/main/default/messageChannels/CartUpdated__c.messageChannel-meta.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<LightningMessageChannel xmlns="http://soap.sforce.com/2006/04/metadata">
    <masterLabel>CartUpdated</masterLabel>
    <isExposed>true</isExposed>
    <description>Notifies components when cart changes</description>
    <lightningMessageFields>
        <fieldName>cartItems</fieldName>
        <description>Array of cart items</description>
    </lightningMessageFields>
    <lightningMessageFields>
        <fieldName>totalPrice</fieldName>
        <description>Total cart price</description>
    </lightningMessageFields>
    <lightningMessageFields>
        <fieldName>action</fieldName>
        <description>Action type: add, remove, clear</description>
    </lightningMessageFields>
</LightningMessageChannel>
```

**Step 2: Publisher Component**

```javascript
// cartPublisher.js
import { LightningElement, wire } from 'lwc';
import { MessageContext, publish } from 'lightning/messageService';
import CART_UPDATED_CHANNEL from '@salesforce/messageChannel/CartUpdated__c';

export default class CartPublisher extends LightningElement {

    @wire(MessageContext)
    messageContext;

    cartItems = [];

    addToCart(product) {
        this.cartItems = [...this.cartItems, product];

        // 📢 Publish message — all subscribers on this channel receive it
        publish(this.messageContext, CART_UPDATED_CHANNEL, {
            cartItems:  this.cartItems,
            totalPrice: this.calculateTotal(),
            action:     'add'
        });
    }

    removeFromCart(productId) {
        this.cartItems = this.cartItems.filter(p => p.id !== productId);
        publish(this.messageContext, CART_UPDATED_CHANNEL, {
            cartItems:  this.cartItems,
            totalPrice: this.calculateTotal(),
            action:     'remove'
        });
    }

    clearCart() {
        this.cartItems = [];
        publish(this.messageContext, CART_UPDATED_CHANNEL, {
            cartItems:  [],
            totalPrice: 0,
            action:     'clear'
        });
    }

    calculateTotal() {
        return this.cartItems.reduce((sum, item) => sum + item.price, 0);
    }
}
```

**Step 3: Subscriber Component**

```javascript
// cartSummary.js (subscriber — can be ANYWHERE on the page)
import { LightningElement, wire } from 'lwc';
import {
    MessageContext,
    subscribe,
    unsubscribe,
    APPLICATION_SCOPE
} from 'lightning/messageService';
import CART_UPDATED_CHANNEL from '@salesforce/messageChannel/CartUpdated__c';

export default class CartSummary extends LightningElement {

    @wire(MessageContext)
    messageContext;

    cartItems  = [];
    totalPrice = 0;
    lastAction = '';
    subscription = null;

    connectedCallback() {
        this.subscribeToChannel();
    }

    subscribeToChannel() {
        // APPLICATION_SCOPE = receive messages from anywhere on the page
        // Default scope = only from same part of the app
        this.subscription = subscribe(
            this.messageContext,
            CART_UPDATED_CHANNEL,
            (message) => this.handleCartUpdate(message),
            { scope: APPLICATION_SCOPE }
        );
    }

    handleCartUpdate(message) {
        this.cartItems  = message.cartItems;
        this.totalPrice = message.totalPrice;
        this.lastAction = message.action;
    }

    disconnectedCallback() {
        // Always unsubscribe to prevent memory leaks!
        unsubscribe(this.subscription);
        this.subscription = null;
    }

    get itemCount() {
        return this.cartItems.length;
    }

    get formattedTotal() {
        return `$${this.totalPrice.toFixed(2)}`;
    }
}
```

```html
<!-- cartSummary.html -->
<template>
    <div class="cart-summary">
        <h3>Cart Summary</h3>
        <p>Items: {itemCount}</p>
        <p>Total: {formattedTotal}</p>
        <p>Last Action: {lastAction}</p>

        <ul>
            <template for:each={cartItems} for:item="item">
                <li key={item.id}>{item.name} — ${item.price}</li>
            </template>
        </ul>
    </div>
</template>
```

---

### Pattern 4: Parent Calling Child Methods

**Use when**: Parent needs to **trigger an action** in the child (reset, refresh, focus).

```
Parent ──(querySelector + method call)──→ Child @api method
```

```javascript
// formChild.js
import { LightningElement, api } from 'lwc';

export default class FormChild extends LightningElement {
    formData = { name: '', email: '', phone: '' };
    errors = {};

    // Public method — parent can call directly
    @api reset() {
        this.formData = { name: '', email: '', phone: '' };
        this.errors = {};
    }

    @api validate() {
        const isValid = Object.values(this.formData).every(v => v.trim() !== '');
        return isValid;
    }

    @api getData() {
        return { ...this.formData };
    }

    @api focusFirstField() {
        const firstInput = this.template.querySelector('input');
        if (firstInput) firstInput.focus();
    }

    // Public getter
    @api get isDirty() {
        return Object.values(this.formData).some(v => v !== '');
    }

    handleInput(event) {
        const field = event.target.dataset.field;
        this.formData = { ...this.formData, [field]: event.target.value };
    }
}
```

```html
<!-- formChild.html -->
<template>
    <input data-field="name"  oninput={handleInput} placeholder="Name"  />
    <input data-field="email" oninput={handleInput} placeholder="Email" />
    <input data-field="phone" oninput={handleInput} placeholder="Phone" />
</template>
```

```javascript
// formParent.js
import { LightningElement } from 'lwc';

export default class FormParent extends LightningElement {

    getChildRef() {
        return this.template.querySelector('c-form-child');
    }

    handleResetClick() {
        this.getChildRef().reset();
    }

    handleSubmitClick() {
        const child = this.getChildRef();

        if (!child.validate()) {
            alert('Please fill all fields!');
            child.focusFirstField();
            return;
        }

        const data = child.getData();
        console.log('Submitting:', data);
        child.reset();  // Clear after submit
    }

    handleCheckDirty() {
        const isDirty = this.getChildRef().isDirty;
        console.log('Form has unsaved changes:', isDirty);
    }
}
```

```html
<!-- formParent.html -->
<template>
    <c-form-child></c-form-child>

    <div class="actions">
        <button onclick={handleResetClick}>Reset Form</button>
        <button onclick={handleSubmitClick}>Submit</button>
        <button onclick={handleCheckDirty}>Check Changes</button>
    </div>
</template>
```

---

### Pattern 5: Bubbling Events

**Use when**: Event needs to travel up **multiple levels** of component hierarchy.

```
DeepChild ──(bubbles+composed)──→ MiddleParent ──→ GrandParent ──→ Root
```

```javascript
// deepChild.js
import { LightningElement } from 'lwc';

export default class DeepChild extends LightningElement {
    handleAction() {
        this.dispatchEvent(new CustomEvent('deepaction', {
            detail:   { message: 'From deep child', timestamp: Date.now() },
            bubbles:  true,   // Bubbles up through DOM
            composed: true    // Crosses shadow DOM boundaries (required in LWC!)
        }));
    }
}
```

```html
<!-- middleParent.html — can intercept or let it bubble -->
<template>
    <!-- Option 1: Let it bubble up (don't add listener) -->
    <c-deep-child></c-deep-child>

    <!-- Option 2: Intercept and re-fire (transform the event) -->
    <c-deep-child ondeepaction={handleAndRetransmit}></c-deep-child>
</template>
```

```javascript
// grandParent.js — catches the bubbled event
import { LightningElement } from 'lwc';

export default class GrandParent extends LightningElement {
    handleDeepAction(event) {
        // This fires even though the event came from a grandchild
        console.log('Caught bubbled event:', event.detail.message);
    }
}
```

```html
<!-- grandParent.html -->
<template>
    <!-- Listener here catches bubbled events from ANY depth below -->
    <div ondeepaction={handleDeepAction}>
        <c-middle-parent></c-middle-parent>
    </div>
</template>
```

> **Key rules**:
> - `bubbles: true` — event travels up the DOM
> - `composed: true` — event crosses shadow DOM boundaries (needed in LWC since each component has its own shadow)
> - Without `composed: true`, events stop at the shadow root

---

### Pattern 6: Pub/Sub Utility Service

**Use when**: Lightweight cross-component communication **without** full LMS setup, or in LWR (Experience Cloud) sites.

```javascript
// pubSubService.js  (create inside lwc/pubSubService/)
const listeners = {};

const subscribe = (eventName, callback) => {
    if (!listeners[eventName]) {
        listeners[eventName] = [];
    }
    if (!listeners[eventName].includes(callback)) {
        listeners[eventName].push(callback);
    }
};

const unsubscribe = (eventName, callback) => {
    if (listeners[eventName]) {
        listeners[eventName] = listeners[eventName].filter(cb => cb !== callback);
    }
};

const publish = (eventName, payload) => {
    if (listeners[eventName]) {
        listeners[eventName].forEach(cb => cb(payload));
    }
};

const clearAll = () => {
    Object.keys(listeners).forEach(key => delete listeners[key]);
};

export { subscribe, unsubscribe, publish, clearAll };
```

```javascript
// publisherComponent.js
import { LightningElement } from 'lwc';
import { publish } from 'c/pubSubService';

export default class PublisherComponent extends LightningElement {
    handleThemeChange(event) {
        publish('themeChanged', { theme: event.detail.value });
    }

    handleUserLogin(userData) {
        publish('userLoggedIn', { user: userData, timestamp: Date.now() });
    }
}
```

```javascript
// subscriberComponent.js
import { LightningElement } from 'lwc';
import { subscribe, unsubscribe } from 'c/pubSubService';

export default class SubscriberComponent extends LightningElement {
    currentTheme = 'light';

    // Bind for correct unsubscribe reference
    _themeHandler  = this.handleThemeChange.bind(this);
    _loginHandler  = this.handleUserLogin.bind(this);

    connectedCallback() {
        subscribe('themeChanged', this._themeHandler);
        subscribe('userLoggedIn', this._loginHandler);
    }

    handleThemeChange(payload) {
        this.currentTheme = payload.theme;
    }

    handleUserLogin(payload) {
        console.log('User logged in:', payload.user.name);
    }

    disconnectedCallback() {
        unsubscribe('themeChanged', this._themeHandler);
        unsubscribe('userLoggedIn', this._loginHandler);
    }
}
```

---

### Communication Pattern Summary

| Pattern | Direction | Use Case | Key API |
|---------|-----------|----------|---------|
| @api property | Parent → Child | Pass data down | `@api propName` |
| Custom Event | Child → Parent | Notify parent of action | `new CustomEvent(...)` |
| LMS | Any → Any | Cross-page-region, siblings | `publish / subscribe` |
| @api method call | Parent → Child | Trigger action in child | `@api methodName()` |
| Bubbling Event | Child → Ancestor | Skip intermediate parents | `bubbles: true, composed: true` |
| Pub/Sub Service | Any → Any | Lightweight, no Salesforce context | Custom utility module |

---

## 9. Wire Service & Apex

### Wire with UI API (Built-in Adapters)

```javascript
import { LightningElement, api, wire } from 'lwc';
import {
    getRecord,
    getFieldValue,
    createRecord,
    updateRecord,
    deleteRecord
} from 'lightning/uiRecordApi';
import { getPicklistValues, getObjectInfo } from 'lightning/uiObjectInfoApi';

import ACCOUNT_OBJECT from '@salesforce/schema/Account';
import NAME_FIELD     from '@salesforce/schema/Account.Name';
import PHONE_FIELD    from '@salesforce/schema/Account.Phone';
import TYPE_FIELD     from '@salesforce/schema/Account.Type';
import STATUS_FIELD   from '@salesforce/schema/Account.Status__c';

export default class AccountManager extends LightningElement {
    @api recordId;

    // ── Get Record ──
    @wire(getRecord, {
        recordId: '$recordId',
        fields: [NAME_FIELD, PHONE_FIELD, TYPE_FIELD]
    })
    account;

    get name()  { return getFieldValue(this.account.data, NAME_FIELD);  }
    get phone() { return getFieldValue(this.account.data, PHONE_FIELD); }
    get type()  { return getFieldValue(this.account.data, TYPE_FIELD);  }

    // ── Get Object Info (metadata) ──
    @wire(getObjectInfo, { objectApiName: ACCOUNT_OBJECT })
    accountInfo;

    // ── Get Picklist Values ──
    @wire(getPicklistValues, {
        recordTypeId: '$accountInfo.data.defaultRecordTypeId',
        fieldApiName: STATUS_FIELD
    })
    statusPicklistValues;

    get statusOptions() {
        if (!this.statusPicklistValues.data) return [];
        return this.statusPicklistValues.data.values.map(v => ({
            label: v.label,
            value: v.value
        }));
    }

    // ── Create Record ──
    async handleCreate() {
        const fields = {};
        fields[NAME_FIELD.fieldApiName]  = 'New Account';
        fields[PHONE_FIELD.fieldApiName] = '555-1234';

        try {
            const result = await createRecord({ apiName: 'Account', fields });
            console.log('Created Account ID:', result.id);
        } catch (error) {
            console.error('Create error:', error.body.message);
        }
    }

    // ── Update Record ──
    async handleUpdate() {
        const fields = {};
        fields['Id'] = this.recordId;
        fields[NAME_FIELD.fieldApiName] = 'Updated Name';

        try {
            await updateRecord({ fields });
            console.log('Updated successfully');
        } catch (error) {
            console.error('Update error:', error.body.message);
        }
    }

    // ── Delete Record ──
    async handleDelete() {
        try {
            await deleteRecord(this.recordId);
            console.log('Deleted successfully');
        } catch (error) {
            console.error('Delete error:', error.body.message);
        }
    }
}
```

### Wire with Apex

```apex
// AccountController.cls
public with sharing class AccountController {

    @AuraEnabled(cacheable=true)
    public static List<Account> getAccountsByIndustry(String industry, Integer limitCount) {
        return [
            SELECT Id, Name, Industry, AnnualRevenue, Phone, Type
            FROM Account
            WHERE Industry = :industry
            ORDER BY Name ASC
            LIMIT :limitCount
        ];
    }

    @AuraEnabled(cacheable=true)
    public static Account getAccountById(Id accountId) {
        return [SELECT Id, Name, Phone, Industry FROM Account WHERE Id = :accountId LIMIT 1];
    }

    @AuraEnabled
    public static Account upsertAccount(Account acc) {
        upsert acc;
        return acc;
    }

    @AuraEnabled
    public static void deleteAccount(Id accountId) {
        delete new Account(Id = accountId);
    }
}
```

```javascript
// accountManager.js
import { LightningElement, wire, track } from 'lwc';
import { ShowToastEvent }   from 'lightning/platformShowToastEvent';
import { refreshApex }      from '@salesforce/apex';
import getAccountsByIndustry from '@salesforce/apex/AccountController.getAccountsByIndustry';
import upsertAccount         from '@salesforce/apex/AccountController.upsertAccount';
import deleteAccount         from '@salesforce/apex/AccountController.deleteAccount';

export default class AccountManager extends LightningElement {

    selectedIndustry = 'Technology';
    accountLimit = 10;

    @track accounts = [];
    wiredResult;  // Store for refreshApex

    isLoading = false;
    error = null;

    // ── Wire as function (full control over data/error) ──
    @wire(getAccountsByIndustry, {
        industry:   '$selectedIndustry',  // Reactive — reruns on change
        limitCount: '$accountLimit'
    })
    wiredAccounts(result) {
        this.wiredResult = result;  // Store reference for refreshApex
        if (result.data) {
            this.accounts = result.data;
            this.error = null;
        } else if (result.error) {
            this.error = result.error.body.message;
            this.accounts = [];
        }
    }

    // ── Imperative Apex (called manually, not auto-reactive) ──
    async handleSearchByName(event) {
        const searchTerm = event.target.value;
        if (!searchTerm) return;

        this.isLoading = true;
        try {
            this.accounts = await searchAccountsByName({ searchTerm });
        } catch (error) {
            this.showToast('Error', error.body.message, 'error');
        } finally {
            this.isLoading = false;
        }
    }

    // ── DML via Imperative Apex ──
    async handleSave(event) {
        const { accountData } = event.detail;
        this.isLoading = true;

        try {
            await upsertAccount({ acc: accountData });
            this.showToast('Success', 'Account saved!', 'success');
            // Refresh wired data to reflect changes
            await refreshApex(this.wiredResult);
        } catch (error) {
            this.showToast('Error', error.body.message, 'error');
        } finally {
            this.isLoading = false;
        }
    }

    async handleDelete(event) {
        const { accountId } = event.detail;
        try {
            await deleteAccount({ accountId });
            this.showToast('Deleted', 'Account removed.', 'success');
            await refreshApex(this.wiredResult);
        } catch (error) {
            this.showToast('Error', error.body.message, 'error');
        }
    }

    handleIndustryChange(event) {
        this.selectedIndustry = event.detail.value;
        // Wire automatically reruns because selectedIndustry is reactive
    }

    showToast(title, message, variant) {
        this.dispatchEvent(new ShowToastEvent({ title, message, variant }));
    }

    get hasAccounts()  { return this.accounts.length > 0; }
    get hasError()     { return !!this.error; }
}
```

---

## 10. Forms & Validation

### Native HTML Form Equivalent in LWC

```html
<!-- contactForm.html -->
<template>
    <!-- fieldset + legend equivalent -->
    <fieldset>
        <legend>Contact Information</legend>

        <!-- text input -->
        <lightning-input
            label="First Name"
            type="text"
            value={formData.firstName}
            required
            onchange={handleFieldChange}
            data-field="firstName">
        </lightning-input>

        <!-- email input -->
        <lightning-input
            label="Email"
            type="email"
            value={formData.email}
            required
            onchange={handleFieldChange}
            data-field="email">
        </lightning-input>

        <!-- number input with min/max -->
        <lightning-input
            label="Age"
            type="number"
            min="18"
            max="100"
            value={formData.age}
            onchange={handleFieldChange}
            data-field="age">
        </lightning-input>

        <!-- date input -->
        <lightning-input
            label="Birth Date"
            type="date"
            value={formData.birthDate}
            onchange={handleFieldChange}
            data-field="birthDate">
        </lightning-input>

        <!-- password input -->
        <lightning-input
            label="Password"
            type="password"
            value={formData.password}
            onchange={handleFieldChange}
            data-field="password">
        </lightning-input>

        <!-- checkbox -->
        <lightning-input
            label="I agree to terms"
            type="checkbox"
            checked={formData.agreed}
            onchange={handleCheckboxChange}
            data-field="agreed">
        </lightning-input>

        <!-- textarea equivalent -->
        <lightning-textarea
            label="Notes"
            value={formData.notes}
            max-length="500"
            onchange={handleFieldChange}
            data-field="notes">
        </lightning-textarea>

        <!-- select / combobox -->
        <lightning-combobox
            label="Role"
            value={formData.role}
            options={roleOptions}
            required
            onchange={handleFieldChange}
            data-field="role">
        </lightning-combobox>

        <!-- radio group -->
        <lightning-radio-group
            label="Gender"
            options={genderOptions}
            value={formData.gender}
            type="radio"
            onchange={handleFieldChange}
            data-field="gender">
        </lightning-radio-group>

        <!-- file upload -->
        <lightning-file-upload
            label="Attachment"
            name="fileUploader"
            accept={acceptedFormats}
            record-id={recordId}
            onuploadfinished={handleUploadFinished}>
        </lightning-file-upload>
    </fieldset>

    <!-- Buttons -->
    <div class="form-buttons">
        <lightning-button label="Submit"  variant="brand"   onclick={handleSubmit}></lightning-button>
        <lightning-button label="Reset"   variant="neutral" onclick={handleReset}></lightning-button>
        <lightning-button label="Cancel"  variant="neutral" onclick={handleCancel}></lightning-button>
    </div>

    <!-- Error messages -->
    <template lwc:if={formErrors}>
        <div class="error-panel">
            <template for:each={formErrors} for:item="error">
                <p key={error}>{error}</p>
            </template>
        </div>
    </template>
</template>
```

```javascript
// contactForm.js
import { LightningElement, api } from 'lwc';

export default class ContactForm extends LightningElement {
    @api recordId;

    formData = {
        firstName: '',
        email:     '',
        age:       '',
        birthDate: '',
        password:  '',
        agreed:    false,
        notes:     '',
        role:      '',
        gender:    ''
    };

    formErrors = [];

    roleOptions = [
        { label: 'Developer', value: 'developer' },
        { label: 'Admin',     value: 'admin'     },
        { label: 'Manager',   value: 'manager'   }
    ];

    genderOptions = [
        { label: 'Male',   value: 'male'   },
        { label: 'Female', value: 'female' },
        { label: 'Other',  value: 'other'  }
    ];

    acceptedFormats = ['.pdf', '.png', '.jpg'];

    // Generic field handler using data-field attribute
    handleFieldChange(event) {
        const field = event.target.dataset.field;
        this.formData = { ...this.formData, [field]: event.detail.value };
    }

    handleCheckboxChange(event) {
        const field = event.target.dataset.field;
        this.formData = { ...this.formData, [field]: event.detail.checked };
    }

    // Validate using lightning-input's built-in validity
    validateForm() {
        const allInputs = [
            ...this.template.querySelectorAll('lightning-input'),
            ...this.template.querySelectorAll('lightning-combobox'),
            ...this.template.querySelectorAll('lightning-textarea')
        ];

        // Check built-in HTML5 validity on all inputs
        const allValid = allInputs.reduce((acc, input) => {
            input.reportValidity();  // Shows error message on invalid fields
            return acc && input.checkValidity();
        }, true);

        return allValid;
    }

    // Custom validation logic
    customValidate() {
        this.formErrors = [];

        if (this.formData.firstName.length < 2) {
            this.formErrors.push('First name must be at least 2 characters');
        }

        if (!this.formData.email.includes('@')) {
            this.formErrors.push('Please enter a valid email address');
        }

        if (parseInt(this.formData.age) < 18) {
            this.formErrors.push('You must be 18 or older');
        }

        if (!this.formData.agreed) {
            this.formErrors.push('You must agree to the terms');
        }

        return this.formErrors.length === 0;
    }

    handleSubmit() {
        const isLightningValid = this.validateForm();
        const isCustomValid    = this.customValidate();

        if (isLightningValid && isCustomValid) {
            console.log('Form data:', this.formData);
            this.dispatchEvent(new CustomEvent('formsubmit', {
                detail: { ...this.formData }
            }));
        }
    }

    handleReset() {
        this.formData  = { firstName: '', email: '', age: '', agreed: false, notes: '', role: '', gender: '' };
        this.formErrors = [];
        // Clear lightning-input errors
        this.template.querySelectorAll('lightning-input').forEach(input => {
            input.setCustomValidity('');
            input.reportValidity();
        });
    }

    handleCancel() {
        this.dispatchEvent(new CustomEvent('formcancel'));
    }

    handleUploadFinished(event) {
        const uploadedFiles = event.detail.files;
        console.log('Files uploaded:', uploadedFiles.length);
    }
}
```

### Record Edit Form (Salesforce Data Form)

```html
<template>
    <!-- lightning-record-edit-form = HTML form connected to Salesforce object -->
    <lightning-record-edit-form
        record-id={recordId}
        object-api-name="Contact"
        onsuccess={handleSuccess}
        onerror={handleError}
        onsubmit={handleSubmit}>

        <lightning-messages></lightning-messages>

        <lightning-input-field field-name="FirstName"></lightning-input-field>
        <lightning-input-field field-name="LastName"  required></lightning-input-field>
        <lightning-input-field field-name="Email"     required></lightning-input-field>
        <lightning-input-field field-name="Phone"></lightning-input-field>
        <lightning-input-field field-name="AccountId"></lightning-input-field>

        <lightning-button type="submit" label="Save" variant="brand"></lightning-button>
        <lightning-button type="reset" label="Reset" variant="neutral"></lightning-button>
    </lightning-record-edit-form>
</template>
```

```javascript
import { LightningElement, api } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

export default class ContactEditForm extends LightningElement {
    @api recordId;  // Pass null for create mode, actual ID for edit mode

    handleSuccess(event) {
        const record = event.detail;
        this.dispatchEvent(new ShowToastEvent({
            title:   'Success',
            message: `Contact ${record.fields.Name.value} saved!`,
            variant: 'success'
        }));
    }

    handleError(event) {
        console.error('Form error:', event.detail.detail);
    }

    handleSubmit(event) {
        event.preventDefault();  // Prevent default submit
        const fields = event.detail.fields;
        fields.LastName = fields.LastName + ' (Verified)';  // Modify before submit
        this.template.querySelector('lightning-record-edit-form').submit(fields);
    }
}
```

---

## 11. Tables in LWC

### HTML Table Equivalent

```html
<!-- basicTable.html -->
<template>
    <table>
        <!-- thead equivalent -->
        <thead>
            <tr>
                <th onclick={handleSort} data-field="name">
                    Name {getSortIndicator('name')}
                </th>
                <th onclick={handleSort} data-field="email">Email</th>
                <th onclick={handleSort} data-field="status">Status</th>
                <th>Actions</th>
            </tr>
        </thead>

        <!-- tbody equivalent -->
        <tbody>
            <template for:each={sortedUsers} for:item="user">
                <tr key={user.id} class={getRowClass(user)}>
                    <td>{user.name}</td>
                    <td>{user.email}</td>
                    <td>
                        <span class={user.statusClass}>{user.status}</span>
                    </td>
                    <td>
                        <button data-id={user.id} onclick={handleEdit}>Edit</button>
                        <button data-id={user.id} onclick={handleDelete}>Delete</button>
                    </td>
                </tr>
            </template>
        </tbody>

        <!-- tfoot equivalent -->
        <tfoot>
            <tr>
                <td colspan="4">Total: {userCount} users</td>
            </tr>
        </tfoot>
    </table>
</template>
```

### Lightning Datatable (Production Table Component)

```html
<!-- userTable.html -->
<template>
    <lightning-datatable
        key-field="id"
        data={users}
        columns={columns}
        sorted-by={sortedBy}
        sorted-direction={sortedDirection}
        onsort={handleSort}
        onrowaction={handleRowAction}
        selected-rows={selectedRowIds}
        onrowselection={handleRowSelection}
        max-row-selection="5"
        draft-values={draftValues}
        onsave={handleSave}
        hide-checkbox-column={hideCheckbox}
        is-loading={isLoading}>
    </lightning-datatable>

    <!-- Caption (like HTML caption element) -->
    <p>Showing {users.length} of {totalCount} records</p>
</template>
```

```javascript
// userTable.js
import { LightningElement, track } from 'lwc';

export default class UserTable extends LightningElement {

    @track users = [
        { id: '1', name: 'Alice Smith', email: 'alice@example.com', status: 'Active',   age: 30, salary: 75000 },
        { id: '2', name: 'Bob Jones',   email: 'bob@example.com',   status: 'Inactive', age: 25, salary: 55000 },
        { id: '3', name: 'Carol White', email: 'carol@example.com', status: 'Active',   age: 35, salary: 90000 }
    ];

    sortedBy = 'name';
    sortedDirection = 'asc';
    selectedRowIds = [];
    @track draftValues = [];
    isLoading = false;

    // Column definitions (like HTML <th> with config)
    columns = [
        {
            label: 'Name',
            fieldName: 'name',
            type: 'text',
            sortable: true,
            editable: true
        },
        {
            label: 'Email',
            fieldName: 'email',
            type: 'email',
            sortable: true
        },
        {
            label: 'Status',
            fieldName: 'status',
            type: 'text',
            cellAttributes: {
                class: { fieldName: 'statusClass' }  // Dynamic CSS class per cell
            }
        },
        {
            label: 'Age',
            fieldName: 'age',
            type: 'number',
            sortable: true
        },
        {
            label: 'Salary',
            fieldName: 'salary',
            type: 'currency',
            typeAttributes: { currencyCode: 'USD', minimumFractionDigits: 0 }
        },
        {
            label: 'Profile',
            fieldName: 'profileUrl',
            type: 'url',
            typeAttributes: { label: { fieldName: 'name' }, target: '_blank' }
        },
        {
            type: 'action',
            typeAttributes: {
                rowActions: [
                    { label: 'View',   name: 'view'   },
                    { label: 'Edit',   name: 'edit'   },
                    { label: 'Delete', name: 'delete' }
                ]
            }
        }
    ];

    handleSort(event) {
        const { fieldName, sortDirection } = event.detail;
        this.sortedBy        = fieldName;
        this.sortedDirection = sortDirection;

        this.users = [...this.users].sort((a, b) => {
            const aVal = a[fieldName];
            const bVal = b[fieldName];
            const direction = sortDirection === 'asc' ? 1 : -1;
            return aVal > bVal ? direction : aVal < bVal ? -direction : 0;
        });
    }

    handleRowAction(event) {
        const { name }   = event.detail.action;
        const { row }    = event.detail;

        switch (name) {
            case 'view':
                console.log('Viewing:', row.name);
                break;
            case 'edit':
                this.handleEditRow(row);
                break;
            case 'delete':
                this.users = this.users.filter(u => u.id !== row.id);
                break;
        }
    }

    handleRowSelection(event) {
        this.selectedRowIds = event.detail.selectedRows.map(row => row.id);
        console.log('Selected:', this.selectedRowIds);
    }

    // Inline editing save
    handleSave(event) {
        const updatedFields = event.detail.draftValues;

        this.users = this.users.map(user => {
            const draft = updatedFields.find(d => d.id === user.id);
            return draft ? { ...user, ...draft } : user;
        });

        this.draftValues = [];
    }

    get userCount() { return this.users.length; }
}
```

---

## 12. CSS & Styling

LWC uses **Shadow DOM** — styles are automatically scoped to the component.

```css
/* myComponent.css — scoped automatically */

/* Host element styling (like :root) */
:host {
    display: block;
    padding: 16px;
    font-family: 'Salesforce Sans', Arial, sans-serif;
}

/* Basic selectors work as in normal CSS */
.container {
    max-width: 800px;
    margin: 0 auto;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.title {
    font-size: 1.5rem;
    font-weight: bold;
    color: #032d60;
    margin-bottom: 8px;
}

/* Pseudo-classes */
.btn:hover    { background-color: #0070d2; }
.btn:active   { transform: scale(0.98); }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }

/* CSS Custom Properties (variables) */
:host {
    --primary-color: #0070d2;
    --text-color:    #3e3e3c;
    --spacing-sm:    8px;
    --spacing-md:    16px;
    --spacing-lg:    24px;
}

.badge {
    padding: var(--spacing-sm) var(--spacing-md);
    border-radius: 4px;
}

/* Status variants */
.badge-active   { background-color: #4bca81; color: white; }
.badge-inactive { background-color: #c23934; color: white; }
.badge-pending  { background-color: #ffb75d; color: #3e3e3c; }

/* Responsive */
@media (max-width: 640px) {
    .container { padding: 8px; }
    .title     { font-size: 1.2rem; }
}
```

### SLDS (Salesforce Lightning Design System) Classes

```html
<template>
    <!-- SLDS utility classes work directly in LWC templates -->
    <div class="slds-box slds-theme_default">
        <h2 class="slds-text-heading_medium">Title</h2>
        <p class="slds-text-body_regular slds-m-top_small">Content</p>
    </div>

    <div class="slds-grid slds-wrap">
        <div class="slds-col slds-size_1-of-2">Left</div>
        <div class="slds-col slds-size_1-of-2">Right</div>
    </div>

    <!-- Spacing utilities -->
    <div class="slds-m-around_medium">Margin all sides</div>
    <div class="slds-p-horizontal_large">Padding left/right</div>

    <!-- Dynamic class based on state -->
    <div class={cardClass}>
        Dynamic card
    </div>
</template>
```

```javascript
get cardClass() {
    return [
        'slds-box',
        this.isSelected ? 'slds-theme_shade' : 'slds-theme_default',
        this.isLarge    ? 'slds-p-around_large' : 'slds-p-around_small'
    ].join(' ');
}
```

---

## 13. Slots — Content Projection

Like HTML `<slot>` in Web Components — lets parent inject content into child.

### Default Slot

```html
<!-- modal.html (child with slot) -->
<template>
    <div class="modal">
        <header class="modal-header">
            <slot name="header">Default Header</slot>
        </header>

        <div class="modal-body">
            <!-- Default slot — unnamed receives unlabeled content -->
            <slot>No content provided</slot>
        </div>

        <footer class="modal-footer">
            <slot name="footer">
                <button onclick={handleClose}>Close</button>
            </slot>
        </footer>
    </div>
</template>
```

```html
<!-- parentPage.html — using the modal with slot content -->
<template>
    <c-modal>
        <!-- Goes into slot name="header" -->
        <span slot="header">Confirm Deletion</span>

        <!-- Goes into default (unnamed) slot -->
        <p>Are you sure you want to delete this record?</p>
        <p>This action cannot be undone.</p>

        <!-- Goes into slot name="footer" -->
        <span slot="footer">
            <button onclick={handleConfirm}>Confirm</button>
            <button onclick={handleCancel}>Cancel</button>
        </span>
    </c-modal>
</template>
```

---

## 14. Navigation Service

```javascript
import { LightningElement } from 'lwc';
import { NavigationMixin } from 'lightning/navigation';

// Extend NavigationMixin instead of LightningElement directly
export default class NavigationDemo extends NavigationMixin(LightningElement) {

    // Navigate to a record page
    navigateToRecord(recordId) {
        this[NavigationMixin.Navigate]({
            type: 'standard__recordPage',
            attributes: {
                recordId: recordId,
                actionName: 'view'   // 'view' | 'edit' | 'clone'
            }
        });
    }

    // Navigate to create new record
    navigateToNewRecord() {
        this[NavigationMixin.Navigate]({
            type: 'standard__objectPage',
            attributes: {
                objectApiName: 'Contact',
                actionName: 'new'
            }
        });
    }

    // Navigate to an LWC/Aura app page
    navigateToAppPage() {
        this[NavigationMixin.Navigate]({
            type: 'standard__navItemPage',
            attributes: { apiName: 'My_Custom_Tab' }
        });
    }

    // Navigate to a named page (Home, Chatter, etc.)
    navigateToHome() {
        this[NavigationMixin.Navigate]({
            type: 'standard__namedPage',
            attributes: { pageName: 'home' }
        });
    }

    // Navigate to external URL
    navigateToURL(url) {
        this[NavigationMixin.Navigate]({
            type: 'standard__webPage',
            attributes: { url }
        });
    }

    // Generate URL without navigating
    async generateRecordURL(recordId) {
        const url = await this[NavigationMixin.GenerateUrl]({
            type: 'standard__recordPage',
            attributes: { recordId, actionName: 'view' }
        });
        return url;
    }
}
```

---

## 15. Salesforce-Specific Features

### Custom Labels, User Info, Permissions

```javascript
import { LightningElement } from 'lwc';
import WELCOME_MESSAGE from '@salesforce/label/c.WelcomeMessage';
import ERROR_MESSAGE   from '@salesforce/label/c.ErrorMessage';
import userId          from '@salesforce/user/Id';
import userFullName    from '@salesforce/user/FullName';
import isGuest         from '@salesforce/user/isGuest';
import canEditAccounts from '@salesforce/customPermission/Edit_Accounts';
import COMPANY_LOGO    from '@salesforce/resourceUrl/CompanyLogo';

export default class SalesforceFeatures extends LightningElement {

    // Custom labels (i18n)
    welcomeMessage = WELCOME_MESSAGE;
    errorMessage   = ERROR_MESSAGE;

    // Current user info
    currentUserId   = userId;
    currentUserName = userFullName;
    isGuestUser     = isGuest;

    // Custom permission
    canEdit = canEditAccounts;

    // Static resource
    logoUrl = COMPANY_LOGO;

    get greeting() {
        return `${this.welcomeMessage}, ${this.currentUserName}!`;
    }
}
```

### Platform Events & Streaming

```javascript
import { LightningElement, wire } from 'lwc';
import { subscribe, unsubscribe, onError } from 'lightning/empApi';

export default class PlatformEventListener extends LightningElement {

    channelName = '/event/Order_Update__e';
    subscription = null;
    latestEvent = null;

    connectedCallback() {
        this.subscribeToEvent();
    }

    subscribeToEvent() {
        const messageCallback = (event) => {
            this.latestEvent = event.data.payload;
            console.log('Platform Event received:', this.latestEvent);
        };

        subscribe(this.channelName, -1, messageCallback)
            .then(sub => {
                this.subscription = sub;
                console.log('Subscribed to', this.channelName);
            });

        onError(error => {
            console.error('EMP API error:', error);
        });
    }

    disconnectedCallback() {
        if (this.subscription) {
            unsubscribe(this.subscription, () => {
                console.log('Unsubscribed from platform events');
            });
        }
    }
}
```

---

## 16. Best Practices

### ✅ DO

```javascript
// 1. Always reassign arrays/objects for reactivity
this.items = [...this.items, newItem];
this.user  = { ...this.user, name: 'New Name' };

// 2. Use getters for computed values
get displayName() { return `${this.firstName} ${this.lastName}`; }

// 3. Guard renderedCallback for one-time logic
renderedCallback() {
    if (!this._initialized) {
        this._initialized = true;
        this.setup();
    }
}

// 4. Unsubscribe / cleanup in disconnectedCallback
disconnectedCallback() {
    unsubscribe(this.subscription);
    clearInterval(this.timer);
}

// 5. Use optional chaining to avoid null errors
get accountName() { return this.account?.data?.fields?.Name?.value ?? ''; }

// 6. Use data-* attributes for event context in loops
// HTML: <button data-id={item.id} onclick={handleClick}>
handleClick(event) {
    const id = event.currentTarget.dataset.id;
}

// 7. Use @salesforce/schema for field/object API names (refactoring safety)
import NAME_FIELD from '@salesforce/schema/Account.Name';
```

### ❌ DON'T

```javascript
// 1. Don't mutate @api properties
@api userData;
handleClick() {
    this.userData.name = 'Bob'; // ❌ THROWS ERROR — @api is read-only in child
}

// 2. Don't query DOM in constructor or connectedCallback expecting rendered elements
constructor() {
    const el = this.template.querySelector('input'); // ❌ Not rendered yet
}

// 3. Don't set state in renderedCallback without a guard (infinite loop!)
renderedCallback() {
    this.count++; // ❌ Setting state causes re-render → renders again → infinite loop
}

// 4. Don't use document.querySelector (breaks shadow DOM)
const el = document.querySelector('.my-class'); // ❌ Use this.template.querySelector

// 5. Don't forget key in for:each
// <template for:each={items} for:item="item"> ← Missing key={item.id} ❌

// 6. Don't expose sensitive data in @api
@api password; // ❌ @api is public — anyone can read it
```

---

## 17. Practice Projects

Build these to master LWC from basics to advanced:

### Beginner
- [ ] **Hello World** — Component with `@api` title, rendered in App Builder
- [ ] **Counter** — Increment/decrement with min/max, reset button
- [ ] **To-Do List** — Add, delete, mark complete. Array reactivity practice
- [ ] **Color Picker** — Range sliders for RGB, live preview via getters

### Intermediate
- [ ] **Parent-Child Product Selector** — ProductList → Cart (custom events)
- [ ] **Contact Form** — All input types + validation + success toast
- [ ] **Sortable Datatable** — `lightning-datatable` with sort, search, pagination
- [ ] **Record Viewer** — `@wire(getRecord)` + display fields dynamically

### Advanced
- [ ] **Shopping Cart** (LMS) — ProductCatalog publishes, CartSummary subscribes
- [ ] **Dynamic Form Builder** — Render form fields from Apex metadata
- [ ] **Real-time Dashboard** — Platform Events + `empApi` streaming
- [ ] **Infinite Scroll List** — Load more on scroll with Apex cursor pagination

---

## Quick Reference Card

```
Data Flow:
  Parent → Child    :  @api property / attribute
  Child → Parent    :  new CustomEvent + dispatchEvent
  Any → Any         :  Lightning Message Service (LMS)
  Parent → Child    :  @api method call via querySelector
  Deep bubbling     :  CustomEvent with bubbles:true, composed:true

Directives:
  lwc:if / lwc:elseif / lwc:else   ← Conditionals (API 56+)
  for:each / for:item / key        ← Loops
  iterator:it                      ← Loop with first/last access
  lwc:ref                          ← DOM reference (API 59+)

Decorators:
  @api    ← Public property or method
  @track  ← Deep nested reactivity
  @wire   ← Reactive data from Apex or UI API

Lifecycle:
  constructor → connectedCallback → renderedCallback
  disconnectedCallback → errorCallback

Wire Syntax:
  @wire(apexMethod, { param: '$reactiveVar' })
  wiredResult({ data, error }) { ... }

Events:
  Native:    onclick, onchange, onkeyup, oninput, onfocus
  Custom:    new CustomEvent('myevent', { detail: {...} })
  Listening: onmyevent={handlerMethod} in parent HTML
```

---

*Made with ❤️ for LWC learners. Star ⭐ the repo if this helped you!*

*Resources: [LWC Dev Guide](https://developer.salesforce.com/docs/component-library) | [SLDS](https://www.lightningdesignsystem.com/) | [Trailhead](https://trailhead.salesforce.com)*
