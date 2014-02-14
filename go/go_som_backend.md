# Go som backend til din neste webapplikasjon

Go er et programmeringsspråk som er utviklet av Google og har fokus på enkelhet, stabilitet og ytelse. Videre er Go et språk som prøver å bygge en bro mellom effektive, statisk typede språk og produktive, dynamiske språk. Go kommer med kraftige verktøy og et rikt standardbibliotek, og sammen med Go sin modell for parallellitet gjør dette at man enkelt skal kunne skrive applikasjoner med høy ytelse. I denne blogposten vil jeg deg en introduksjon til Go, samt vise hvordan man kan skrive en HTTP-backend i Go, deploye den til Heroku og forhåpentligvis inspirere deg til å teste ut Go selv. I tillegg er målet å vise deg hvor gøy det er å jobbe med Go.

# Kort intro til språket
Go er utviklet av Google og XX XX, YY YY og ZZ ZZ står bak, og spårket ble sluppet i 2009. Språket kompilerer ned til maskinkode og resultatet er en binary som man kan kjøre som et hvilket som helst annet program på maskinen. Go har statisk linking av biblioteker, det vil si at alle bibliotekene programmet ditt bruker blir pakket sammen med den eksekverbare versjonen av programmet. Dette fører til at programmet blir noe større, men fordelen er at man ikke har noen dynamiske avhengigheter som må være på plass på maskinen hvor programmet kjører.

Et av de viktigste målene når Go ble utviklet var å lage det "enkelt". Enkelt i denne forstand betyr at språket skulle ha få egenskaper, og de egenskapene språket fikk skulle være velbegrunnet og ikke bare være med fordi andre språk hadde de egenskapene. Go har blant annet ingen arv i tradisjonell forstand, ikke genreics og bare én type løkker. Ved første øyekast kan språket kanskje virke for enkelt med få muligheter, men etterhvert som man jobber med det, vil man se at språket har alle de egenskapene som trengs.

# Installasjon
Go har støtte for de fleste plattformer og kan lastes ned på Go sine sider http://golang.com. Etter installasjonen skal man ha tilgang til blant annet følgende kommandoer.

```
// Kjører programmet hello.go. Fint for å teste go
go run hello.go

// Kompilerer alle go filer i mappen du står og lager en eksekverbar fil
go install

// Formaterer koden til å følge Go sin standard
go fmt

// Starter en webserver på port 6060 med tilgang til all dokumentasjon
godoc -http=6060
```

# Hello world

Da er det på tide å vise litt Go-kode. Eksempelet nedenfor viser Go sin versjon av Hello World. Vi legger merke til at programmet ligger i pakken main og at vi importerer en annen pakke som heter "fmt". "fmt" er en pakke som følger med i distribusjonen av Go og brukes blant annet til å skrive til standard out. Vi skal senere se hvordan vi kan importere eksterne biblioteker som ikke er laget av Go. Til slutt har vi en main-funksjon hvor vi skriver ut "Hello world!" til konsollet. 

Hvis vi lagrer programmet i filen *hello.go* vil vi kunne kjøre programmet med følgende kommando:

```
go hello.go
```

```go
package main

import "fmt"

func main() {
	fmt.Printf("Hello world!")
}
```

# Channels og go-routines
Foreløpig har vi kun sett på de mest grunnleggende tingene med Go, og som den observante leser du er, har du kanskje lurt på hva som gjør Go så annerledes og spennende som jeg snakket om i introduksjonen? Det skal vi prøve å gjøre noe med nå.

I mange språk implementeres parallellitet ved å sikre tilgang til delte variabler. Dette er ofte komplekst og krever stor kunnskap for å få det helt korrekt. Go har derimot en annen tilnærming til denne problemstillingen og kommer med et konsept som kalles for go-rutiner (eng: go routines) og kanaler (eng: channels). En go-rutine kan sees på som en lettvekts prosess og disse prosessene kommuniserer ved å sende data til hverandre over en eller flere kanaler. Dette fører til at en delt variabel aldri vil blir forsøkt aksessert av to prosesser samtidig. Dette fører igjen til at det er mye mindre sannsynlighet for race conditions eller deadlock-situasjoner kan oppstå, sammenlignet med mer tradisjonelle språk. Go har utledet denne måten å tenke på til følgende slagord:

