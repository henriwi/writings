# Go som backend til din neste webapplikasjon

Go er et programmeringsspråk som er utviklet av Google og har fokus på enkelhet, stabilitet og ytelse. Videre er Go et språk som prøver å bygge en bro mellom effektive, statisk typede språk og produktive, dynamiske språk. Go kommer med kraftige verktøy og et rikt standardbibliotek, og sammen med Go sin modell for parallellitet gjør dette at man enkelt skal kunne skrive applikasjoner med høy ytelse. I denne blogposten vil jeg deg en introduksjon til Go, samt vise hvordan man kan skrive en HTTP-backend i Go og deploye denne til Heroku. Forhåpentligvis vil dette inspirere deg til å teste ut Go selv, og i tillegg er målet å vise deg hvor gøy det er å jobbe med Go!

# Introduksjon
Go er utviklet av Google og det er XX XX, YY YY og ZZ ZZ som står bak. Den første offisielle versjonen ble sluppet i 2009. Språket kompilerer til maskinkode, og resultatet er en binary som man så kan kjøre på den plattformen man har kompilert programmet for. Go har statisk linking av biblioteker, det vil si at alle bibliotekene programmet ditt bruker blir pakket sammen med den eksekverbare versjonen av programmet. Dette fører til at programmet blir noe større, men fordelen er at man ikke har noen dynamiske avhengigheter som må være på plass på maskinen hvor programmet kjører.

Et av de viktigste målene utviklerne av Go hadde, var å lage Go "enkelt". I denne sammenhengen betyr enkelt at språket skulle ha få egenskaper, og de egenskapene språket fikk skulle være velbegrunnet og ikke bare være med fordi andre språk hadde de egenskapene. For eksempel har Go ingen arv i tradisjonell forstand, ikke genreics og bare én type løkker. Ved første øyekast kan språket virke litt enkelt med få muligheter sammenlignet med andre språk. Men etterhvert som man jobber med det, vil man se at selv med begrensede egenskaper er språket veldig kraftig.

# Installasjon
Go har støtte for de fleste plattformer og lastes ned fra Go sine sider http://golang.com. Etter installasjonen har man tilgang til kommandoen *go*, og kan dermed utføre følgende kommandoer.

```
// Kjører programmet hello.go.
go run hello.go

// Kompilerer alle go filer i mappen du står og lager en eksekverbar fil
go install

// Formaterer koden til å følge Go sin standard
go fmt

// Starter en webserver på port 6060 med tilgang til all dokumentasjon
godoc -http=6060
```

# Hello world

Da er det på tide å vise litt Go-kode. Eksempelet nedenfor viser Go sin versjon av Hello World. Vi legger merke til at programmet ligger i pakken main og at vi importerer en annen pakke som heter "fmt". "fmt" er en pakke som følger med i distribusjonen av Go og brukes her til å skrive til konsollet. Vi skal senere se hvordan vi kan importere eksterne biblioteker som ikke er laget av Go. Til slutt har vi en main-funksjon hvor vi skriver ut "Hello world!".

```go
package main

import "fmt"

func main() {
	fmt.Printf("Hello world!")
}
```

Hvis vi lagrer programmet i filen *hello.go* vil vi kunne kjøre programmet med følgende kommando:

```
go hello.go
```

# Typer
Go har ingen klasser slik vi kjenner fra språk som Java og C#, men man bruker *structs* (kjent fra blant annet C) for å deklarere typer. Eksempelet nedenfor viser hvordan vi kan definere en type *Page* som har attributtene *Header* og *Body*, begge av typen *string*.

Videre ser vi hvordan vi oppretter en funksjon *appendToBody* som kan kalles på et Page-objekt.

Til slutt ser vi hvordan vi kan opprette en instans av *Page* med den innebygde funksjonen *new*. Denne funksjonen vil allokere minne til typen og returnerer en peker (addressen til hvor i minne objektet ligger) til objektet.

```go
type Page struct {
	Header 	string
	Body 		string
}

func (page *Page) appendToBody(message string) {
	page.Body.append(message)
}

page := new(Page)
page.appendToBody("my new message")
```

# Channels og go-routines
Foreløpig har vi kun sett på de mest grunnleggende tingene med Go, og du tenker kanskje at det ikke er så mye med Go som skiller seg fra andre språk du har sett før? En av de mest spennende tingene med Go er hvordan språket håndterer samtidighet (eng: concurrency), og det skal vi se på nå.

