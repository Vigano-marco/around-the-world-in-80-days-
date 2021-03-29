# around-the-world-in-80-days-

Questo repository contiene il progetto 'Il giro del mondo in 80 giorni' di Luca Cardinale, Claudia Manili e Marco Viganò, svolto come progetto finale del corso Python Programming - Master in Data Science for Economics, Business and Finance 2020/2021 (Università degli Studi di Milano).

# Descrizione
A partire da un dataset contenente nomi di città (con relativo stato di appartenenza, numero di abitanti, latitudine e longitudine), il progetto si propone di calcolare il tempo di percorso partendo da Londra e arrivando a Londra, supponendo che sia possibile viaggiare fino alle 3 città più vicine, esclusivamente verso est.

# Criteri di viaggio/ conteggio
Viaggiare tra città implica diverse 'durate':

città più vicina: +2
seconda più vicina: +4
terza più vicina: +8
destinazione in un'altra nazione: +2
destinazione con più di 200000 abitanti +2

# Flusso di lavoro
Il gruppo ha lavorato su questa repository utilizzando Jupyter Notebook. I dati sono nella cartella 'data' del repository.

Libreria necessarie all'esecuzione dello script
Pandas, Numpy, Sklearn, Scipy, Math, NetworkX, Plotly

# Implementazione
Nello script della repository, ogni riga di codice è commentata dalla descrizione di quello che il codice implementa, in lingua inglese. In questo documento sono presentati i passaggi concettuali che il codice percorre per rispondere alla traccia di progetto. Il programma lavora sulla creazione di una matrice contenenti i "pesi" necessari per viaggiare tra città, già "direzionato" verso est, escludendo dunque ogni combinazione possibile nella direzione opposta.

# Data leaning and preparation
Il dataset viene importato e pulito, rimuovendo tutti i valori NaN. Ai fine del progetto, non conoscere eventuali parametri di viaggio, automaticamente esclude la possibilità di poter tenere conto dei dati incompleti: ogni città ha precisi attributi di nazione, numero di abitanti e location, che non è possibile stimare. Per facilitare il compito finale viene creata una "London" di partenza (chiamata London_st), con il valore di longitudine lievemente ad est di "London" di arrivo, e tutti gli altri attributi uguali. Per evitare che il dataset contenga nomi di città doppi, e per avere un ID facilmente identificabile ai fini del controllo dell'esecuzione dello script, si concatena "city_name" e "iso2" per avere un ID univoco.

# Getting weights - part I
Si procede creando diversi oggetti con le componenti di "peso" differenti. Si identificano le città con più di 200000 abitanti in un array numpy, utilizzando la colonna "population" Si pesano le combinazione di città con differente nazione di appartenenza usando "iso3" in un array numpy.

# Getting direction
Lavorando sulla direzione si pone il problema delle longitudini negative. Le longitudini vengono convertite da scala -180/+180 a scala 0-360, dove 0 è il meridiano di "Greenwich", centrato su Londra. Le longitudini tra città vengono confrontate fra loro: solo qualora la longitudine di una città è maggiore dell'altra si mantiene il valore, in caso contrario no. Il risultato è salvato in un array numpy. Per "alleggerire" il peso della matrice di distanze su cui poi si lavorerà, creiamo un array numpy dove vengono rimossi tutti i valori delle latitudini maggiori di 60gradi tra città, percorsi troppo lontani per essere considerati.

Con i primi pesi e la direzione trovate, vengono creati 3 dataframe "direction", "same_state", "population". Che verrano utilizzati in seguito.

