# Foxentry Javascript API

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
~~V niektorých prípadoch je potrebné po načítaní webovej stránky celé Foxentry reštartovať, napr. v prípade, ak sa formulár, na ktorého inputy sa má Foxentry aktivovať, nenachádza na stránke (v HTML kóde) v čase spustenia Foxentry (po načítaní stránky). Foxentry momentálne **nepodporuje automatickú inicializáciu funkcionality na inputoch pridaných po svojom spustení**, preto je potrebné ho reštartovať pomocou deaktivácie a následnej aktivácie.~~

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

## Validácia adresného bodu
Umožňuje overiť, či zadaná adresa existuje.

**Parametre**
- **query** - obsahuje zoznam overovaných informácií, pričom podporované sú:
  - **street** - ulica
  - **number** - číslo orientačné/popisné
  - **city** - mesto
  - **zip** - PSČ
  - **streetWithNumber** - kombinácia názvu ulice a čísla orientačného/popisného
- **options** - object s nastaveniami validácie:
  - **country** - kód štátu, v rámci ktorého sa má údaj validovať (české alebo slovenské adresy), možné hodnoty sú "*cz*" a "*sk*"
  - **addressCityExtended** - určuje, v akom formáte má byť názov mesta vo výstupe. V prípade hodnoty *false* bude názov mesta napr. "Praha 3", v prípade hodnoty *true* bude názov mesta napr. "Praha 3 - Žižkov"
  - **zipFormat** - určuje, v akom formáte má byť PSČ vo výstupe. Dostupné sú hodnoty *default* (PSČ bez medzier) alebo *spaced* (PSČ s medzerou medzi 3. a 4. číslicou)
- **callback** - funkcia (metóda), ktorá bude spracúvať výstup z validátora. Validátor túto metódu zavolá po validácii a vloží do nej stav validácie ako jej prvý parameter

### Validácia adresy s oddelenou ulicou a číslom

#### Validácia
```javascript
function addressValidationHandler(apiResponse) {
  console.warn(apiResponse);
  console.warn("Is address valid: "+apiResponse.data.valid);
}

Foxentry.api.address.validate(
  {
    street : "Jeseniova",
    number : "1151/55",
    city : "Praha",
    zip : "13000"
  },
  { 
    country : "cz",
    addressCityExtended: true,
    zipFormat: "spaced"
  }, 
  addressValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "valid":true,
    "data":{
      "id":21778825,
      "country":"cz",
      "city":"Praha 3",
      "city.name":"Praha",
      "city.population":1280508,
      "cityPart":"Žižkov",
      "cityWithCityPart":"",
      "cityPartWithDistrict": "Praha 3 - Žižkov",
      "cityPartWithDistrictPrague":"Praha 3 - Žižkov",
      "cityDistrict":"Praha 3",
      "cityDistrictPrague":"Praha 3",
      "street":"Jeseniova",
      "streetNumber":"1151/55",
      "streetNumber.number1":"1151",
      "streetNumber.number2":"55",
      "streetNumber.letter":"",
      "zip":"13000",
      "gps":"50.087435280377,14.46327549904",
      "gps.lat":50.087435280377,
      "gps.lon":14.46327549904,
      "streetWithNumber":"Jeseniova 1151/55"
    }
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":0,
    "creditsAfter":1000
  }
}
```

### Validácia adresy s oddelenou ulicou a číslom

#### Validácia
```javascript
function addressValidationHandler(apiResponse) {
  console.warn(JSON.stringify(apiResponse));
  console.warn("Is address valid: "+apiResponse.data.valid);
}

Foxentry.api.address.validate(
  {
    streetWithNumber : "Jeseniova 1151/55",
    city : "Praha",
    zip : "13000"
  },
  { 
    country : "cz",
    addressCityExtended: true,
    zipFormat: "spaced"
  }, 
  addressValidationHandler  
);
```
#### Výstup validácie
```json
Rovnaký ako v prípade validácie s oddelenou ulicou a číslom.
```

### Validácia neúplnej adresy
Vráti informáciu, či daný údaj alebo kombinácia údajov je validná. Napríklad umožňuje zistiť, či kombinácia ulice *Slovenská* a mesta *Praha* je možná (teda či táto ulica existuje v tomto meste)

#### Validácia
```javascript
function addressValidationHandler(apiResponse) {
  console.warn(JSON.stringify(apiResponse));
  console.warn("Street in city exists: "+apiResponse.data.valid);
}

Foxentry.api.address.validate(
  {
    street : "Jeseniova",
    city : "Praha"
  },
  { 
    country : "cz",
    addressCityExtended: true,
    zipFormat: "spaced"
  }, 
  addressValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
	"data":{
	  "valid":true,
    "validAll":false,
    "data":{
       "street":"Jeseniova",
       "city":"Praha"
     }
   },
   "usage":{
      "creditsBefore":1000,
      "creditsUsed":0,
      "creditsAfter":1000
   }
}
```

