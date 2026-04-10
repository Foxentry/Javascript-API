# Foxentry Javascript API - dokumentace

## Verze dokumentace

### v2 (aktuální)
Aktuální verze knihovny, kompatibilní s novou aplikací ([app.foxentry.com](https://app.foxentry.com/))

- [Přejít na dokumentaci v2](#foxentry-javascript-api-v2)

### Migrace z v1 na v2
Postup a rozdíly při přechodu z legacy implementace na aktuální API v2.

- [Přejít na návod k migraci](#migrace-z-v1-na-v2)

### v1 (legacy)
Původní verze knihovny, kompatibilní s původní aplikací ([old.app.foxentry.com](https://old.app.foxentry.com/)).

- [Přejít na dokumentaci v1](#foxentry-javascript-api-v1)

---

## Foxentry Javascript API v2

> Tato sekce je vyhrazena pro dokumentaci nové verze knihovny (v2).
> Pokud aktuálně používáte původní implementaci, pokračujte na [dokumentaci v1](#foxentry-javascript-api-v1).

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
    address: function (validatorResponse) {
      console.warn(validatorResponse);
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

### 2. Změna signatury callbacku

Ve v1 callback obvykle přijímal jeden argument (např. `validatorResponse`).
Ve v2 callback přijímá dva argumenty:

```typescript
(input: HTMLElement, validation: ValidationCallbackValidationArgument) => unknown
```

- `input` - input element, nad kterým validace proběhla
- `validation.group` - výsledek validační skupiny (`isValid`, `isLoading`, `validatorType`, `inputs`)

### 3. Změna názvu validátoru `address` -> `location`

Pokud ve v1 používáte callback pro adresu (`address`), ve v2 je potřeba použít klíč `location`.

#### v1
```javascript
FoxentryBuilder.setCallbacks({
  address: function (validatorResponse) {
    console.warn(validatorResponse);
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
3. Upravte callbacky na signaturu `(input, validation)`.
4. V kódu callbacků přejděte z původního `validatorResponse` na `validation.group`.
5. Přejmenujte callback klíč `address` na `location`.
6. Po změnách ověřte chování voláním `await Foxentry.validate(...)` a kontrolou přes `Foxentry.getValidationStatus(...)`.

## Foxentry Javascript API v1

Foxentry Javascript API umožňuje našeptávať a validovať údaje pomocou metód priamo v Javascripte. Nepotrebujete teda napájať Foxentry na inputy vo vašom formulári pomocou predvolených našeptávačov/validátorov, ale môžete si vytvoriť vlastnú implementáciu Foxentry funkcionality do svojho webu.

**Pre implementáciu Javascript APi je potrebné mať vytvorený Foxentry projekt na [app.foxentry.cz](https://app.foxentry.cz) a mať v ňom dostatok kreditov. Do HTML kódu webu musí byť vložený integračný javascript kód, ktorý nájdete vo svojom Foxentry projekte v sekcii Integrácia (v menu vpravo hore).**

Našeptávanie/validácia prebieha asynchrónne, je teda potrebné použiť **callback funkciu**, ktorá výstup zo serveru spracuje a vyhodnotí. 
Výstupom každej metódy je objekt s údajmi o validácii a jej výsledkoch. Zároveň tento výstup obsahuje aj informáciu o spotrebe kreditov (počte kreditov pre požiadavkou, po požiadavke a počet kreditov, ktoré požiadavka potrebovala).

## Rýchla validácia formuláru
Umožňuje vyhodnotiť, či je formulár vyplnený správne a či neobsahuje nejaké nesprávne vyplnené inputy. Validácia berie do úvahy len inputy nakonfigurované v nastaveniach projektu v administrácii Foxentry a zohľadňuje u každého inputu jeho nastavenie "**Propouštět jen validní údaje?**" nachádzajúce sa v nastaveniach projektu. V prípade, že je nastavenie nastavené na hodnotu "**Ano**", je validita údaju v tomto inpute posudzovaná veľmi striktne a v prípade, že údaj nie je presne zhodný s validným údajom (napr. preklep v názve ulice), to input vyhodnotí ako nevalidne vyplnený. V opačnom prípade (hodnota "**Ne**") sa input označí v prípade podľa nás nevalidného údaju žltým trojuholníkom a žltým orámovaním a v rámci validácie formuláru sa input vyhodnotí ako síce validný, no s varovaním. Toto je vo výstupe validácie formuláru zobrazené ako kombináciou parametrov valid = false a validWithWarning = true.

**Parametre**
- **form** - formulár, u ktorého chcete vyhodnotiť stav

#### Validácia formuláru
```javascript
var form = document.querySelector("#orderForm")
var validForm = Foxentry.isFormValid(form);
console.warn(validForm);
```
#### Výsledok validácie formuláru
```
true alebo false
```

## Detailnejšia validácia formuláru
Funguje na rovnakom princípe ako základná (rýchla) validácia s tým rozdielom, že poskytujú zoznam inputov, ktoré sú vyplnené správne (inputs.valid), ktoré nesprávne (inputs.invalid) a ktoré sú síce možno vyplnené správne, no nie sme si istí ich správnosťou (inputs.withWarning). Do týchto 3 zoznamov inputov sa vždy vkladá *name* parameter daného inputu.

V prípade, ak chcete zistiť, či sú napr. inputy obsahujúce adresu vyplnené správne, spustite túto detailnejšiu validáciu formuláru a skontrolujte, či sa dané inputy (hodnoty ich name parameterov) nachádzajú v zozname validných inputov (inputs.valid).

#### Validácia formuláru
```javascript
var form = document.querySelector("#orderForm")
var validation = Foxentry.formValidation(form);
console.warn(validation);
```
#### Výsledok validácie formuláru
```json
{
  "valid":false,
  "validWithWarnings":false,
  "inputs":{
    "valid":[
      "addressStreet",
      "addressCity",
      "addressZip",
      "addressStreetNumber"
    ],
    "invalid":[
      "email",
      "phone"
    ]
    "withWarning":[
      "name", 
      "surname"
    ]
  }
}
```

## Aktivácia / deaktivácia Foxentry funkcionality
Foxentry je možné v prípade potreby deaktivovať a neskôr znova aktivovať pomocou javascriptu.

**Upozornenie:** Deaktivácia Foxentry má vplyv iba na našeptávanie/validáciu údajov priamo vo formulári. Nemá vplyv na API metódy (popísané nižšie), ktoré sú dostupné aj po deaktivácii.

```javascript
Foxentry.deactivate();
Foxentry.activate();
```

## Nastavenie callbacku pri validácii údaju
V niektorých prípadoch je potrebné pri validácii údajov zadaných do formuláru s nimi ďalej pracovať. V takom prípade je potrebné zadefinovať tzv. callbacky, teda funkcie, ktoré sa spustia po validácii zadanéh údaju a ktorým Foxentry validátor poskytne všetky potrebné informácie o validácii (teda, či je údaj validný alebo nie, v prípade, že áno, dodá aj detailnejšie informácie napr. o konkrétnom adresnom bode alebo firme).

Pre pridanie callbacku je potrebné **pridať do kódu JS funkciu** s presne zadefinovaným názvom "**onFoxentryProjectLoad** a v nej definíciu callbackov:
```javascript
function addressValidationHandler(validatorResponse){
  console.warn(validatorResponse);
}

function onFoxentryProjectLoad(){
  FoxentryBuilder.setCallbacks(
    {
      "address" : addressValidationHandler
    }
  );
}
```
Foxentry funkcionalita v sebe obsahuje 5 typov validátorov a preto je dostupný callback pre každý z nich: address, company, email, name, phone.