> *Do not communicate by sharing memory; instead, share memory by communicating.*

Så la oss se på noen konkrete eksempler på hvordan denne tankegangen er implementert i Go. Eksempelet nedenfor viser hvordan man oppretter kanalen *my_chan*, hvordan man sender en melding på denne kanalen og  til slutt hvordan man henter ut en verdi fra kanalen og lagrer denne i variabelen *a*. Det å opprette kanalen og sende en melding på denne er såkalte ikke-blokkerende operasjoner, det vil si at programmet vil utføre disse to operasjonene og så fortsette umiddelbart. Derimot så er det å vente på svar fra en kanal en blokkerende operasjon. Det vil si at programmet vil blokkere helt til det faktisk ligger data på kanalen *my_chan* og først da vil programmet hente ut denne verdien, lagre den i variabelen *a* før det fortsetter.


```go
//Oppretter kanal
my_chan := make(chan int)

// Sender en verdi på kanalen
my_chan <- 1

// Venter på svar og lagrer verdien i en variabel
a := <- my_chan
```

Kunnskapen om at en go-rutine vil blokkere ved en slik operasjon kan vi bruke videre til å lage et større eksempel. Eksempelet har som mål å vise hvordan vi kan synkronisere to go-rutiner til å utføre en oppgave annenhver gang. 

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
Vi starter med å definere en struct Ball. Videre i main-funksjonen oppretter vi en kanal *table* ved hjelp av den innebygde funksjonen *make*. Kanaler er typet og vi kan sende objekter av typen *Ball* på denne kanalen. Så starter vi to go-rutiner ved å bruke nøkkelordet *go* etterfulgt av funksjonen vi ønsker skal eksekveres som en go-rutine. Det interessante er at hvis vi hadde fjernet *go* foran kallet til funksjonen *player* hadde vi utført et vanlig synkront funksjonskall.

Funksjonen *player* tar i mot navnet på spilleren, samt kanalen vi opprettet tidligere for å sende meldinger. Funksjonen inneholder en evig løkke som vil utføres så lenge programmet kjører. Første linjen i løkka venter på at det kommer en melding på kanalen *table* og når det kommer en verdi her vil denne lagres i variabelen *ball*. Videre vil vi øke antall treff, skrive ut en melding og vente litt (kun for å få en bedre visuell effekt av programutskriften) før vi sender en melding tilbake på *table*.

Tilbake i hovedprogrammet starter vi selve spillet med å sende en melding på *table*. Vi sover så i et par sekunder før vi venter til det kommer en ny melding på *table* og så fjerner vi denne. Dermed vil programmet avsluttes.

Når programmet legger en ny ball på bordet, så vil *ping* stå å vente på at det kommer en melding på kanalen. Meldingen vil å bli plukket av, antall treff vil bli økt før den til slutt legger ballen tilbake på bordet. Deretter vil *pong* stå å vente på en melding fra *table* og vil så utføre det samme. Og *ping* og *pong* vil å se hente ut en melding fra kanalen annenhver gang.

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


Som nevnt er en go-rutine en lettvekts prosess og et vanlig spørsmål er hvordan Go håndterer disse "prosessene". Målet med Go er å abstrahere bort alle kompliserte detaljer rundt synkronisering av samtidige prosesser slik at utviklerne ikke trenger å forholde seg til tråder og synkronisering av disse. F.eks. så kan to go-rutiner kjøre på samme OS-tråd, men hvis en av de to go-rutinene blokkerer, vil den ene go-rutinen automatisk (og transparent for utvikleren) bytte til en annen OS-tråd. Poenget er at utvikleren ikke skal trenger å forholde seg til disse detaljene, men opprette go-rutiner i programmet og la Go selv håndtere det på en best mulig måte.

# RSS-feeder
Til slutt skal jeg gå gjennom et litt større eksempel skrevet i Go. Det vi skal lage er en veldig enkel RSS-leser, det vil si vi skal lage backenden til denne RSS-leseren i Go. Programmet skal tilby et REST-grensesnitt med følgende operasjoner:

