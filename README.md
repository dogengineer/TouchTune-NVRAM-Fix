# TouchTune — Fix NVRAM per Mazda Connect 74.00.324A

[![License: GPL-3.0-or-later](https://img.shields.io/badge/license-GPL--3.0--or--later-blue.svg)](LICENSE)
[![Firmware](https://img.shields.io/badge/Mazda%20Connect-74.00.324A-brightgreen.svg)](VERSION)

Questa repository contiene una modifica di **TouchTune by Miatafy** per mantenere
attivo il touchscreen di Mazda Connect durante la marcia.

Il progetto originale supporta Mazda Connect Gen 6 con firmware
**74.00.324 / 74.00.324A**.

Questa versione introduce inoltre un fix per un problema riscontrato su una CMU
reale con firmware **EU 74.00.324A**, nella quale le chiavi NVRAM utilizzate per
la limitazione del touchscreen non erano inizialmente presenti.

Il fix è stato **testato con successo su vettura reale**.

> Il progetto originale TouchTune è sviluppato da Miatafy.
> Questa repository mantiene i relativi crediti e la licenza GPL-3.0-or-later.

![TouchTune install prompt on Mazda Connect](docs/touchtune-install-prompt.png)


## Scopo del progetto

Questo repository documenta, a scopo didattico e di ricerca tecnica, l'analisi
di un comportamento delle impostazioni NVRAM di Mazda Connect 74.00.324A e una
modifica sperimentale al progetto open-source TouchTune.

Il materiale è pubblicato esclusivamente per documentare l'analisi tecnica e
il relativo fix. Non costituisce documentazione ufficiale Mazda e non è
destinato a essere utilizzato come guida per modificare veicoli o sistemi
infotainment in uso.

La modifica delle impostazioni della CMU può alterare limitazioni previste dal
costruttore e comportare rischi, inclusa la distrazione durante la guida.

Per le istruzioni di utilizzo e la documentazione del progetto originale si
rimanda a TouchTune by Miatafy.


## Il problema

TouchTune gestisce due impostazioni NVRAM relative alle limitazioni
dell'interfaccia durante il movimento del veicolo:

    bus_bcm_speed_restriction
    lvds_speed_restriction

Sulla CMU utilizzata durante l'attività sperimentale queste impostazioni
non erano inizialmente presenti.

La versione originale di TouchTune interpretava questa condizione come
un errore e interrompeva l'operazione prima di applicare la modifica.

L'analisi tecnica ha mostrato che l'assenza iniziale delle relative chiavi
NVRAM non rappresenta necessariamente una condizione anomala e che il
software della CMU è in grado di gestire la loro configurazione anche
quando non sono ancora presenti.


## Analisi tecnica

Per comprendere il comportamento osservato è stata effettuata un'analisi
tecnica del firmware Mazda Connect EU 74.00.324A.

L'analisi è stata utilizzata esclusivamente per comprendere il funzionamento
delle impostazioni NVRAM interessate e verificare il comportamento previsto
dal sistema.

Il firmware Mazda, le immagini del filesystem e gli altri file proprietari
esaminati durante l'attività di ricerca non sono inclusi né distribuiti da
questo repository.

L'analisi ha permesso di stabilire che il problema non era causato
dall'impossibilità della CMU di configurare le impostazioni, ma dal controllo
preliminare effettuato da TouchTune quando le relative chiavi NVRAM non
risultavano ancora leggibili.


## Il fix

Il fix modifica esclusivamente la gestione di questa particolare condizione
all'interno di TouchTune.

Quando una delle due chiavi NVRAM non è presente, il suo stato iniziale viene
considerato equivalente allo stato:

    enable

La stessa logica viene applicata durante il controllo di preflight precedente
alla modifica.

In questo modo l'assenza iniziale della chiave non viene interpretata
automaticamente come un errore e il sistema può procedere con il normale
meccanismo di configurazione.

Le altre verifiche previste da TouchTune rimangono invariate.


## Verifica di Common.js

Durante l'attività di analisi è stato verificato anche il file `Common.js`
corrispondente alla versione firmware studiata.

SHA-256 del file originale verificato:

    376b30a46366a543122956d7feb1b44f147425015837a4d83fedf12a67943351

Il valore coincide con `MZD_PROFILE_STOCK_SHA256` previsto dal progetto
TouchTune.

Applicando la trasformazione prevista da TouchTune si ottiene:

    019eba18e8d629ddb1d55563aab138ce1eb3329fab67b71d9b4178377cf9d9ce

Questo valore coincide con `MZD_PROFILE_PATCHED_SHA256` previsto da TouchTune.

Gli hash vengono riportati esclusivamente per identificare e documentare
la versione del file utilizzata durante la verifica. Il file originale
`Common.js` non è incluso né distribuito da questo repository.


## Analisi tecnica completa

Una descrizione più dettagliata dell'attività di analisi e del problema
individuato è disponibile in:

**[Analisi del fix per Mazda Connect 74.00.324A](docs/FIX_74.00.324A_IT.md)**

Il documento descrive:

- il comportamento osservato sulla CMU utilizzata per il test;
- l'analisi delle impostazioni NVRAM interessate;
- la verifica di `Common.js` mediante SHA-256;
- la causa dell'interruzione della versione originale di TouchTune;
- la modifica apportata alla gestione delle chiavi NVRAM mancanti;
- il risultato dell'attività sperimentale.

Il documento non contiene né distribuisce firmware Mazda, immagini della
RootFS o altri file proprietari del sistema.


## Compatibilità

TouchTune originale è progettato per la famiglia firmware Gen 6:

    74.00.324
    74.00.324A

Il fix documentato in questa repository è stato verificato direttamente con:

    Mazda Connect EU 74.00.324A

Non è possibile garantire che tutte le varianti hardware, regionali o firmware
si comportino nello stesso modo.

L'installer mantiene inoltre i controlli sull'hash di `Common.js` e rifiuta file
non riconosciuti.


## Differenze rispetto a TouchTune 1.2.0

La modifica principale è contenuta in:

    usb/patches/touch-while-driving.sh

Rispetto a TouchTune 1.2.0:

1. una speed-restriction NVRAM assente viene interpretata come stato iniziale
   `enable`;
2. il controllo di preflight applica la stessa interpretazione;
3. gli script originali Mazda possono quindi creare la chiave durante
   l'impostazione a `disable`.

Il resto del meccanismo TouchTune, inclusi backup, verifica di `Common.js`,
controllo delle scritture e rollback, rimane invariato.


## Sviluppo

I file principali dell'installer sono:

    usb/install-patches.sh
    usb/patches/touch-while-driving.sh
    usb/lib/touchtune-helpers.sh

Per i test host:

    ./tests/run.sh

Ulteriori informazioni sul funzionamento originale di TouchTune sono disponibili
in:

    SAFETY.md
    CHANGELOG.md
    DISCLAIMER.md


## Sicurezza

La modifica disabilita una limitazione prevista dal costruttore.

L'utilizzo del touchscreen durante la guida può distrarre il conducente.
Utilizzare il sistema in modo responsabile e nel rispetto delle norme applicabili.

La modifica del software della CMU comporta inoltre un rischio: verificare
sempre firmware, file e backup prima di procedere.


## Crediti

### TouchTune

Progetto originale:

**Miatafy — TouchTune**

https://github.com/Miatafy/TouchTune

TouchTune deriva inoltre dal lavoro della comunità Mazda Connect / MZD-AIO.


### Fix NVRAM 74.00.324A

Questo fix nasce dall'analisi del firmware Mazda Connect EU 74.00.324A e dal
debugging del comportamento delle chiavi:

    bus_bcm_speed_restriction
    lvds_speed_restriction

L'analisi ha permesso di verificare il comportamento degli script originali
Mazda e di adattare TouchTune al caso in cui tali chiavi non siano ancora
presenti.


## Licenza

GPL-3.0-or-later.

Consultare:

- [LICENSE](LICENSE)
- [NOTICE](NOTICE)
- [DISCLAIMER.md](DISCLAIMER.md)

Il software viene fornito senza garanzia.
