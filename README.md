# Pt.2 Sogni d'Oro
Final project for programming with Python by Bichi Tommaso, Ciobica Oana Maria, Gessner Voltolina Lorenzo

### INTRODUZIONE e APPROCCIO GENERALE

Questo programma consiste nell'implementazione di quattro diverse strategie per l'allocazione dei clienti presso gli hotel presenti nei seguenti file:

*hotels.xlsx*: per 400 hotel è disponibile l'informazione circa il numero di camere libere e il costo unitario di ogni camera

*guests.xlsx*: per 4000 clienti potenziali è disponibile l'informazione sulla frazione del costo unitario della camera a cui hanno diritto come sconto

*preferences.xlsx*: per ogni cliente è disponbile l'informazione sull'ordine di preferenza dell'hotel, dove il valore numerico associato alla preferenza indica l'ordine di preferenza (i.e., 1 è la prima scelta, 2 la seconda, etc.).

Per ogni strategia è stata creata una funzione dedicata, atta a rispettare le condizioni indicate di seguito:

**random_model**. Casuale: i clienti sono distribuiti casualmente nelle stanze fino a esaurimento dei posti o dei clienti

**preference_model**. Preferenza cliente: i clienti sono distribuiti nelle stanze in ordine di prenotazione (il numero del cleinte ne indica l'ordine) e attribuendogli l'hotel in ordine di preferenza, fino a esaurimento dei posti o dei clienti

**price_model**. Prezzo: i posti in hotel sono distribuiti in ordine di prezzo, partendo dall'hotel più economico e in subordine in ordine di prenotazione e preferenza fino a esautimento posti o clienti 

**room_model**. Disponibilità: i posti in hotel sono distribuiti in ordine di disponibilità di stanze, partendo dall'hotel più capiente e in subordine in ordine di prenotazione e preferenza fino a esautimento posti o clienti 


Per tutte e quattro le strategie si è scelto di partire dalla matrice `rank_matrix` (4000X400) che prende come righe i clienti (guest), come colonne gli hotel e come valori le preferenze (priority) dei clienti per ogni hotel. Alla fine della funzione ognuna delle quattro strategie dovrà restituire una `choice_matrix` di dimensioni uguali alla `rank_matrix`, ma con valori binari {0,1}, dove 1 rappresenta una camera occupata da un cliente. La `choice_matrix` verrà infine fatta passare attraverso la funzione *results* in modo da ottenere per ogni strategia:

  * il numero di clienti sistemati
  * il numero di stanze occupate
  * il numero di hotel diversi occupati
  * il volume complessivo di affari e il grado di soddisfazione dei clienti

### SVOLGIMENTO

Dopo l'import dei dati:

`hotels`: DataFrame con due colonne con il prezzo per camera `hotels["price"]` e il numero di camere `hotels["rooms"]`

`vec_discount`: vettore di 4000 valori che corrispondono alla percentuale pagata da ogni cliente sul prezzo delle camere. 

`vec_prices`: vettore di 400 valori che rappresentano il prezzo della camera per hotel.

`rank_matrix`: dai dati in *preferences.xlsx*, viene creato una matrice dove su ogni riga troviamo le preferenze di ogni guest e su ogni colonna l'hotel di riferimento. Da notare che alcuni guest hanno due preferenze sullo stesso hotel. Con `aggfunc="min"` si è scelto di selezionare il valore minore.  Con la funzione  `df.rank(axis = 1)` si vengono a colmare eventuali "salti" nell'ordinamento delle preferenze per riga (es. se `pref=<1,2,5,6>` allora `rank(pref)=<1,2,3,4>`).

*to_utility_matrix*(`rank_matrix`):
 Questa funzione trasforma le preferenze della `rank_matrix` in un valore-utilità per ogni guest. Mantenendo le dimensioni della `rank_matrix`, su ogni riga attribuiamo un valore-utilità 1 alla prima scelta e 0.1 all'ultima scelta. Le preferenze intermedie vengono distribuite con una distanza uniforme tra 1 e 0.1. Gli hotel senza preferenza per un determinato guest avranno un valore-utilità di 0.

*results*(`choice_matrix`, `vec_prices`, `vec_discount`, `utility_matrix`):
 La funzione ritorna una lista con 5 valori:
 
  *guest_placed (la somma del risultato della somma per riga della `choice_matrix`)
  *rooms_occupied (la somma del risultato della somma per colonna della `choice_matrix`)
  *hotels_occupied (la somma delle colonne della `choice_matrix` la cui somma dei valori è maggiore di 1)
  *revenue (il volume com plessivo d'affari, calcolato moltiplicando choice_matrix per vec_prices e vec_discount e poi sommando i valori)
  *utility (il grado di soddisfazione complessivo, ricavato dalla somma dei valori ottenuti moltiplicando choice_matrix per utility_matrix)

*random_model*(`hotels`, `rank_matrix`, `t`):
 Dal DataFrame `hotels` viene creato un vettore `vec_rooms` che corrisponde all'indice di ogni hotel ripetuto per il numero delle sue camere.
Trattandosi di una funzione random, i risultati possono variare sensibilmente tra un campionamento e l'altro. Sarà quindi necessario ripetere l'operazione `t` volte,  e calcolare la media dei risultati di tutte le iterazioni. In ogni iterazione viene creata una nuova `choice_matrix` estraendo un random sample `np.random.choice()` da `vec_rooms` di dimensione pari minimo tra il numero di clienti e il numero di camere. Ad ogni guest verrà associata una camera dove l'indice del guest è uguale all'indice del `vec_rooms` e l'indice dell'hotel è pari al valore di `vec_rooms`. Per ogni iterazione la `choice_matrix` passerà attraverso la funzione *results* ed i rusultati sono appesi. La funzione restituisce una matrice `t`x5, dove `t` è il numero di iterazioni.

*preference_model*(`hotels`, `rank_matrix`):
 In questa funzione viene inizializzata la `choice_matrix` con `np.zeros(rank_matrix.shape)` ed un vettore con il numero di camere `rooms`. Ogni vettore riga (corrispondente alle preferenze di un guest) della `rank_matrix` viene moltiplicato per un vettore con valori binari {0,1} dove `rooms` è uguale a 1 dove è presente almeno una camera disponibile. In questo modo le preferenze del guest vengono azzerate per quegli hotel che hanno esaurito le camere. La scelta del cliente viene selezionata prendendo l'indice dove è presente il valore minore diverso da zero. La scelta di ogni guest è registrata nella `choice_matrix` che infine viene passata attraverso la funzione *results* .

*room_model*(`hotels`, `rank_matrix`):
 Il DataFrame creato dalla trasposta di `rank_matrix` (`rank_matrix.T`) è ordinato prima in base al numero di `rooms` (dall'hotel più capiente a quello con meno camere) ed in subordine dal `price` prezzo decrescente. Ogni riga di questo DataFrame (corrispondente a tutte le preferenze per un hotel) viene ordinato usando la preferenza del cliente (valore di riga) ed in subordine l'indice del valore (corrispondente all'ordine di prenotazione dei guest). Dalla riga vengono selezionati un numero di valori fino ad un massimo uguale al numero di camere disponibili per l'hotel considerato. I risultati vengono inseriti nella `choice_matrix` e le colonne corrispondenti ai guest selezionati vengono riempite da `np.nan` per essere esclusi dai successivi hotel. La `choice_matrix` che infine viene passata attraverso la funzione *results*.

*price_model*(`hotels`, `rank_matrix`):
 La procedura è del tutto identica a *room_model* tranne che il dataframe iniziale che viene ordinato prima per ordine di prezzo (dal più economico al più caro) e poi per numero di camere.