- Hente alle feeds
- Opprette en ny feed
- Markere et item som lest

Jeg kommer ikke til å vise alle detlajer fra programmet, men for de som er interessert er all kildekoden tilgjengelig på https://github.com/henriwi/gorss og applikasjonen kjører på http://simplyrss.herokuapp.com/

Det første vi trenger er en webserver som kan route de ulike HTTP-kallene videre nedover i programmet. Go kommer med en innebygd http-pakke med god støtte for å sette opp en web-server. Men i dette eksempelet skal vi bruke en ekstern pakke *martini* som er en utvidelse av Go sin http-pakke. Nedenfor viser vi hvordan vi bruker Martini til å sette opp 4 endepunkter, et endepunkt for / hvor vi serverer statiske filer. Dette er javascript, html og css som vi sender til klienten. Videre definerer vi opp tre endepunkter i api-et vårt. To GET-metoder og én POST. FetchFeeds, FetchFeed og AddFeed er funksjoner som håndterer de ulike forespørslene som klienten gjør.

Til slutt starter vi opp webserveren vår ved hjelp av go sin http-pakke.

```go
m := martini.Classic()
m.Use(martini.Static("static"))
m.Get("/api/feed", FetchFeeds)
m.Get("/api/feed/:id", FetchFeed)
m.Post("/api/feed", AddFeed)

http.ListenAndServe(":"+os.Getenv("PORT"), m)k
```

## Fetch feeds
Vi skal se mer detaljert på funksjonen FetchFeeds. Denne er ansvarlig for å hente alle feedsene som brukeren har opprettet og sende disse tilbake til klienten. Her skal vi se hvordan vi kan bruke go-rutiner og kanaler for å hente alle feedsene asynkront.

Tanken bak implementasjonen er å se gjennom alle url-ene vi skal hente feedsene for. Så for hver url setter vi i gang en go-rutine som henter innholdet på urlen og returnerer en feed eller en feil hvis noe har gått galt under hentingen. Så hvis brukeren har lagret 100 URL-er vil programmet vårt starte opp 100 go-rutiner som vil jobbe i parallell for å hente innholdet. Men hvordan kan vi synkronisere disse rutinene, og returnere resultatet til klienten når alle er ferdig? Til dette kan vi bruke en kanal.

Hver enkelt go-rutine vil legge resultatet av forespørselen på en kanal. Dermed kan vi lytte på denne kanalen og hente ut alle meldingene som ligger på denne. Når vi har fått like mange meldinger som antall go-rutiner vi startet vet vi at vi er ferdig og vi kan returnere hele resultatet (alle feedene) til klienten.

Eksempelet nedenfor viser en hjelpefunksjon *asyncFetchFeeds* som gjør det vi nå har forklart. For hver url starter den en anonym funksjon som en go-rutine og legger resultatet av funksjonen på en kanal. Etter å ha startet alle go-rutinene går vi inn i en løkke hvor vi hjelp av *select case* sjekker om vi har fått noe på kanalen. Hvis vi har så henter vi meldingen ut fra kanalen og legger dette til i en array. Når vi har fått alle meldingene returnerer vi arrayet som inneholder selve feeden, samt en evt. feil. I tillegg har vi en ekstra case som returnerer alle responsene etter 5 sekunder, hvis f.eks. en URL ikke svarer.


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

Selve FetchFeeds-funksjonen er vist nedenfor og her ser vi at vi starter med å hente alle feeds fra databasen før vi kaller hjelpefunksjonen asyncFetchFeeds. Denne returnerer som vi så et array med alle responsene. Merk at kallet til asyncFetchFeeds er et vanlig synkront funksjonskall og vil ikke returnere før alle feedsene er blitt forsøkt hentet.

Vi ser så gjennom alle responsene vi har fått og legger vi til alle feeds i et nytt array som inneholder resultatet vi til slutt skal sende til klienten. Til slutt bruker vi en JSON-parser til å serialisere typene våre til JSON før vi skriver dette til responsen.

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