I mange språk implementeres parallellitet ved å sikre tilgang til delte variabler. Dette er ofte komplekst og krever stor kunnskap for å få det helt korrekt. Go har derimot en annen tilnærming til denne problemstillingen og kommer med et konsept som kalles for go-rutiner (eng: go routines) og kanaler (eng: channels). En go-rutine kan sees på som en lettvekts prosess og disse prosessene kommuniserer ved å sende data til hverandre over en eller flere kanaler. Dette fører til at en delt variabel aldri vil blir forsøkt aksessert av to prosesser samtidig. Dette fører igjen til at det er mindre sannsynlighet for at race conditions eller deadlock-situasjoner kan oppstå fordi to prosesser *by design* ikke prøver å aksessere et delt minne samtidig. Go har utledet denne måten å tenke på til følgende slagord:

> *Do not communicate by sharing memory; instead, share memory by communicating.*

Så la oss se på noen konkrete eksempler på hvordan denne tankegangen er implementert i Go. Eksempelet nedenfor viser hvordan man oppretter kanalen *my_chan*, hvordan man sender en melding på denne kanalen og  til slutt hvordan man henter ut en verdi fra kanalen og lagrer denne i variabelen *a*. Det å opprette kanalen og sende en melding på denne, er såkalte ikke-blokkerende operasjoner, det vil si at programmet vil utføre disse to operasjonene og så fortsette umiddelbart. Derimot så er det å vente på svar fra en kanal en blokkerende operasjon. Det vil si at programmet vil blokkere helt til det faktisk ligger data på kanalen *my_chan* og først da vil programmet hente ut denne verdien og lagre den i variabelen *a*.

Til slutt ser vi hvordan vi starter en go-rutine ved hjelp av nøkkelordet go og så funksjonen vi ønsker å starte som en go-rutine. Det interessante er at *my_function* er en helt vanlig funksjon uten noen spesielle egenskaper, og hvis vi hadde fjernet *go* foran kallet til funksjonen, hadde vi utført et vanlig synkront funksjonskall.

```go
//Oppretter kanal
my_chan := make(chan int)

// Sender en verdi på kanalen
my_chan <- 1

// Venter på svar og lagrer verdien i en variabel
a := <- my_chan

// Starter my_function som en go-rutine
go my_function()
```

Kunnskapen om at en go-rutine vil blokkere ved en slik operasjon kan for eksempel bruke til å synkronisere en eller flere go-rutiner, slik at de utfører en oppgave annenhver gang. Eksempelet nedenfor viser nettopp dette. 

```go
package main

import (
	"fmt"
	"time"
)

type Ball struct { 
	hits int 
}

func main() {
	table := make(chan *Ball)
	go player("ping", table)
	go player("pong", table)

	table <- new(Ball)
	time.Sleep(2 * time.Second)
	<-table
}

func player(name string, table chan *Ball) {
	for {
	  ball := <-table
	  ball.hits++
	  fmt.Println(name, ball.hits)
	  time.Sleep(300 * time.Millisecond)
	  table <- ball
	}
}
```

Etter den obligatoriske pakkedeklarasjonen og et par pakkeimportering, starter vi med å definere en struct *Ball*. I main-funksjonen oppretter vi en kanal *table* ved hjelp av den innebygde funksjonen *make*. Kanaler er typet og vi kan dermed sende objekter av typen *Ball* på denne kanalen. Så starter vi to go-rutiner ved å bruke nøkkelordet *go* etterfulgt av funksjonen vi ønsker skal eksekveres som en go-rutine.

Funksjonen *player* tar i mot navnet på spilleren, samt kanalen *table*. Funksjonen inneholder en evig løkke som vil utføres så lenge programmet kjører. Første linjen i løkka venter på at det kommer en melding på kanalen *table* og lagrer så denne verdien i variabelen *ball*. Videre øker vi antall treff, og skriver ut status før vi sender en melding tilbake på *table* (programmet sover i 300 millisekunder kun for å få en bedre effekt av programutskriften).

Når vi har kommet så langt så står begge go-rutinene å venter på at det skal komme en melding på kanalen. Tilbake i hovedprogrammet starter vi så selve spillet med å sende en melding på *table*. Spilleren *ping* vil nå stå klar til å hente ut denne meldingen. Meldingen vil å bli plukket av, antall treff vil bli økt før den til slutt legger ballen tilbake på bordet. Deretter vil *pong* stå å vente på en melding fra *table* og vil utføre samme operasjoner. Og *ping* og *pong* vil så hente ut en melding fra kanalen annenhver gang.