# Distance computation
Per il calcolo delle distanze fra loro viene utilizzata la cosidetta "Harvesine Distance", che calcola la distanza tra punti su una sfera. "In mathematics, computer science and especially graph theory, a distance matrix is a square matrix containing the distances, taken pairwise, between the elements of a set. If there are N elements, this matrix will have size N×N. In graph-theoretic applications the elements are more often referred to as points, nodes or vertices." Utilizzando la latitudine e la longitudine del dataframe originario e la metrica di Scipy chiamata "harvesine", calcoliamo la "pairwise distance" tra città, moltiplicata per 6373 per ottenere la metrica in km. L'output è un numpy array, che verrà poi convertito in pandas dataframe. Il dataframe è un oggetto quadrato chiamato "matrice di distanze" dove ogni intersezione tra colonna e riga contiene la distanza tra le due relative città. Le distanze sono calcolate solo in caso di "direzione est" utilizzando il dataframe creato in precedenza.

# Getting weights - part II
Ora, conoscendo le distanze tra città, è possibile calcolare le tre più vicine dove è possibile andare, e dare i rispettivi pesi. Il codice lavora sulle righe della matrice distanze e ripopola la matrice prendendo per ogni riga in quest'ordine:

il minimo valore diverso da 0, diventa 2
il minimo valore diverso da 2, diventa 4
il minimo valore diverso da 4, diventa 8
tutti i valori maggiori di 8, diventano 0
Il dataframe con le distanze tra città più vicine (solo ad est, visto il calcolo delle distanze già fatto solo per città ad est fra loro), viene moltiplicato per gli altri due dataframe "same_state" e "population", per avere il dataframe con le distanze "pesate" tra città da utilizzare con NetworkX per il calcolo su grafi.

# Shortest path algorithm
Per il calcolo del percorso più corto, è necessario utilizzare un algoritmo chiamato "Dijkstra", che calcola il percorso più corto tra nodi (nodes) in un grafo. Il grafo è una particolare struttura dati contenente nodi (nodes) collegati tra loro tra linee (edges). Nel nostro caso i nodi sono le città, e le linee sono le ore di viaggio (pesi) calcolati nel dataframe di distanze. Per il calcolo del percorso più breve, abbiamo creato un "grafo" con NetworkX, a partire dal dataframe convertito in numpy array. Il grafo viene creato utilizzando solo le distanze esistenti tra nodi. Abbiamo così la struttura data che può essere utilizzata da NetworkX per calcolare il percorso più breve utilizzando il comando "networkx.single_source_dijkstra()" che prende in input il grafo, l'origine (London_st - linea 6622) e la destinazione (London - linea 31). L'output del comando sono due oggetti:

uno contenente la lunghezza del percorso (calcolata con i "pesi" degli edges)
l'altro contenente la lista dei "nodi" visitati
Soluzione
Il nostro percorso più breve tocca 66 città nel mondo (non contando Londra duplicata come città di partenza), e impiega 278 ore, quindi approssimativamente 11 giorni e mezzo per fare il giro del mondo.

# Data visualization
Al fine di dare una rappresentazione grafica del percorso ottimo che il programma ha calcolato, si è scelto di precedere a plottare sulla mappa del mondo la traiettoria ottenuta connettendo le città visitate. A tale scopo abbiamo utilizzato "Plotly", che permette, attraverso l'oggetto grafico "Scattergeo", di plottare le coordinate delle città visitate e di unirle con una linea sulla cartina del mondo. Coordinate e elenco delle città sono stati inseriti in un dataframe (ottenuto filtrando il dataframe originale di tutte le città per le sole città toccate dal percorso) che è stato usato a sua volta come input per Scattergeo. Per ogni città è presente un marker che ne indica la posizione e tutti i marker, come detto, sono collegati da una linea che rappresenta idealmente il percorso compiuto per completare il giro del mondo.

# References
'How to calculate Distance in Python and Pandas using Scipy spatial and distance functions' (2019). In Kanoki.org. https://kanoki.org/2019/12/27/how-to-calculate-distance-in-python-and-pandas-using-scipy-spatial-and-distance-functions/. Accessed 20 Dec 2020.

Al-Taie MZ, Kadry S (2017) Python for Graph and Network Analysis. Springer, Cham.

'NetworkX. Network Analysis in Python' (2020). https://networkx.org/. Accessed 20 Dec 2020.
