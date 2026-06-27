# HYPie — formulieren laten binnenkomen

De pagina stuurt aanmeldingen, nieuwsbrief-inschrijvingen en voordrachten via één
centrale `dispatch`-functie. Bovenaan het `<script>` in `index.html` staat:

```js
var CONFIG = {
  ENDPOINT: '',                 // <-- hier plak je je URL
  CONTACT_EMAIL: 'info@hypie.nl'
};
```

- **ENDPOINT leeg** → alles valt terug op een vooringevulde e-mail (mailto). Werkt
  direct, maar opent de mailapp van de bezoeker.
- **ENDPOINT ingevuld** → inzendingen komen stil en automatisch binnen (aanbevolen).

Je hoeft verder niets in de code te veranderen. Kies optie A of B.

---

## Optie A — Formspree (snelst, geen code)

1. Maak een gratis account op **formspree.io**.
2. Maak een nieuw formulier aan. Je krijgt een endpoint-URL, bijv.
   `https://formspree.io/f/xxxxxxx`.
3. Plak die URL in `CONFIG.ENDPOINT`:
   ```js
   ENDPOINT: 'https://formspree.io/f/xxxxxxx',
   ```
4. Klaar. De pagina stuurt JSON met `Accept: application/json` (al ingebouwd), dus
   inzendingen verschijnen in je Formspree-dashboard en/of mailbox.

Let op: het gratis plan kent een maandlimiet (±50 inzendingen). Eén endpoint vangt
alle drie de formuliertypes op; in elk bericht staat een veld `type`
(`aanmelding` / `nieuwsbrief` / `voordracht`) zodat je ze kunt onderscheiden.

---

## Optie B — Google Sheet via Apps Script (gratis, onbeperkt, eigen beheer)

1. Maak een nieuwe Google Sheet.
2. Ga naar **Extensies → Apps Script** en plak onderstaande code.
3. Pas eventueel de kolomvolgorde aan.
4. **Implementeren → Nieuwe implementatie → Web-app**. Zet "Uitvoeren als: ikzelf"
   en "Toegang: iedereen". Kopieer de web-app-URL (eindigt op `/exec`).
5. Plak die URL in `CONFIG.ENDPOINT`.

> Tip bij CORS: als de browser de POST blokkeert, stuur dan als `text/plain`.
> Wijzig daarvoor in de pagina de `Content-Type` van de fetch naar
> `'text/plain;charset=utf-8'` (Apps Script leest `e.postData.contents` toch als JSON).

```js
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Inzendingen')
             || SpreadsheetApp.getActiveSpreadsheet().insertSheet('Inzendingen');
    if (sheet.getLastRow() === 0) {
      sheet.appendRow(['Tijd','Type','Voornaam','Achternaam','E-mail','Leeftijd',
                       'Regio','Kantoor','Wens','Voorgedragen','Waarom','Door','Toestemming']);
    }
    sheet.appendRow([
      new Date(), data.type || '', data.voornaam || '', data.achternaam || '',
      data.email || '', data.leeftijd || '', data.regio || '', data.kantoor || '',
      data.wens || '', data.voorgedragen || '', data.waarom || '', data.door || '',
      data.toestemming_top50 ? 'ja' : (data.bevestigd_geinformeerd ? 'voordracht-bevestigd' : '')
    ]);
    return ContentService.createTextOutput(JSON.stringify({ok:true}))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ok:false, error:String(err)}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

---

## Velden die binnenkomen

| type        | velden                                                                 |
|-------------|------------------------------------------------------------------------|
| aanmelding  | voornaam, achternaam, email, leeftijd, regio, kantoor, wens, toestemming_top50 |
| nieuwsbrief | email                                                                  |
| voordracht  | voorgedragen, kantoor, waarom, door, bevestigd_geinformeerd            |

## Aanrader voor livegang
- Zet **double opt-in** op de nieuwsbrief (bevestigingsmail) als je e-mailtool dat ondersteunt.
- Bewaar inzendingen niet langer dan nodig (zie privacy.html) en sluit een
  verwerkersovereenkomst met Formspree of regel het zelf via je eigen Google-omgeving.
- Stuur nieuwe leden de **welkomstmail** (`hypie-welkomstmail.html`).
