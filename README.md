# Dokumentacija za Aplikaciju za Vremensku Prognozu

## Opis rada aplikacije
Aplikacija za vremensku prognozu omogućava korisnicima da pregledaju trenutne vremenske uvjete i prognoze za različite gradove. Korisnici mogu pretraživati vremenske informacije za odabrane gradove, spremati omiljene lokacije, te pregledati detalje vremenskih uvjeta uključujući temperaturu, vlažnost, brzinu vjetra, UV indeks, i još mnogo toga.

Pri otvaranju aplikacije korisniku će biti prikazan početni ekran. Na početnom ekranu se nalaze podaci o vremenskoj prognozi (default grad je Sarajevo). U gornjem desnom uglu se nalazi ikona za dijeljenje. Korisnik može podijeliti informacije o vremenu trenutne lokacije.

![home](https://github.com/esmaagic/WeatherApp/assets/108765837/5516b697-9976-428d-9d0b-0944d65f4b57)


Navigacija sadrži dva dugmeta:
- **Krajnji desni dugmic** vodi korisnika na stranicu na kojoj može pretraživati druge lokacije. Ako korisnik unese ispravnu ili približno ispravnu lokaciju, biće preusmjeren na početni zaslon gdje će biti prikazani podaci za traženu lokaciju, a ista će se sačuvati u bazi podataka kojoj može pristupiti klikom na desno dugme navigacije. Ako korisnik unese neispravnu lokaciju, dobit će adekvatnu poruku.


![search](https://github.com/esmaagic/WeatherApp/assets/108765837/77b8bdbb-04e5-4fc8-93cb-2f72647d4d40)


- **Krajnji lijevi dugmic** će preusmjeriti korisnika na njegove sačuvane lokacije. Korisnik ima mogućnost da briše pohranjene lokacije. Klikom na neku od lokacija biće preusmjeren na početni zaslon sa informacijama o izabranoj lokaciji.


![locations](https://github.com/esmaagic/WeatherApp/assets/108765837/a4029895-3924-4dad-bf58-a519d379ffae)


## Opis arhitekture aplikacije
Aplikacija je razvijena koristeći Kotlin programski jezik i Jetpack Compose za izradu korisničkog sučelja. Arhitektura aplikacije slijedi Model-View-ViewModel (MVVM) arhitekturni obrazac. Glavni dijelovi arhitekture uključuju:

- **Model**: Sadrži podatke i poslovnu logiku aplikacije (WeatherModel, Locations).
- **View**: Prikazuje podatke korisniku i šalje korisničke akcije ViewModelu (WeatherApp, SearchScreen, MyLocationsPage, WeatherPage).
- **ViewModel**: Posreduje između Modela i Viewa, upravlja podacima koji se prikazuju i obrađuje korisničke interakcije (LocationsViewModel, WeatherViewModel).
- **Repository**: Izolira podatkovni sloj od ostatka aplikacije (LocationsRepository).

## Opis funkcionalnosti pojedinačnih klasa i funkcija

### Api package
- **WeatherModel**: Ova data klasa predstavlja model vremenskih podataka koji će biti dobiveni iz API-ja.
- **RetrofitInstance**: Ova singleton klasa služi za kreiranje Retrofit instance koja se koristi za mrežnu komunikaciju s vremenskim API-jem.
  - `baseUrl`: Osnovna URL adresa API-ja.
  - `getInstance()`: Privatna funkcija koja kreira i konfigurira Retrofit instance.
  - `weatherApi`: Javna varijabla koja koristi kreiranu Retrofit instance za implementaciju sučelja WeatherApi.
- **WeatherApi**: Ovaj interfejs definira funkciju za dobijanje vremenskih podataka iz API-ja koristeći HTTP GET zahtjev.
- **NetworkResponse**: Zapečaćena klasa koja predstavlja različite statuse mrežnog zahtjeva:
  - `Success<T>`: Predstavlja uspješan odgovor koji sadrži podatke tipa T.
  - `Error`: Predstavlja grešku s porukom.
  - `Loading`: Predstavlja stanje kada je zahtjev u toku.
- **Constant**: Ova singleton klasa sadrži konstantu `apiKey` koja predstavlja API ključ potreban za autentifikaciju prilikom slanja zahtjeva API-ju.

### Data package
- **Location**: Ova data klasa predstavlja entitet u bazi podataka.
- **LocationDao**: Ovaj interfejs definira Data Access Object (DAO) za entitet Location.
  - `@Insert`: Metoda za umetanje nove lokacije u bazu. U slučaju sukoba (postojeća lokacija s istim imenom), novi unos se ignorira.
  - `@Query`: Metoda za dohvaćanje svih lokacija iz baze, vraća `Flow` liste lokacija.
  - `@Delete`: Metoda za brisanje određene lokacije iz baze.
- **LocationsDatabase**: Ova apstraktna klasa predstavlja Room bazu podataka.
  - `@Database`: Oznaka koja definira entitete koje baza sadrži i verziju baze.
  - `locationDao()`: Apstraktna funkcija koja vraća DAO za lokacije.
  - `companion object`: Singleton objekt koji osigurava jedinstvenu instancu baze podataka. `getDatabase(context)` metoda vraća instancu baze podataka.
- **LocationsRepository**: Ovaj interfejs definira repozitorij za upravljanje podacima lokacija.
  - `getAllLocationsStream()`: Funkcija koja vraća `Flow` liste lokacija.
  - `insertLocation(location: Location)`: Suspendirana funkcija za umetanje lokacije.
  - `deleteLocation(location: Location)`: Suspendirana funkcija za brisanje lokacije.
- **OfflineLocationsRepository**: Ova klasa implementira `LocationsRepository` interfejs koristeći DAO za pristup podacima.

### Ui package
- **WeatherScreen(enum)**: Predstavlja moguće ekrane u aplikaciji, uključujući Početni zaslon (Home), Zaslon sačuvanih lokacija (Locations) i Pretraga (Search).
- **WeatherAppBar (Composable funkcija)**: Komponenta koja prikazuje gornju traku aplikacije. Omogućuje korisniku kretanje kroz aplikaciju.
- **SearchScreen (Composable funkcija)**: Pretstavlja ekran na kojem korisnik može pretražiti lokacije za dobavljanje podataka o vremenskoj prognozi.
- **LocationsPage (Composable funkcija)**: Pretstavlja ekran na kojem korisnik može pregledati svoje sačuvane lokacije.
- **WeatherApp (Composable funkcija)**: Glavna komponenta aplikacije koja upravlja navigacijom između ekrana. Koristi `NavHost` za prikazivanje različitih ekrana ovisno o stanju navigacije.
- **LocationsViewModel**: ViewModel koji komunicira sa bazom, gdje dobavlja, kreira i briše podatke iz baze.
- **WeatherViewModel**: Upravlja podacima vremenske prognoze.
  - `weatherApi`: Instanca Retrofit-a za pozive API-ju.
- **LocationsViewModelFactory**: LocationsViewModelFactory je klasa koja služi za kreiranje instance LocationsViewModel. ViewModel-e obično kreiramo pomoću ViewModelProvider, koji koristi factory za stvaranje ViewModel-a.
  - **Funkcija i svrha LocationsViewModelFactory**:
    - **Kreiranje ViewModel-a s parametrima**: `LocationsViewModel` ima konstruktor koji prima `LocationsRepository` kao parametar. `ViewModelProvider` ne može direktno kreirati takve ViewModel-e jer očekuje ViewModel-e s praznim konstruktorom. `LocationsViewModelFactory` omogućuje kreiranje `LocationsViewModel` instance s potrebnim parametrima.
    - **Omogućava Dependency Injection**: Korištenjem factory-a možemo ubrizgati zavisnosti kao što su repozitoriji, API servisi ili bilo koje druge komponente potrebne za ViewModel.
- **shareWeather**
  - **Parametri funkcije**:
    - `context`: Objekt Context koji se koristi za pokretanje aktivnosti.
    - `location`: Trenutna lokacija u view model-u.
    - `temp`: Trenutno vrijeme za datu lokaciju.
  - **Priprema podataka za dijeljenje**:
    - Definira se varijabla `subject` koja sadrži naslov poruke koja će se dijeliti.
    - Stvara se `summary` koristeći funkciju `buildString`. U `summary` se dodaju dodatne informacije. Ove informacije se formatiraju i dodaju na kraj `summary` stringa.
  - **Kreiranje Intenta za dijeljenje**:
    - Stvara se objekt `Intent` s akcijom `ACTION_SEND`, što označava da korisnik želi dijeliti podatke.
    - Tip podataka koji će se dijeliti postavljen je na "text/plain" pomoću metode `type`.
    - U `Intent` se dodaju podaci koji će biti dijeljeni:
      - `EXTRA_SUBJECT`: Naslov poruke.
      - `EXTRA_TEXT`: Sadržaj poruke, odnosno `summary`.
  - **Pokretanje aktivnosti za dijeljenje**:
    - Kako bi se omogućilo korisniku odabir aplikacije za dijeljenje, koristi se `Intent.createChooser` metoda koja stvara dijaloški okvir s popisom aplikacija koje podržavaju dijeljenje teksta.
    - Nakon odabira aplikacije, `context.startActivity` pokreće aktivnost za dijeljenje s pripremljenim podacima.
  - Ova funkcija omogućuje korisnicima da podijele informacije o vremenskoj prognozi s drugima putem različitih aplikacija kao što su poruke, društvene mreže ili e-pošta. Na ovaj način, funkcija `shareWeather` interagira sa životnim ciklusom aktivnosti tako što koristi Context dobavljen iz aktivnosti kako bi pokrenula aktivnost za dijeljenje, prateći tok životnog ciklusa aplikacije i osiguravajući ispravno funkcioniranje.

## Opis opštih koncepata Android frameworka

### Aktivnost (Activity)
Predstavlja jedan zaslon korisničkog sučelja s kojim korisnici mogu interaktivno komunicirati.

### Životni ciklus (Lifecycle)
Definira različite faze u životnom vijeku aktivnosti, od stvaranja do uništenja, omogućujući programerima da upravljaju resursima i ponašanjem aplikacije.

### Intent
Objekt koji predstavlja namjeru za obavljanje određene akcije, kao što je pokretanje nove aktivnosti ili slanje poruke sustavu.

### Rasporedi (Layouts)
Definiraju strukturu i raspored elemenata korisničkog sučelja u aktivnosti ili fragmentu.

### Resursi (Resources)
Statički podaci kao što su slike, nizovi, boje, nacrti i stringovi koji se koriste u aplikaciji.

### Room
Room je ORM (Object Relational Mapping) biblioteka koja omogućava rad s SQLite bazom podataka na Androidu.
- Olakšava rad s bazom podataka pružajući apstrakciju nad direktnim SQLite upitima.
- Omogućava provjeru SQL upita u vrijeme kompajliranja, što smanjuje mogućnost grešaka.
- Podržava reaktivno programiranje korištenjem LiveData i Flow za obavještavanje o promjenama podataka.

### Komponente
- **Entity**: Predstavlja tabelu u bazi podataka.
- **DAO (Data Access Object)**: Interfejs koji definira metode za pristup bazi podataka.
- **Database**: Apstraktna klasa koja nasljeđuje RoomDatabase i služi kao glavni pristupnik za povezivanje s bazom podataka.
