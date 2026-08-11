`Excel`

## 🔧 Come registrare le mie LAMBDA nel Gestore nomi

1. Vai sulla scheda **Formule** → **Gestione nomi** → **Nuovo…** oppure premi CTRL+F3
2. **Nome**: il `nome` della lambda, fino alla prima "(" esclusa
3. **Ambito**: Cartella di lavoro
4. **Commento**: la `descrizione` della lambda *(opzionale)*
5. **Riferito a**: incolla la formula LAMBDA
6. Conferma con **OK**

**nome**: ANNO.FISCALE(data;data_inizio)    
**descrizione**: Restituisce l'anno fiscale di appartenenza a partire da una data e una data di inizio dell'anno fiscale    

    =LAMBDA( data; data_inizio;    
        LET(    
            diff_anni; ANNO(data) - ANNO(data_inizio);    
            SWITCH(    
                diff_anni;    
                0; SE(data >= data_inizio; ANNO(data) + 1; ANNO(data));    
                1; SE(MESE(data) >= MESE(data_inizio); ANNO(data) + 1; ANNO(data));    
                -1; SE(MESE(data) >= MESE(data_inizio); ANNO(data_inizio); ANNO(data));    
                #VALORE!    
            )    
        )    
     )
     
 **nome**: IVA(importo;[azione];[aliquota])    
 **descrizione**: Restituisce il calcolo dell'IVA, dello scorporo, dell'importo dell'iva (azione = 2, default) e del totale lordo (azione = 3). aliquota = 1 è la regolare (22%) default, aliquota = 2 è la ridotta (10%) , aliquota = 3 è la minima (5%), aliquota = 4 è al 4%
                  
    =LAMBDA( importo; [azione]; [aliquota];    
        LET( IVA;    
            SWITCH(    
                aliquota;    
                1; 0,22;    
                2; 0,1; 
                3; 0,05;
                4; 0,04;    
                0,22    
            );    
            SWITCH(    
                azione;    
                1; importo / (1 + IVA);    
                2; importo * IVA;    
                3; importo * (1 + IVA);    
                importo * IVA    
            )    
        )    
    )
    
**nome**: IRPEF(reddito)    
**descrizione**: Calcola l'irpef sul reddito fiscale in base alle aliquote aggiornate con la nuova legge di bilancio del 2026    

    =LAMBDA( reddito;    
        LET(
            impo1; MAX(0; reddito - 50000);
            aliq1; 0,43;
            impo2; MAX(0; MIN(reddito; 50000) - 28000);
            aliq2; 0,33;
            impo3; MIN(reddito; 28000);
            aliq3; 0,23;
            aliq1 * impo1 + aliq2 * impo2 + aliq3 * impo3
        )
    )

**nome**: QUADRIMESTRE(data;[mese_inizio])    
**descrizione**: Restituisce il quadrimestre di riferimento, assumendo come primo mese del primo quadrimestre il numero passato al secondo argomento (default 1)    

    =LAMBDA( data; [mese_inizio];    
        LET(    
            mese; MESE(data);    
            offset; RESTO((mese - mese_inizio + 12); 12);    
            1 + TRONCA(offset / 3)    
        )    
    )

**nome**: SETTIMANA.FISCALE(data; [data_inizio])   
**descrizione**: Restituisce la settimana fiscale di appartenenza a partire da una data e se presente da una data di inizio dell'anno fiscale 

    =LAMBDA( data; [data_inizio];
        LET(
             t; SE(data_inizio=0;DATA(ANNO(data);1;1);data_inizio);
            fs; DATA(ANNO(data);MESE(t);GIORNO(t));
            fe; SE(data<fs;DATA(ANNO(fs)-1;MESE(t);GIORNO(t));fs);
            wd; GIORNO.SETTIMANA(fe;2);
            dm; RESTO(8-wd;7);
            fm; fe+SE(wd=1;0;dm);
            SE(wd=1;TRONCA((data-fe)/7)+1;SE(data<fm;1;TRONCA((data-fm)/7)+2))
        )
    )

**nome**: EMAIL.VALIDA(testo)

**descrizione**: Verifica se il testo passato contiene un indirizzo email formalmente valido. Usa REGEXTEST con controllo senza distinzione tra maiuscole e minuscole.

```
=LAMBDA( testo;   
           REGEX.TEST (   
             testo; "^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$"; 1   
           )   
       )
```

**nome**: ESTRAI.EMAIL(testo)

**descrizione**: Estrae dal testo il primo indirizzo email trovato. È utile per pulire note, commenti o descrizioni importate da sistemi esterni.

```
=LAMBDA( testo;   
           REGEX.ESTRAI (   
             testo; "[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}"; 0; 1   
           )   
       )
```

**nome**: SOLO.NUMERI(testo)

**descrizione**: Rimuove tutti i caratteri non numerici da una stringa, mantenendo solo le cifre da 0 a 9. Utile per normalizzare telefoni, codici o riferimenti misti.

```
=LAMBDA( testo;   
           REGEX.SOSTITUISCI ( testo; "\D"; "" )   
       )
```

**nome**: NORMALIZZA.SPAZI(testo)

**descrizione**: Sostituisce tabulazioni, ritorni a capo e spazi multipli con un singolo spazio, poi elimina gli spazi iniziali e finali.

