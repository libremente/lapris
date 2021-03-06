# lapris
_Progetto obiettivo 2018 LAmpinet e PRISma - Fase 4_

L'applicazione consente di aggregare le stime di precipitazione prodotte dal processo prisma (interpolazione ottimale di radar e pluviometri su grigliato UTM 1kmx1km) su aree a scelta dell'utente utilizzando media e percentili standard (min,10,25,50,75,90,max); in output fornisce 4 immagini: campo totale cumulato in overlay con le areee scelte, media e massimo su aree scelte, tabella con valori dei  percentili sulle aree scelte. 

L'applicazione utilizza come visualizzazione il codice Hanis di Tom Whittaker: la configurazione e il funzionamento di dettaglio è descritto [qui](https://www.ssec.wisc.edu/hanis/index.html#framelabel)

# Uso in locale
- scaricare repository
- salvare nella cartella locale dati/ i dati prodotti da prisma del periodo di interesse; i dati si trovano sul server mediano, /home/meteo/dati/prisma/ascii, senza modificare il nome dei file (contiene le infomazioni sulla data e ora dei dati contenuti)
- nella cartella info sono contenuti già alcuni shapefile con i poligoni su cui aggregare le stime di precipitazione; si possono inserire altri shapefile nella stessa cartella

## struttura directory da replicare in locale
prisma_cumula.R
_file di appoggio_

/dati (file txt con dati di input)

/info (shapefiles)

## uso

Lancio da CMD di Windows con questo comando 
```
Rscript prisma_cumula.R yyyymmgghh n_ore basename_shp label
```
dove:
- yyyymmgghh n_ore = Orario di partenza da cui cumulare per n-ore(Es. 2018052718 12 = Cumulo a partire dalle h 18.00 UTC e fino alle 06.00 UTC del giorno successivo)  
- basename_shp = Nome shapefile senza estensione (Es. Allerta, Province, etc.)  
- label = Nome colonna della tabella attributi shp da usare per etichette (Es. CODICE_IM, SIGLA, BACINI_AGG, etc.)

__Esempio__
```
Rscript prisma_cumula.R 2018052718 12 Allerta CODICE_IM
```

ATTENZIONE: Il periodo di cumulazione è espresso in UTC

## output
 - Mappa con dati cumulati disaggregati su shapefile scelto
 - Mappa con media areale
 - Mappa con massimo areale
 - Tabella riassuntiva con statistiche su aree
 - Composite delle 4 precedenti immagini in un unico PNG per visualizzazione GHOST

# funzionamento come container
Il container è lanciato come servizio da UCP: automaticamente esegue il file _launch.sh_ 
1. esiste un bucket su minio diviso in dati e immagini; nella cartella dati vengono copiati i file in uscita da prisma _(to be implemented)_
2. il container copia a orari prestabiliti il file dal bucket alla directory _dati_, elabora il file e produce le immagini
3. le immagini viengono servite via Flask 
4. le immagini vengono archiviate nel bucket di minio

# uso dall'interno del container per periodo arbitrario (_recupero.sh_)
Poichè il servizio viene eseguito alle 5 del mattino ma il container è sempre attivo ma in fase _quiescente_, è possibile eseguire il programma R direttamente nel container senza interferire con il normale funzionamento del servizio. <br> Questo comporta il vantaggio di non dover installare niente sul proprio pc, Successivamente all'esecuzione si potranno scaricare le mappe o da _minio_ o direttamente dal sito.
Le istruzioni sono le seguenti:
1. andare nella pagina form (http://10.10.99.135:8891/form)
2. inserire i dati per il la produzione delle mappe di interesse
```
- AAAAMMGG    = data di inizio della cumulata<br>
- HH          = ora di inizio della cumulata<br>
- N           = numero di ore di cumulata<br>
- _shapefile_ = shapefile dell'area di cumulata<br>
- _label_     = campo dello shapefile per la cumulata <br>
```
3. inviare il form
Le immagini vengono automaticamente copiate su minio dove possono essere facilmente recuperate