I main-funksjonen sover vi så i et par sekunder (mens *ping* og *pong* spiller) før vi fjerner meldingen fra *table* og forkaster denne. Dermed vil programmet avsluttes.

Resultatet av kjøringen av programmet vil være som følger

```go
ping 1
pong 2
ping 3
pong 4
ping 5
pong 6
ping 7
pong 8
```

Som nevnt er en go-rutine en lettvekts prosess og et vanlig spørsmål er hvordan Go håndterer disse "prosessene". Målet med Go er å abstrahere bort alle kompliserte detaljer rundt synkronisering av samtidige prosesser slik at utviklerne ikke trenger å forholde seg til tråder og synkronisering av disse. For eksempel så kan to go-rutiner kjøre på samme OS-tråd, men hvis en av de to go-rutinene blokkerer, vil den ene go-rutinen automatisk (og transparent for utvikleren) bytte til en annen OS-tråd. Poenget er at utvikleren ikke skal trenger å forholde seg til disse detaljene, men opprette go-rutiner i programmet og la Go selv håndtere det på en best mulig måte.

# RSS-feeder
I introen til bloggposten nevnte jeg at jeg skulle vise hvordan man kan skrive en HTTP-backend i Go og deploye denne til Heroku. Som eksempel skal jeg vise en veldig enkel RSS-leser, det vil si vi skal lage backenden til denne RSS-leseren i Go. Programmet skal tilby et REST-grensesnitt med følgende operasjoner:

- Oppdatere alle lagrede feeds
- Opprette en ny feed
- Markere et item som lest

Jeg kommer ikke til å vise alle detaljer fra programmet, men for de som er interessert er kildekoden tilgjengelig på https://github.com/henriwi/gorss og applikasjonen kjører på http://simplyrss.herokuapp.com/

Det første vi trenger er en mekanisme for å ta i mot HTTP-kall og route disse videre nedover i programmet. Go kommer med en innebygd http-pakke med god støtte for å sette opp en web-server. Men i dette eksempelet skal vi bruke en ekstern pakke *martini* som er en utvidelse av Go sin http-pakke. 

Martini er tilgjengelig på GitHub på følgende URL: https://github.com/SlyMarbo/rss. For å si til Go at vi ønsker å bruke denne eksterne pakken importerer vi den på følgende måte

```go
import (
	"github.com/SlyMarbo/rss"
)
```

Hvis vi nå prøver å kompilere programmet vil Go klage fordi den ikke finner denne pakken. Vi må derfor kjøre følgende kommando som vil hente alle eksterne pakker (sjekke ut koden fra GitHub) og kompilere det til en pakke som vi så kan importere i vårt program.

```go
go get
```

Nedenfor viser vi hvordan Martini kan brukes til å sette opp 4 endepunkter, et endepunkt for / hvor vi serverer statiske filer som html, javascript og css, og fire endepunkter for api-et vårt. *FetchFeeds*, *AddFeed*, *DeleteFeed* og *MarkUnread* er funksjoner som håndterer de ulike forespørslene som klienten gjør.

```go
m := martini.Classic()
m.Use(martini.Static("static"))
m.Get("/api/feed", FetchFeeds)
m.Post("/api/feed", AddFeed)
m.Delete("/api/feed", DeleteFeed)
m.Post("/api/feed/read", MarkUnread)

http.ListenAndServe(":"+os.Getenv("PORT"), m)k
```

## Fetch feeds
Vi skal gå litt i detalj på en funsjon fra RSS-lesere vår, nemlig *FetchFeeds*. Denne er ansvarlig for å hente alle feedsene som brukeren har lagret, oppdatere disse og sende resultatet tilbake til klienten. Her skal vi se hvordan vi kan bruke go-rutiner og kanaler for å oppdatere dette asynkront.

Tanken bak implementasjonen er å se gjennom alle feeds vi skal oppdatere. Så for hver feed setter vi i gang en go-rutine som oppdaterer feeden og returnerer den oppdaterte feeden, eller en feil hvis noe har gått galt under hentingen. Hvis brukeren har lagret 100 feeds vil programmet starte opp 100 go-rutiner som vil jobbe i parallell for å hente innholdet. Men hvordan kan vi synkronisere disse rutinene, og returnere resultatet til klienten når alle er ferdig? Til dette kan vi bruke en kanal.

Hver enkelt go-rutine vil legge resultatet av forespørselen på en kanal. Dermed kan vi lytte på denne kanalen og hente ut meldingene etterhvert som de ulike go-rutinene legger meldinger på denne kanalen. Når vi har fått like mange meldinger tilbake som antall go-rutiner vi startet, vet vi at vi er ferdig og vi kan returnere hele resultatet (alle feedene) til klienten.

