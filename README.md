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
