# Foxentry Javascript API v2

Aktuální verze knihovny, kompatibilní s novou aplikací [app.foxentry.com](https://app.foxentry.com/).

- [Zpět na rozcestník](../README.md)
- [Přejít na dokumentaci v1](../v1/README.md)
- [Přejít na návod k migraci](#migrace-z-v1-na-v2)

---

Foxentry Javascript API v2 umožňuje ovládat validaci namapovaných inputů přímo z Javascriptu. Knihovna je určená pro novou aplikaci Foxentry a pracuje nad inputy, které jsou nakonfigurované a namapované v rámci vašeho projektu.

**Pro použití Javascript API v2 je potřeba mít vytvořený Foxentry projekt v aplikaci [app.foxentry.com](https://app.foxentry.com/) a mít na webu vložený integrační javascript knihovny.**

Ve verzi v2 je také dostupná globální funkce `onFoxentryProjectLoad`. Pokud ji deklarujete ve svém kódu, knihovna ji po svém načtení automaticky zavolá.

#### Příklad použití `onFoxentryProjectLoad`
```javascript
function onFoxentryProjectLoad() {
  console.warn("Foxentry v2 je načtené");
}
```

## Zjištění, zda je knihovna připravená

Metoda `isReady()` indikuje, zda je knihovna plně načtená a připravená k použití. Tuto metodu doporučujeme použít před spuštěním validace nebo před čtením stavů validace.

#### Volání metody
```javascript
var ready = Foxentry.isReady();
console.warn(ready);
```

#### Výsledek volání
```text
true nebo false
```

## Spuštění validace namapovaných inputů

Metoda `validate()` provede validaci nad všemi namapovanými inputy uvnitř zadaného rodiče. Pokud rodiče nepředáte, validace proběhne nad všemi namapovanými inputy.

**Parametry**
- `parent` - nepovinný rodičovský element nebo selector typu `HtmlElementOrSelector`

#### Validace všech namapovaných inputů
```javascript
await Foxentry.validate();
```

#### Validace inputů uvnitř konkrétního formuláře
```javascript
var form = document.querySelector("#orderForm");

await Foxentry.validate(form);
```

#### Validace pomocí selectoru
```javascript
Foxentry.validate("#orderForm").then(function () {
  console.warn("Validace dokončena");
});
```

## Získání aktuálního stavu validace

Metoda `getValidationStatus()` vrací aktuální stav validací namapovaných inputů. Pokud je předán parametr `parent`, vrací pouze stav inputů uvnitř daného rodiče. Pokud parametr nepředáte, vrací stav všech namapovaných inputů.

**Parametry**
- `parent` - nepovinný rodičovský element nebo selector typu `HtmlElementOrSelector`

#### Získání stavu všech inputů
```javascript
var status = Foxentry.getValidationStatus();
console.warn(status);
```

#### Získání stavu inputů uvnitř formuláře
```javascript
var status = Foxentry.getValidationStatus("#orderForm");
console.warn(status.isValid);
console.warn(status.inputs);
```

#### Typ návratové hodnoty
```typescript
export interface InputValidationInfo {
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

#### Popis atributů InputsValidationStatus
- `inputs` - seznam všech namapovaných inputů v daném rozsahu a jejich aktuálních validačních stavů
- `isValid` - informace, zda jsou všechny zahrnuté inputy aktuálně validní
- `isLoading` - informace, zda u některého ze zahrnutých inputů právě probíhá validace

#### Popis atributů InputValidationInfo
- `isValid` - informace, zda je konkrétní input aktuálně vyhodnocen jako validní
- `isLoading` - informace, zda u konkrétního inputu právě probíhá validace
- `mark` - stav inputu; může nabývat hodnot `valid`, `invalid`, `alert` nebo `null`, pokud není nastavena žádná značka
- `message` - validační zpráva pro daný input; pokud není k dispozici, vrací `null`
- `ref` - reference na konkrétní DOM element inputu

#### Příklad návratové hodnoty
```json
{
  "inputs": [
    {
      "isValid": true,
      "isLoading": false,
      "mark": "valid",
      "message": null,
      "ref": "HTMLElement"
    },
    {
      "isValid": false,
      "isLoading": false,
      "mark": "invalid",
      "message": "Zadaná hodnota není validní",
      "ref": "HTMLElement"
    }
  ],
  "isValid": false,
  "isLoading": false
}
```

## Typ parent: HTMLElementOrSelector

Metody `validate()` a `getValidationStatus()` mohou přijímat nepovinný parametr `parent`, který omezuje zpracování pouze na namapované inputy uvnitř zadaného rodiče.

```typescript
type HtmlElementOrSelector = HTMLElement | JQuery | string | null
```

Předat je možné:

- `HTMLElement` - konkrétní DOM element, například formulář nebo kontejner
- `JQuery` - jQuery objekt obsahující rodičovský element
- `string` - CSS selector, podle kterého se rodičovský element dohledá
- `null` - prázdná hodnota; prakticky stejné použití, jako když parametr nepředáte

Pokud parametr `parent` nepředáte, budou zpracovány všechny namapované inputy na stránce.

#### Příklady hodnot parent
```javascript
var formElement = document.querySelector("#orderForm");
var jqueryForm = $("#orderForm");
var selector = "#orderForm";
var emptyParent = null;
```

## Nastavení callbacků pro jednotlivé validátory

Metoda `setCallbacks()` slouží k nastavení callback funkcí pro jednotlivé typy validátorů. Callback je zavolán po proběhnutí validace daného validátoru a umožňuje dále pracovat s výsledkem validace v Javascriptu.

Callbacky můžete nastavit přímo, nebo uvnitř `onFoxentryProjectLoad()`, kterou knihovna ve v2 po načtení automaticky vyvolá.

#### Typ callbacků
```typescript
export type FoxentryCallback = (input: HTMLElement, validation: ValidationCallbackValidationArgument) => unknown;

export type FoxentryCallbacksType = {
  [validatorType in ValidatorTypes]: FoxentryCallback;
};

export interface ValidationCallbackValidationArgument {
  group: GroupValidationInfo;
}

export interface GroupValidationInfo {
  isValid: boolean;
  isLoading: boolean;
  validatorType: ValidatorTypes;
  inputs: Array<InputValidationInfo>;
}
```

První argument callbacku `input` obsahuje referenci na input, nad kterým validace proběhla. Druhý argument `validation` obsahuje objekt typu `ValidationCallbackValidationArgument`, ve kterém je výsledek validace dostupný v atributu `group`.

#### Popis atributů ValidationCallbackValidationArgument
- `group` - detail validační skupiny, do které patří právě validovaný input

#### Popis atributů GroupValidationInfo
- `isValid` - informace, zda je skupina inputů aktuálně vyhodnocena jako validní
- `isLoading` - informace, zda u skupiny inputů právě probíhá validace
- `validatorType` - typ validátoru, ke kterému callback patří
- `inputs` - seznam inputů zahrnutých do dané validační skupiny, včetně jejich aktuálního stavu

#### Dostupné typy validátorů
Knihovna podporuje tyto typy validátorů:

- `email`
- `phone`
- `company`
- `name`
- `location`

#### Nastavení callbacků
```javascript
function onFoxentryProjectLoad() {
  function emailCallback(input, validation) {
    console.warn("Email input:", input);
    console.warn("Výsledek validace skupiny:", validation.group);
  }

  function phoneCallback(input, validation) {
    console.warn("Telefonní input:", input);
    console.warn("Typ validátoru:", validation.group.validatorType);
    console.warn("Validované inputy:", validation.group.inputs);
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

## Migrace z v1 na v2

Tato sekce popisuje nejdůležitější rozdíly při migraci z v1 na v2. Největší změny se týkají práce s callbacky.

### 1. Změna registrace callbacků

Ve v1 se callbacky typicky nastavovaly přes `FoxentryBuilder.setCallbacks(...)` uvnitř `onFoxentryProjectLoad()`.
Ve v2 zůstává `onFoxentryProjectLoad()` dostupná, ale callbacky je potřeba registrovat přes `Foxentry.setCallbacks(...)`.

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

### 2. Změna formátu obou argumentů callbacku

Ve v1 callback také přijímal dva argumenty (`data`, `validatorInfo`).
Ve v2 callback nadále používá dva argumenty, ale oba mají nový formát:

```typescript
(input: HTMLElement, validation: ValidationCallbackValidationArgument) => unknown
```

- `input` - první argument je nyní přímo DOM element (`HTMLElement`) validovaného inputu (ve v1 byl první argument `data`)
- `validation.group` - druhý argument je nyní objekt typu `ValidationCallbackValidationArgument` (ve v1 to byl `validatorInfo`)

Názvy argumentů si můžete ponechat i původní (`data`, `validatorInfo`). Důležité je upravit práci s jejich obsahem podle formátu v2.

### 3. Změna názvu validátoru `address` -> `location`

Pokud ve v1 používáte callback pro adresu (`address`), ve v2 je potřeba použít klíč `location`.

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

### 4. Doporučený migrační postup

1. Funkci `onFoxentryProjectLoad()` můžete zachovat; knihovna ji ve v2 po načtení také zavolá.
2. Uvnitř `onFoxentryProjectLoad()` nahraďte `FoxentryBuilder.setCallbacks(...)` za `Foxentry.setCallbacks(...)`.
3. Upravte práci s daty: první argument je DOM element a druhý argument je objekt, jehož výsledek je v `validation.group`.
4. Přejmenujte callback klíč `address` na `location`.
5. Po změnách ověřte chování voláním `await Foxentry.validate(...)` a kontrolou přes `Foxentry.getValidationStatus(...)`.
