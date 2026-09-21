# Fix TouchTune per Mazda Connect 74.00.324A

## Introduzione

Questo documento descrive l'analisi eseguita sul firmware Mazda Connect
**74.00.324A EU N** e il problema riscontrato utilizzando TouchTune per
disabilitare la limitazione del touchscreen durante la marcia.

La patch originale riconosceva correttamente il firmware e `Common.js`, ma
l'installazione poteva interrompersi quando le chiavi NVRAM relative alle
speed restriction non erano ancora presenti.

L'analisi del firmware originale ha permesso di verificare il comportamento
previsto dalla CMU e di individuare una soluzione.

Il fix descritto qui è stato successivamente testato con successo su una CMU
reale con firmware 74.00.324A.


## 1. Firmware analizzato

L'analisi tecnica è stata effettuata sulla versione:

    Mazda Connect 74.00.324A EU N

L'esame del software della CMU ha permesso di verificare il comportamento
delle impostazioni NVRAM interessate dal funzionamento di TouchTune.

Il firmware Mazda e i relativi file proprietari non sono distribuiti da
questo repository.


## 2. Analisi del software della CMU

L'analisi del filesystem della CMU è stata utilizzata esclusivamente per
comprendere il comportamento delle impostazioni coinvolte.

Questo repository non contiene né distribuisce immagini del filesystem,
firmware Mazda o altri file proprietari estratti dal sistema.


## 3. Analisi del touchscreen

L'analisi del software della CMU ha permesso di individuare la parte del
sistema coinvolta nella gestione delle limitazioni dell'interfaccia durante
il movimento del veicolo.

In particolare, è stato verificato che TouchTune interviene sul file:

    /jci/gui/common/js/Common.js

L'analisi di `Common.js` ha confermato che TouchTune modifica la gestione
dell'evento `Global.AtSpeed`, coinvolto nel comportamento dell'interfaccia
in relazione allo stato di movimento del veicolo.

La modifica impedisce quindi all'interfaccia di applicare normalmente la
limitazione del touchscreen associata a tale evento.

In questo documento viene descritto esclusivamente il comportamento
individuato durante l'analisi. Il codice originale Mazda contenuto in
`Common.js` non viene riprodotto né distribuito da questo repository.


## 4. Verifica di Common.js

È stato calcolato l'SHA-256 del `Common.js` originale estratto direttamente
dal firmware Mazda 74.00.324A:

    376b30a46366a543122956d7feb1b44f147425015837a4d83fedf12a67943351

Il valore coincide esattamente con:

    MZD_PROFILE_STOCK_SHA256

presente in TouchTune.

È stata quindi applicata manualmente la stessa trasformazione eseguita da
TouchTune.

SHA-256 risultante:

    019eba18e8d629ddb1d55563aab138ce1eb3329fab67b71d9b4178377cf9d9ce

Anche questo valore coincide esattamente con:

    MZD_PROFILE_PATCHED_SHA256

di TouchTune.

Questo ha confermato che la modifica di `Common.js` prevista da TouchTune è
corretta per il firmware analizzato.


## 5. Le speed restriction della CMU

L'analisi del software della CMU ha permesso di verificare che il sistema
Mazda gestisce due impostazioni NVRAM relative alle limitazioni durante la
marcia:

    bus_bcm_speed_restriction
    lvds_speed_restriction

Gli script presenti nel sistema sono in grado di gestire queste impostazioni
anche quando le relative chiavi NVRAM non sono ancora presenti.

In particolare, l'analisi ha mostrato che l'assenza iniziale di una chiave
non rappresenta necessariamente una condizione di errore: il software della
CMU può creare la configurazione necessaria durante l'applicazione
dell'impostazione.

Questa osservazione è stata determinante per individuare la causa del
fallimento dell'installer originale di TouchTune.

Il codice e gli script originali Mazda analizzati durante questa verifica
non sono riprodotti né distribuiti da questo repository.


## 6. Il problema individuato

La versione originale di TouchTune verificava che entrambe le chiavi NVRAM
fossero leggibili prima di procedere con la modifica.

Sulla CMU utilizzata durante il test le chiavi:

    bus_bcm_speed_restriction
    lvds_speed_restriction

non erano inizialmente presenti.

TouchTune interpretava questa condizione come un errore e interrompeva
l'operazione.

L'analisi del comportamento della CMU ha invece mostrato che l'assenza
iniziale delle chiavi è compatibile con il normale meccanismo utilizzato
dal sistema per configurarle.

Il problema non era quindi l'impossibilità di modificare le impostazioni,
ma il controllo preliminare effettuato da TouchTune prima che la
configurazione potesse essere applicata.