### Validácia neúplnej adresy, ktorej výstupom je konkrétny adresný bod
V prípade, že zadané parametre dostatočne identifikujú konkrétny existujúci adresný bod, API vráti informácie o tomto adresnom bode. Napríklad kombinácia ulice *Jeseniova 55* a mesta *Praha* vráti konkrétny adresný bod *Jeseniova 1151/55, 130 00 Praha 3" aj napriek tomu, že nebolo zadané celé číslo domu ani PSČ.

#### Validácia
```javascript
function addressValidationHandler(apiResponse) {
  console.warn(JSON.stringify(apiResponse));
  console.warn("Street in city exists: "+apiResponse.data.valid);
}

Foxentry.api.address.validate(
  {
    streetWithNumber : "Jeseniova 55",
    city : "Praha"
  },
  { 
    country : "cz",
    addressCityExtended: true,
    zipFormat: "spaced"
  }, 
  addressValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "valid":true,
    "validAll":true,
    "data":{
      "id":21778825,
      "country":"cz",
      "city":"Praha 3",
      "city.name":"Praha",
      "city.population":1280508,
      "cityPart":"Žižkov",
      "cityWithCityPart":"",
      "cityPartWithDistrict":"Praha 3 - Žižkov",
      "cityPartWithDistrictPrague":"Praha 3 - Žižkov",
      "cityDistrict":"Praha 3",
      "cityDistrictPrague":"Praha 3",
      "street":"Jeseniova",
      "streetNumber":"1151\/55",
      "streetNumber.number1":"1151",
      "streetNumber.number2":"55",
      "streetNumber.letter":"",
      "zip":"13000",
      "gps":"50.087435280377,14.46327549904",
      "gps.lat":50.087435280377,
      "gps.lon":14.46327549904,
      "streetWithNumber":"Jeseniova 1151\/55"
    }
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":0,
    "creditsAfter":1000
  }
}
```

## Validácia emailovej adresy
```javascript
Foxentry.api.email.validate(email, options, callback);
```
Parametre
- **email** - emailová adresa, ktorú chcete zvalidovať _(string)_
- **options** - objekt s nastaveniami validácie:
  - **validationType** - "basic" (základná validácia, validuje sa iba formát emailovej adresy) alebo "extended" (kontroluje sa existencia domény, nastavenie jej MX záznamov a reálna existencia emailovej adresy na doméne)
- **callback** - funkcia (metóda), ktorá bude spracúvať výstup z validátora. Validátor túto metódu zavolá po validácii a vloží do nej stav validácie ako jej prvý parameter

### Ukážka validácie emailovej adresy
#### Validácia
```javascript
function emailValidationHandler(apiResponse) {
  console.warn(apiResponse);
  console.warn("Is email valid: "+apiResponse.data.valid)
}

Foxentry.api.email.validate(
  "info@foxentry.cz", 
  {
    validationType : "basic"
  }, 
  emailValidationHandler  
);
```

#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "valid":true
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":1,
    "creditsAfter":999
  }
}
```

## Validácia telefónneho čísla
Metóda zvaliduje zadané telefónne číslo a vráti informáciu, či je telefónne číslo validné alebo nie. V prípade, že áno, vráti základné informácie o telefónnom čísle (krajina, jeho formát a podobne) a v prípade rozšírenej validácie aj informáciu o jeho aktuálnom mobilnom operátorovi. **Rozšírená validácia nie je dostupná pre projekty, ktoré nemajú zakúpený balíček kreditov.**
```javascript
Foxentry.api.phone.validate(phoneNumber, phonePrefix, options, callback);
```
Parametre
- **phoneNumber** - telefónne číslo, ktoré chcete zvalidovať _(string)_. Môže, no nemusí obsahovať medzinárodnú predvoľbu. V prípade, že ju obsahuje, bude parameter phonePrefix rovný _null_.
- **phonePrefix** - medzinárodná predvoľba telefónneho čísla (napr. +420 alebo +421), nutné zadať v prípade, že ju telefónne číslo zadané do phoneNumber parametra neobsahuje
- **options** - objekt s nastaveniami validácie:
  - **validationType**
    - "basic" (základná validácia, validuje sa iba formát telefónneho čísla)
    - "extended" (rozšírená validácia, kontroluje sa reálna existencia telefónneho čísla a jeho mobilný operátor)
- **callback** - funkcia (metóda), ktorá bude spracúvať výstup z validátora. Validátor túto metódu zavolá po validácii a vloží do nej stav validácie ako jej prvý parameter

### Ukážka validácie telefónneho čísla
#### Validácia
```javascript
function phoneValidationHandler(apiResponse) {
  console.warn(apiResponse);
  console.warn("Is phone valid: "+apiResponse.data.valid)
}