```
=LAMBDA( testo;   
          ANNULLA.SPAZI (   
            REGEX.SOSTITUISCI ( testo; "\s+"; " " )   
          )   
       )
```

**nome**: SLUG.TESTO(testo)

**descrizione**: Converte una descrizione in uno slug testuale minuscolo, sostituendo sequenze di caratteri non alfanumerici con un trattino e rimuovendo eventuali trattini iniziali o finali.

```
=LAMBDA( testo;   
          LET ( base; MINUSC ( ANNULLA.SPAZI ( testo ));   
                senza_accenti;   
                  SOSTITUISCI (   
                     SOSTITUISCI (   
                        SOSTITUISCI (   
                           SOSTITUISCI (   
                              SOSTITUISCI ( base; "à"; "a" ); "è"; "e"); "é"; "e" ); "ì";"i" ); "ò"; "o" );   
                pulito;   
                  SOSTITUISCI ( senza_accenti; "ù"; "u" );   
                REGEX.SOSTITUISCI ( REGEX.SOSTITUISCI ( pulito; "[^a-z0-9]+"; "-" ); "(^-|-$)"; "" )   
              )   
       )
```

**nome**: INIZIALI(testo)

**descrizione**: Restituisce le iniziali maiuscole delle parole presenti nel testo. Usa DIVIDI.TESTO, TESTO.UNISCI e normali funzioni di manipolazione testo.

```
=LAMBDA( testo;   
          LET (   
                parole; DIVIDI.TESTO ( ANNULLA.SPAZI ( testo ); " " );   
                TESTO.UNISCI ( ""; VERO; MAIUSC ( SINISTRA ( parole; 1 )))   
              )   
       )
```

**nome**: CAMEL.CASE(testo)

**descrizione**: Converte una stringa composta da parole separate da spazi, trattini o underscore in notazione CamelCase.

```
=LAMBDA( testo;   
          LET (   
                pulito; REGEX.SOSTITUISCI ( ANNULLA.SPAZI ( testo ); "[-_]+"; " " );   
                parole; DIVIDI.TESTO ( pulito; " " );   
                TESTO.UNISCI ( ""; VERO; MAIUSC.INIZ ( parole ))   
              )   
       )
```

**nome**: PROGRESSIVO.TESTO(intervallo; [separatore])

**descrizione**: Restituisce una matrice dinamica con la concatenazione progressiva dei valori di un intervallo. Usa SCAN per mostrare ogni risultato intermedio.

```
=LAMBDA( intervallo; [separatore];   
          LET (   
                sep; SE ( ISOMITTED ( separatore ); ", "; separatore );   
                SCAN ( ""; intervallo; LAMBDA ( accumulo; valore; SE ( accumulo = ""; valore; accumulo & sep & valore ) ) )   
              )   
       )
```

**nome**: SOMMA.PROGRESSIVA(intervallo)

**descrizione**: Calcola una matrice dinamica con la somma progressiva dei valori dell'intervallo. È una LAMBDA base utile per saldi contabili e progressivi mensili.

```
=LAMBDA( intervallo;   
          SCAN ( 0; intervallo; LAMBDA ( accumulo; valore; accumulo + valore ))  
       )
```

**nome**: CONTA.PROGRESSIVA.TESTO(intervallo)

**descrizione**: Conta progressivamente le celle non vuote di un intervallo testuale, restituendo una matrice dinamica con il contatore aggiornato riga per riga.

```
=LAMBDA( intervallo;   
          SCAN ( 0; intervallo; LAMBDA ( accumulo; valore; SE ( valore = ""; accumulo; accumulo + 1 )))   
       )
```

**nome**: PERCORSO.ULTIMO.NODO(percorso; [separatore])

**descrizione**: Estrae l'ultimo segmento di un percorso testuale, ad esempio il nome file da un percorso con barre, oppure l'ultima voce di una gerarchia separata da un carattere scelto.

```
=LAMBDA( percorso; [separatore];   
          LET (   
                sep; SE ( ISOMITTED ( separatore ); "/"; separatore );   
                REGEX.SOSTITUISCI ( percorso; "[^" & sep & "]+$" )   
              )   
       )
```

## 🔧 Come riutilizzare la funzione IVA per creare una nuova LAMBDA    
La funzione che segue utilizza una delle formule date (IVA) per generare una nuova che calcola il totale di una    
parcella di un professionista al netto della ritenuta d'acconto del 20%, con la possibilità di scegliere l'aliquota    
della cpa (di default 4%).    

**nome**: IVARACPA(imponibile;[sceltacpa])    
**descrizione**: Calcola il totale ivato al lordo della cpa e al netto della ritenuta d'acconto. Se si usa la cpa al 2%,    
il secondo parametro va valorizzato a 2, ad es.: *=IVARACPA(100000;2)*

    =LAMBDA( imponibile; [sceltacpa];
         LET(
            aliqcpa; SWITCH(sceltacpa; 2; 0,02; 0,04);
            cpa; imponibile * aliqcpa;
            somma_iva; IVA(imponibile + cpa; 2); // calcola l'IVA al 22% sulla somma dell'imponibile e cpa
            ritenuta; -imponibile * 0,2;        // ritenuta d'acconto al 20% già in negativo
            imponibile + ritenuta + somma_iva   // qui la ritenuta è sommata perché sopra è già in negativo
         )
     )

