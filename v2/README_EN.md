# Foxentry Javascript API v2

The latest version of the library, compatible with the new application [app.foxentry.com](https://app.foxentry.com/).

- [Back to the index](../README.md)
- [Go to v1 documentation](../v1/README.md)
- [Go to migration guide](#migration-from-v1-to-v2)

---

The Foxentry JavaScript API v2 allows you to control the validation of mapped inputs directly from JavaScript. The library is designed for the new Foxentry application and works with inputs that are configured and mapped within your project.

**To use the JavaScript API v2, you must have a Foxentry project created in the [app.foxentry.com](https://app.foxentry.com/) application and have the integration JavaScript library embedded on your website.**

The `onFoxentryProjectLoad` global function is also available in version v2. If you declare it in your code, the library will automatically call it after it loads.

#### Example of using `onFoxentryProjectLoad`
```javascript
function onFoxentryProjectLoad() {
  console.warn("Foxentry v2 is loaded");
}
```

## Checking if the library is ready

The `isReady()` method indicates whether the library is fully loaded and ready for use. We recommend using this method before running validation or reading validation statuses.

#### Calling the method
```javascript
var ready = Foxentry.isReady();
console.warn(ready);
```

#### Result of the call
```text
true or false
```

## Running validation on mapped inputs

The `validate()` method performs validation on all mapped inputs within the specified parent. If no parent is provided, validation is performed on all mapped inputs.

**Parameters**
- `parent` - optional parent element or selector of type `HtmlElementOrSelector`

#### Validating all mapped inputs
```javascript
await Foxentry.validate();
```

#### Validating inputs within a specific form
```javascript
var form = document.querySelector("#orderForm");

await Foxentry.validate(form);
```

#### Validation using a selector
```javascript
Foxentry.validate("#orderForm").then(function () {
  console.warn("Validation complete");
});
```

## Retrieving the Current Validation Status

The `getValidationStatus()` method returns the current validation status of the mapped inputs. If the `parent` parameter is provided, it returns only the status of the inputs within that parent. If the parameter is not provided, it returns the status of all mapped inputs.

**Parameters**
- `parent` - optional parent element or selector of type `HtmlElementOrSelector`

#### Getting the status of all inputs
```javascript
var status = Foxentry.getValidationStatus();
console.warn(status);
```

#### Getting the status of inputs inside a form
```javascript
var status = Foxentry.getValidationStatus("#orderForm");
console.warn(status.isValid);
console.warn(status.inputs);
```

#### Return type
```typescript
export interface InputValidationInfo {
  type: string;
  isValid: boolean;
  isLoading: boolean;
  mark: 'valid' | 'invalid' | 'alert' | null;
  message: string | null;
  ref: HTMLElement;
}

export interface InputsValidationStatus {
  inputs: InputValidationInfo[];
  isValid: boolean;
  isLoading: boolean;
}
```

#### Description of InputsValidationStatus Attributes
- `inputs` - a list of all mapped inputs within the given scope and their current validation statuses
- `isValid` - indicates whether all included inputs are currently valid
- `isLoading` - indicates whether validation is currently in progress for any of the included inputs

#### Description of InputValidationInfo attributes
- `type` - the mapped input type
- `isValid` - indicates whether a specific input is currently evaluated as valid
- `isLoading` - indicates whether validation is currently in progress for a specific input
- `mark` - the input's state; can take the values `valid`, `invalid`, `alert`, or `null` if no flag is set
- `message` - the validation message for the given input; returns `null` if not available
- `ref` - a reference to the specific DOM element of the input

#### Example of a return value
```json
{
  "inputs": [
    {
      "type": "street",
      "isValid": true,
      "isLoading": false,
      "mark": "valid",
      "message": null,
      "ref": "HTMLElement"
    },
    {
      "type": "streetWithNumber",
      "isValid": false,
      "isLoading": false,
      "mark": "invalid",
      "message": "The entered value is invalid",
      "ref": "HTMLElement"
    }
  ],
  "isValid": false,
  "isLoading": false
}
```

## `parent` type: HTMLElementOrSelector

The `validate()` and `getValidationStatus()` methods can accept an optional `parent` parameter, which limits processing to only the mapped inputs inside the specified parent.

```typescript
type HtmlElementOrSelector = HTMLElement | JQuery | string | null
```

You can pass:

- `HTMLElement` - a specific DOM element, such as a form or container
- `JQuery` - a jQuery object containing the parent element
- `string` - a CSS selector used to locate the parent element
- `null` - an empty value; effectively the same as not passing the parameter

If you do not pass the `parent` parameter, all mapped inputs on the page will be processed.

#### Examples of `parent` values
```javascript
var formElement = document.querySelector("#orderForm");
var jqueryForm = $("#orderForm");
var selector = "#orderForm";
var emptyParent = null;
```

## Setting callbacks for individual validators

The `setCallbacks()` method is used to set callback functions for individual validator types. The callback is invoked after the validation of the given validator has completed and allows you to further process the validation result in JavaScript.

You can set callbacks directly or within `onFoxentryProjectLoad()`, which the library automatically calls in v2 after loading.

#### Callback types
```typescript
export type FoxentryCallback = (input: HTMLElement, validation: ValidationCallbackValidationArgument) => unknown;

export type FoxentryCallbacksType = {
  [validatorType in ValidatorTypes]: FoxentryCallback;
};

export interface ValidationCallbackValidationArgument {
  group: GroupValidationInfo;
  response: ApiResponse;
}

export interface GroupValidationInfo {
  isValid: boolean;
  isLoading: boolean;
  validatorType: ValidatorTypes;
  inputs: Array<InputValidationInfo>;
}
```

The first argument of the `input` callback contains a reference to the input field that was validated. The second argument, `validation`, contains an object of type `ValidationCallbackValidationArgument`, where the immediate state of the form inputs is available in the `group` attribute and the API response is available in the `response` attribute.

The `ApiResponse` format is not described further in this documentation. You can find it on [foxentry.dev](https://foxentry.dev) in the API version 2.1 documentation.

#### Description of ValidationCallbackValidationArgument Attributes
- `group` - details of the validation group to which the currently validated input belongs; represents the immediate state of the form inputs in a processable structure
- `response` - the API response; its format is described on [foxentry.dev](https://foxentry.dev) in the API version 2.1 documentation

#### Description of GroupValidationInfo Attributes
- `isValid` - indicates whether the group of inputs is currently evaluated as valid
- `isLoading` - indicates whether validation is currently in progress for the group of inputs
- `validatorType` - the type of validator to which the callback belongs
- `inputs` - a list of inputs included in the given validation group, including their current status

#### Available validator types
The library supports the following validator types:

- `email`
- `phone`
- `company`
- `name`
- `location`

#### Configuring callbacks
```javascript
function onFoxentryProjectLoad() {
  function emailCallback(input, validation) {
    console.warn("Email input:", input);
    console.warn("Validation group result:", validation.group);
    console.warn("API response:", validation.response);
  }

  function phoneCallback(input, validation) {
    console.warn("Phone input:", input);
    console.warn("Validator type:", validation.group.validatorType);
    console.warn("Validated inputs:", validation.group.inputs);
    console.warn("API response:", validation.response);
  }

  Foxentry.setCallbacks({
    email: emailCallback,
    phone: phoneCallback,
    company: function (input, validation) {
      console.warn("Company:", input, validation);
    },
    name: function (input, validation) {
      console.warn("Name:", input, validation);
    },
    location: function (input, validation) {
      console.warn("Location:", input, validation);
    }
  });
}
```

## Migration from v1 to v2

This section describes the most important differences when migrating from v1 to v2. The biggest changes concern working with callbacks.

### 1. Change in callback registration

In v1, callbacks were typically set using `FoxentryBuilder.setCallbacks(...)` inside `onFoxentryProjectLoad()`.
In v2, `onFoxentryProjectLoad()` remains available, but callbacks must be registered using `Foxentry.setCallbacks(...)`.

#### v1
```javascript
function onFoxentryProjectLoad() {
  FoxentryBuilder.setCallbacks({
    address: function (data, validatorInfo) {
      console.warn(data);
      console.warn(validatorInfo);
    }
  });
}
```

#### v2
```javascript
function onFoxentryProjectLoad() {
  Foxentry.setCallbacks({
    location: function (input, validation) {
      console.warn(input);
      console.warn(validation.group);
    },
    company: function (input, validation) {},
    email: function (input, validation) {},
    name: function (input, validation) {},
    phone: function (input, validation) {}
  });
}
```

### 2. Change in both callback argument formats

In v1, the callback also accepted two arguments (`data`, `validatorInfo`).
In v2, the callback still uses two arguments, but both argument formats are new:

```typescript
(input: HTMLElement, validation: ValidationCallbackValidationArgument) => unknown
```

- `input` - the first argument is now the validated DOM element (`HTMLElement`) (in v1 the first argument was `data`)
- `validation.group` - the second argument now includes a `group` attribute that represents the immediate state of the form inputs in a processable structure
- `validation.response` - the second argument now also includes a `response` attribute that represents the API response; its format is described on [foxentry.dev](https://foxentry.dev) in the API version 2.1 documentation

### 3. Change in validator name: `address` -> `location`

If you use a callback for `address` in v1, you need to use the `location` key in v2.

#### v1
```javascript
FoxentryBuilder.setCallbacks({
  address: function (data, validatorInfo) {
    console.warn(data);
    console.warn(validatorInfo);
  }
});
```

#### v2
```javascript
Foxentry.setCallbacks({
  location: function (input, validation) {
    console.warn(validation.group.inputs);
  },
  company: function () {},
  email: function () {},
  name: function () {},
  phone: function () {}
});
```

### 4. Recommended Migration Procedure

1. You can keep the `onFoxentryProjectLoad()` function; the library will also call it in v2 after loading.
2. Inside `onFoxentryProjectLoad()`, replace `FoxentryBuilder.setCallbacks(...)` with `Foxentry.setCallbacks(...)`.
3. Update data handling in callback code: the first argument is now a DOM element and the second argument now exposes both `validation.group` and `validation.response`.
4. Rename the `address` callback key to `location`.
5. After making the changes, verify the behavior by calling `await Foxentry.validate(...)` and checking the result via `Foxentry.getValidationStatus(...)`.
