# Compleanni con Età 🎂

Uno script leggero per **Google Apps Script** che legge i compleanni dai tuoi **Contatti Google** (inclusi quelli sincronizzati da dispositivi Samsung/Android) e genera automaticamente gli eventi sul calendario indicando il nome e l'età compiuta:

> Esempio: `🎂 Mario Rossi (30 anni)`

Risolve la mancanza storica di Android, Samsung Calendar e GNOME Calendar (Fedora/Linux), che mostrano solo la ricorrenza senza calcolare l'età (funzione nativa invece su iOS).

---

## 🔒 Perché è sicuro?
- **Zero app di terze parti:** non installi APK o estensioni nel browser.
- **100% Privacy:** il codice gira esclusivamente sui server di Google dentro il tuo account personale.
- **Nessuna fuga di dati:** non effettua chiamate di rete esterne (`UrlFetchApp` assente).
- **Non tocca i tuoi eventi personali:** lavora solo ed esclusivamente all'interno di un calendario secondario dedicato.

---

## 🚀 Istruzioni di installazione (5 minuti)

### 1. Crea il calendario su Google
1. Apri [Google Calendar](https://calendar.google.com) da browser su PC.
2. Nella colonna di sinistra, accanto ad **Altri calendari**, clicca su **+** > **Crea nuovo calendario**.
3. Nominalo esattamente: `Compleanni (età)` e clicca su **Crea calendario**.

---

### 2. Configura Google Apps Script
1. Vai su [script.google.com](https://script.google.com) e clicca su **Nuovo progetto**.
2. Rinomina il progetto in alto a sinistra (es. *Sincronizzatore Compleanni*).
3. Cancella il codice predefinito nell'editor e incolla lo script:
```js
function sincronizzaCompleanniConEta() {
  const NOME_CALENDARIO = "Compleanni (età)";
  const ANNI_DA_CALCOLARE = 3; // Orizzonte temporale in anni
  
  const calendari = CalendarApp.getCalendarsByName(NOME_CALENDARIO);
  if (calendari.length === 0) {
    Logger.log("Calendario non trovato. Crealo prima con il nome: " + NOME_CALENDARIO);
    return;
  }
  const cal = calendari[0];

  // Recupera i contatti con nome e data di nascita
  const response = People.People.Connections.list('people/me', {
    personFields: 'names,birthdays',
    pageSize: 1000
  });
  const contatti = response.connections || [];

  const oggi = new Date();
  const annoCorrente = oggi.getFullYear();

  contatti.forEach(persona => {
    const nome = persona.names && persona.names.length > 0 ? persona.names[0].displayName : null;
    const bday = persona.birthdays && persona.birthdays.length > 0 ? persona.birthdays[0].date : null;

    // Salta i contatti senza anno di nascita specificato
    if (!nome || !bday || !bday.year || !bday.month || !bday.day) return;

    for (let offset = 0; offset < ANNI_DA_CALCOLARE; offset++) {
      const annoEvento = annoCorrente + offset;
      const eta = annoEvento - bday.year;
      const titoloEvento = `🎂 ${nome} (${eta} anni)`;
      
      const dataEvento = new Date(annoEvento, bday.month - 1, bday.day);
      
      // Verifica se l'evento con il titolo corretto esiste già
      const eventi = cal.getEventsForDay(dataEvento);
      const esisteGia = eventi.some(e => e.getTitle() === titoloEvento);

      if (!esisteGia) {
        cal.createAllDayEvent(titoloEvento, dataEvento);
        Logger.log(`Aggiunto: ${titoloEvento}`);
      }
    }
  });
}
```
4. Nella colonna a sinistra, clicca sul pulsante **+** accanto alla voce **Servizi**.
5. Scorri l'elenco, seleziona **People API** o **Google People API** o ancora **peopleapi** (lascia l'identificatore come `People`) e clicca su **Aggiungi**.
6. Clicca sull'icona **Salva**.

---

### 3. Prima esecuzione e autorizzazioni
1. Clicca sul pulsante **Esegui** in alto.
2. Google richiederà i permessi per accedere a Contatti e Calendario:
   * Clicca su **Rivedi le autorizzazioni**.
   * Seleziona il tuo account Google.
   * Se compare l'avviso *"App non verificata"*, clicca su **Avanzate** e poi su **Apri [Nome Progetto] (non sicura)**.
   * Clicca su **Consenti**.
3. Al termine dell'esecuzione, apri Google Calendar: vedrai comparire tutti i compleanni con l'età per l'anno in corso e i successivi.

---

### 4. Automatizza gli aggiornamenti (Trigger periodico)
Per fare in modo che lo script aggiorni le età ogni anno e includa i nuovi contatti aggiunti:
1. Nella colonna sinistra dell'editor di Apps Script, clicca sull'icona della sveglia (**Attivatori**).
2. Clicca in basso a destra su **Aggiungi trigger**.
3. Configura i campi come segue:
   * **Funzione da eseguire:** `sincronizzaCompleanniConEta`
   * **Origine evento:** `Guidato dal tempo`
   * **Tipo di trigger:** `Timer per giorni` (o `Timer mensile` o `Timer settimanale`)
   * **Ora del giorno:** ad esempio `Dalla mezzanotte all'1:00`
4. Clicca su **Salva**.

---

## 📱 Nascondere i compleanni doppi

Ora che hai il nuovo calendario con l'età, disattiva quello predefinito per non avere eventi duplicati:

* **Su Samsung Calendar:** Menu laterale (tre linee) > scorri sotto il tuo account Google > **togli la spunta al vecchio "Compleanni"** e lascia attiva la spunta su **"Compleanni con Età"**.
* **Su Fedora (GNOME Calendar):** Menu calendari > disattiva l'interruttore del vecchio calendario *"Compleanni e anniversari"*.
* **Su Google Calendar (Web/App):** Sotto la colonna *"Altri calendari"*, deseleziona la casella del calendario standard *Compleanni*.

---

## 📄 Licenza
Rilasciato sotto licenza [MIT](LICENSE). Libero da usare, modificare e condividere.