Nedenfor vises en hjelpefunksjon *asyncFetchFeeds* som gjør det vi nå har forklart. For hver feed startes en anonym funksjon som en go-rutine og legger resultatet av oppdateringen på kanalen *ch*. Etter å ha startet alle go-rutinene går vi inn i en løkke hvor vi hjelp av en *select case* sjekker om vi har fått en melding på kanalen. Når det kommer en melding på kanalen, henter vi ut denne, og legger den til i et array. Når vi har fått alle meldingene returnerer vi arrayet som inneholder selve feeden, samt en evt. feil. I tillegg har vi en ekstra case som returnerer alle responsene etter 5 sekunder, hvis f.eks. en URL ikke svarer.

```go
func asyncFetchFeeds(feeds []*rss.Feed) []*HttpResponse {
	ch := make(chan *HttpResponse)
	responses := []*HttpResponse{}

	for _, feed := range feeds {
		go func(f *rss.Feed) {
			fmt.Printf("Fetching %s\n", f.UpdateURL)
			feed, err := rss.Fetch(f.UpdateURL)

			if err != nil {
				fmt.Printf("Error in response %s. Using old feed.\n", err)
				ch <- &HttpResponse{f, err}
			} else {
				ch <- &HttpResponse{feed, err}
			}

		}(feed)
	}

	for {
		select {
		case r := <-ch:
			fmt.Printf("%s was fetched\n", r.feed.UpdateURL)
			responses = append(responses, r)
			if len(responses) == len(feeds) {
				return responses
			}
		case <-time.After(5 * time.Second):
			return responses
		}
	}
}
```

Selve FetchFeeds-funksjonen er vist nedenfor og her ser vi at vi starter med å hente alle feeds fra databasen før vi kaller hjelpefunksjonen *asyncFetchFeeds*. Denne returnerer som vi så et array med alle responsene. Merk at kallet til asyncFetchFeeds er et vanlig synkront funksjonskall og vil ikke returnere før alle feedsene er blitt forsøkt hentet.

Vi ser så gjennom alle responsene vi har fått og legger vi til alle feeds i et nytt array som inneholder resultatet vi til slutt skal sende til klienten. Til slutt serialiserer vi resultatet til JSON og sender dette til klienten.

```go
func FetchFeeds(writer http.ResponseWriter, request *http.Request) {
	rss.CacheParsedItemIDs(false)
	feeds, err := db.GetAll()

	if err != nil {
		writer.WriteHeader(http.StatusInternalServerError)
	}

	if len(feeds) == 0 {
		return
	}

	responses := asyncFetchFeeds(feeds)

	result := []*rss.Feed{}
	for _, r := range responses {
			result = append(result, r.feed)
	}

	jsonResult, _ := json.Marshal(result)
	fmt.Fprintf(writer, string(jsonResult))
}
```

Det er selvfølgelig mange mulige implementasjoner av løsningen ovenfor, og jeg mener på ingen måte at den presenterte løsningen er den beste. Samtidig synes jeg løsningen presentert har en del interessante sider som kan være verdt å ta med seg.

## Heroku

# Oppsummering
Gjennom denne bloggposten har jeg vist hvordan man laster ned og installerer Go, kjører Go programmer og vise flere eksempler skrevet i Go. Jeg har også vist et bruksområde jeg mener Go egner seg godt til, nemlig som en HTTP-backend. I dag ser vi flere og flere eksempler på arkitekturer bestående av såkalte mikrotjenester, små tjenester/applikasjoner som gjør én ting. I motsetning til tidligere hvor det var vanligere med monolittiske applikasjoner som skulle utføre "alt". I tillegg ser vi en utvikling hvor det blir vanligere og separere frontend og backend til en applikasjon, gjerne skrevet som to selvstendige applikasjoner. Her mener jeg at Go passer godt som et alternativ til en slik backend-tjeneste. 

Hvis har lyst til å lære mer om Go, og kanskje prøve Go selv, så anbefaler jeg følgende ressurser:

* http://golang.org – Go sin hjemmeside med god dokumentasjon av selve språket, samt mye linker til andre ressurser
* 
* http://www.youtube.com/watch?v=f6kdp27TYZs – Go Concurrency Patterns (Google IO 2012)
* http://www.youtube.com/watch?v=QDDwwePbDtw - Advanced Go Concurrency Patterns (Google IO 2013)
* http://www.golang-book.com/assets/pdf/gobook.pdf - An Introduction to Programming in Go