## 7. Il fix

Quando una delle due chiavi non è presente, il fix considera come stato
iniziale:

    enable

Ad esempio:

    if ! ENTRY_BUS_STATE=$(mzd_read_nvram bus_bcm_speed_restriction); then
        ENTRY_BUS_STATE=enable
        mzd_log "WARN: bus_bcm_speed_restriction missing; assuming factory state enable"
    fi

Lo stesso comportamento viene applicato a:

    lvds_speed_restriction

È stato inoltre adattato il controllo di preflight.

Se la chiave continua a non esistere immediatamente prima della scrittura,
il suo stato viene interpretato coerentemente come `enable`:

    PREFLIGHT_BUS_STATE=$(mzd_read_nvram bus_bcm_speed_restriction 2>/dev/null || printf '%s\n' enable)
    PREFLIGHT_LVDS_STATE=$(mzd_read_nvram lvds_speed_restriction 2>/dev/null || printf '%s\n' enable)

Questo evita che il controllo di sicurezza interpreti l'assenza della chiave
come una modifica inattesa dello stato durante il preflight.


## 8. Perché il fix funziona

Il punto importante emerso dall'analisi del firmware Mazda è che l'assenza
della chiave NVRAM non impedisce necessariamente di configurarla.

Gli script originali Mazda eseguono infatti esplicitamente l'operazione di
`add` prima di scrivere la configurazione.

TouchTune può quindi lasciare agli script originali Mazda il compito di
creare/configurare:

    bus_bcm_speed_restriction
    lvds_speed_restriction

invece di interrompere anticipatamente l'installazione soltanto perché la
chiave non è ancora leggibile.


## 9. Risultato del test

Configurazione testata:

- Mazda Connect / MZD Connect
- firmware EU 74.00.324A
- `Common.js` originale verificato tramite SHA-256
- chiavi speed restriction inizialmente non disponibili
- installazione TouchTune con gestione delle chiavi mancanti

Risultato:

**patch installata correttamente e touchscreen funzionante durante la marcia.**


## 10. Avvertenze e finalità del progetto

Questo repository è pubblicato esclusivamente a scopo didattico, di studio,
ricerca e documentazione tecnica.

Il contenuto descrive un'attività sperimentale svolta su una specifica
configurazione Mazda Connect 74.00.324A. Le informazioni riportate hanno lo
scopo di documentare il problema osservato, il processo di analisi e la
modifica sperimentale apportata al progetto open-source TouchTune.

Questo repository non costituisce documentazione ufficiale Mazda, non è
affiliato, approvato o supportato da Mazda e non deve essere interpretato
come una guida o una raccomandazione per la modifica di veicoli o sistemi
infotainment.

L'autore non raccomanda l'applicazione della modifica descritta su veicoli
destinati alla circolazione stradale.

La modifica delle configurazioni o del software della CMU può causare
malfunzionamenti, perdita di funzionalità, necessità di ripristino del sistema
o altri comportamenti imprevisti.

La modifica descritta può inoltre alterare limitazioni dell'interfaccia
previste dal costruttore. Tali limitazioni possono avere finalità legate alla
sicurezza e alla riduzione della distrazione del conducente.

L'utilizzo di un touchscreen o di altri sistemi infotainment durante la guida
può costituire una distrazione ed è responsabilità dell'utilizzatore rispettare
le norme applicabili e utilizzare il veicolo in condizioni di sicurezza.

Non viene fornita alcuna garanzia di compatibilità con altre versioni firmware,
varianti regionali, configurazioni hardware o veicoli diversi da quello
utilizzato nell'attività sperimentale descritta.

Il firmware Mazda, la RootFS e gli altri file proprietari analizzati durante
lo studio non sono inclusi né distribuiti da questo repository.

Il software e la documentazione sono forniti "così come sono", senza garanzie
espresse o implicite. L'autore non si assume responsabilità per danni,
malfunzionamenti, perdita di dati, indisponibilità del sistema o altre
conseguenze derivanti dall'utilizzo delle informazioni o del software
contenuti nel repository.

Chiunque scelga autonomamente di utilizzare, modificare o sperimentare il
software lo fa sotto la propria esclusiva responsabilità e deve verificare
preventivamente la conformità alle leggi, ai regolamenti e alle condizioni
applicabili nel proprio Paese.


## Crediti

TouchTune originale:

https://github.com/Miatafy/TouchTune

Il fix documentato in questa repository nasce dall'analisi del firmware
Mazda Connect 74.00.324A e dal debugging del comportamento delle chiavi NVRAM
`bus_bcm_speed_restriction` e `lvds_speed_restriction`.
