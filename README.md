# masterproject_hotel
Final project for programming with python module by Bichi Tommaso, Ciobica Oana Mari, Gessner Voltolina Lorenzo

INTRODUZIONE e APPROCCIO GENERALE

Questo programma consiste nell'implementazione di quattro diverse strategie per l'allocazione dei clienti presso gli hotel presenti nei seguenti file:

hotels.xlsx: per 400 hotel è disponibile l'informazione circa il numero di camere libere e il costo unitario di ogni camera

guests.xlsx: per 4000 clienti potenziali è disponibile l'informazione sulla frazione del costo unitario della camera a cui hanno diritto come sconto

preferences.xlsx: per ogni cliente è disponbile l'informazione sull'ordine di preferenza dell'hotel, dove il valore numerico associato alla preferenza indica l'ordine di preferenza (i.e., 1 è la prima scelta, 2 la seconda, etc.).

Per ogni strategia è stata creata una funzione dedicata, atta a rispettare le condizioni indicate di seguito:

random_model. Casuale: i clienti sono distribuiti casualmente nelle stanze fino a esaurimento dei posti o dei clienti

preference_model. Preferenza cliente: i clienti sono distribuiti nelle stanze in ordine di prenotazione (il numero del cleinte ne indica l'ordine) e attribuendogli l'hotel in ordine di preferenza, fino a esaurimento dei posti o dei clienti

price_model. Prezzo: i posti in hotel sono distribuiti in ordine di prezzo, partendo dall'hotel più economico e in subordine in ordine di prenotazione e preferenza fino a esautimento posti o clienti 

room_model. Disponibilità: i posti in hotel sono distribuiti in ordine di disponibilità di stanze, partendo dall'hotel più capiente e in subordine in ordine di prenotazione e preferenza fino a esautimento posti o clienti 


Per tutte e quattro le strategie si è scelto di partire dalla matrice 'rank_matrix' (4000X400) che prende come assi i clienti (guest) e gli hotel e come valori le preferenze (priority) dei clienti per ogni hotel. Alla fine della funzione ognuna delle quattro strategie dovrà ritornare una 'choice_matrix' delle stesse dimensioni riempita da 0 e 1, dove 1 rappresenta una camera occupata da un cliente  e 0 altrimenti. Le  'choice_matrix' verranno infine fatte passare attraverso la funzione 'results' in modo da ottenere per ogni strategia il numero di clienti sistemati, il numero di stanze occupate, il numero di hotel diversi occupati, il volume complessivo di affari e il grado di soddisfazione dei clienti (calcolato con l'ausilio di una 'utility_matrix' creata ad hoc).



SVOLGIMENTO

Dopo l'import dei dati:

vec_discount: vettore di 4000 valori che corrispondono alla percentuale pagata da ogni cliente sul prezzo delle camere. 

vec_prices: vettore di 400 valori che rappresentano il prezzo della camera per hotel.

rank_matrix: dai dati in preferences.xlsx, viene creato un DataFrame 4000X400 (pref_pivoted). Da notare che alcuni guest hanno due preferenze sullo stesso hotel. Con aggfunc="min" si è scelto di selezionare il valore minore. Viene infine creata la rank_matrix con il rank dei valori delle sequenze di preferenze. In questo modo si vengono a colmare eventuali "salti" nell'ordinamento delle preferenze (es. se pref=1,2,5,6 allora rank(pref)=1,2,3,4).

to_utility_matrix: funzione

results: la funzione prende 4 argomenti: choice_matrix, vec_prices(vettore con prezzo degli alberghi), vec_discount(vettore con la percentuale pagata dal cliente tolto lo sconto spettante), utility_matrix. La funzione ritorna: guest_placed (la somma del risultato della somma per riga della matrice), rooms_occupied (la somma del risultato della somma per colonna della matrice), hotels_occupied (la somma delle colonne la cui somma dei valori è maggiore di 1), revenue (il volume com plessivo d'affari, calcolato moltiplicando choice_matrix per vec_prices e vec_discount e poi sommando i valori), utility (il grado di soddisfazione complessivo, ricavato dalla somma dei valori ottenuti moltiplicando choice_matrix per utility_matrix).

random_model: funzione che prende ???? argomenti: hotels (DF di hotels.xlsx),rank_matrix e t (numero di iterazioni che si vuole ottenere). Viene creato un vettore (vec_rooms = np.repeat(hotels["price"], hotels["rooms"]).index.values) che corrisponde all'indice di ogni hotel ripetuto per il numero delle sue camere (len(vec_rooms)=4617). Trattandosi di una selezione casuale, i risultati possono variare sensibilmente tra un campionamento e l'altro. Sarà quindi necessario ripetere l'operazione t=n volte, calcolando la media dei risultati delle n iterazioni, per ottenere un risultato più stabile. 
Per ogni t: viene creata una choice_matrix (dimensione=rank_matrix,valori=0). Da vec_rooms vengono selezionati casualmente un numero di valori con replace=False pari al numero di clienti (se il numero delle camere fosse minore del numero dei clienti, verrebbe selezionato un numero di clienti pari al numero di camere, least = min([guest_count, room_count]). Ogni cliente (range(least)) viene associato a una camera (random_rooms) e registrato con 1 nella choice_matrix. La choice_matrix viene passata attraverso la funzione results. I risultati vengono aggiunti alla lista res.
La funzione ritorna np.array(res).

preference_model:la funzione prende 2 argomenti: hotels (DF di hotels.xlsx) e rank_matrix. In rank_matrix gli np.nan son o sostituiti da 0 (rank_matrix_filled). Viene anche inizializzata choice_matrix(dimensione=rank_matrix,valori=0). Ogni riga (corrispondente a un guest) di rank_matrix_filled viene moltiplicata per il vettore rooms (rooms = hotels["rooms"].values, numero di camere per hotel) dove rooms è uguale a 1 se sono presenti camere disponibili, 0 altrimenti. In questo modo le preferenze del guest vengono azzerate per quegli hotel che hanno esaurito le camere. La scelta del cliente (user_choice) viene selezionata prendendo il valore minore diverso da zero e convertendolo in 1 per poi inserirlo nella choice_matrix. Al vettore rooms viene sottratta la user_choice. Infine choice_matrix viene passata attraverso la funzione results.

room_model: funzione che prende due argomenti: hotels (DF di hotels.xlsx) e rank_matrix. A rank_matrix.T vengono aggiunte le colonne hotels['price','rooms']. Viene quindi creato un DF dove rank_matrix.T è ordinato prima in base al numero di rooms (dall'hotel più capiente a quello con meno camere) e price come seconda condizione. Per ogni riga del nuovo DF viene creato un DF composto dai valori di riga (values) e dal loro indice(index). In questo modo ogni riga (corrispondente a un hotel) viene ordinata usando la preferenza del cliente(valore di riga) e come secondo criterio l'ordine di prenotazione (index). Infine viene selezionato un numero di clienti fino a un massimo uguale al numero di camere disponibili dell'hotel considerato(rooms[h]). Il risultato viene inserito nella choice_matrix e le colonne corrispondenti ai guest selezionati vengono trasformate in np.nan, in modo da evitare che vengano inseriti nei successivi hotel.

price_model: la procedura è del tutto identica a room_model tranne che per il DF iniziale che viene ordinato prima per ordine di prezzo (dal più economico al più caro) e poi per numero di camere.

CONCLUSIONE e GRAFICI