Foxentry.api.phone.validate(
  "607123456",
  "+420",
  {
    validationType : "basic"
  }, 
  phoneValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "data":{
      "type":"MOBILE",
      "format":{
        "internationalFormatted":"+420 607 123 456",
        "international":"+420607123456",
        "nationalFormatted":"607 123 456",
        "national":"607123456"
      },
      "country":{
        "code":"CZ",
        "prefix":"+420"
      },
      "location":{
        "cz":"Česko",
        "sk":"Česko",
        "en":"Czechia"
       },
       "phonePrefix":"+420",
       "phoneNumber":"607123456"
     },
     "valid":true
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":1,
    "creditsAfter":999
  }
}
```

## Validácia mena alebo priezviska
Umožňuje overiť, či meno alebo priezvisko existuje (vyskytuje sa v danej krajine).
```
Foxentry.api.name.validate(query, options, callback);
```
**Parametre**
- **query** - obsahuje samotný dotaz na API, pričom je možné validovať meno (parameter name), priezvisko (parameter surname) alebo kombináciu mena a priezviska (parameter nameSurname). 
- **options** - object s nastaveniami validácie:
  - **country** - kód štátu, v rámci ktorého sa má údaj validovať (české a slovenské mena a priezviská sa líšia), možné hodnoty sú "*cz*" a "*sk*"
- **callback** - funkcia (metóda), ktorá bude spracúvať výstup z validátora. Validátor túto metódu zavolá po validácii a vloží do nej stav validácie ako jej prvý parameter
  
### Základná validácia mena
#### Validácia
```javascript
function nameValidationHandler(apiResponse) {
  console.warn(apiResponse);
  console.warn("Is name valid: "+apiResponse.data.name.valid)
}

Foxentry.api.name.validate(
  {
    name : { 
      value : "Petr" 
    }
  },
  { 
    country : "cz"
  }, 
  nameValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "name":{
      "valid":true,
      "data":{
        "name":"Petr",
        "gender":1,
        "vocativeName":"Petře"
      }
    }
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":1,
    "creditsAfter":999
  }
}
```

### Validácia kombinácie mena a priezviska
#### Validácia
```javascript
function nameValidationHandler(apiResponse) {
  console.warn(apiResponse);
  console.warn("Is name and surname valid: "+apiResponse.data.nameSurname.valid);
}

Foxentry.api.name.validate(
  {
    nameSurname : { 
      value : "Petr Novák",
    }
  },
  { 
    country : "cz"
  }, 
  nameValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "nameSurname":{
      "valid":true,
      "data":{
        "nameSurname":"Petr Novák",
        "vocativeNameSurname":"Petře Nováku"
      }
    }
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":2,
    "creditsAfter":998
  }
}
```
## Validácia informácií o firme
Umožňuje overiť, či zadané údaje o firme sú platné.

**Parametre**
- **query** - obsahuje zoznam overovaných informácií, pričom podporované sú:
  - **name** - názov firmy
  - **registrationNumber** - IČ firmy
  - **taxNumber** - DIČ firmy
- **options** - object s nastaveniami validácie:
  - **country** - kód štátu, v rámci ktorého sa má údaj validovať (české alebo slovenské firmy), možné hodnoty sú "*cz*" a "*sk*"
- **callback** - funkcia (metóda), ktorá bude spracúvať výstup z validátora. Validátor túto metódu zavolá po validácii a vloží do nej stav validácie ako jej prvý parameter

### Validácia firmy podľa názvu
Vráti informáciu, či firma s takýmto názvom existuje. Ak áno, vráti základné informácie o firme (názov, IČ, DIČ a adresu).

#### Validácia
```javascript
function companyValidationHandler(apiResponse) {
  console.warn(JSON.stringify(apiResponse));
  console.warn("Is company valid: "+apiResponse.data.valid);
}

Foxentry.api.company.validate(
  {
    name : "Avantro s.r.o."
  },
  { 
    country : "cz"
  }, 
  companyValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "valid":true,
    "validAll":true,
    "validInputs":[
      "name"
    ],
    "invalidInputs":[],
    "id":1639909,
    "data":{
      "id":1639909,
      "name":"AVANTRO s.r.o.",
      "registrationNumber":"04997476",
      "taxNumber":"CZ04997476",
      "vatNumber":"",
      "address.full":"Jeseniova 1151/55, Praha",
      "address.city":"Praha",
      "address.street":"Jeseniova",
      "address.streetNumber":"1151/55",
      "address.streetWithNumber":"Jeseniova 1151/55",
      "address.zip":"130 00",
      "address.id":21778825
    }
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":2,
    "creditsAfter":998
  }
}
```

### Validácia firmy podľa názvu a IČ
Vráti informáciu, či firma s takouto kombináciou názvu a IČ existuje.

#### Validácia
```javascript
function companyValidationHandler(apiResponse) {
  console.warn(JSON.stringify(apiResponse));
  console.warn("Is company valid: "+apiResponse.data.valid);
}

Foxentry.api.company.validate(
  {
    name : "Foxentry Javascript API s.r.o.",
    taxNumber : "123456789"
  },
  { 
    country : "cz"
  }, 
  companyValidationHandler  
);
```
#### Výstup validácie
```json
{
  "success":true,
  "data":{
    "valid":false
  },
  "usage":{
    "creditsBefore":1000,
    "creditsUsed":2,
    "creditsAfter":998
  }
}
```


