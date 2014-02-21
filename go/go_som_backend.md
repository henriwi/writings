<!--Go er et programmeringsspråk som er utviklet av Google og har fokus på enkelhet, stabilitet og ytelse. I denne blogposten vil jeg vise hvordan du kan komme i gang med Go, vise noen eksempler og vise hvordan man kan skrive en HTTP-backend i Go og deploye denne til Heroku. Forhåpentligvis vil dette inspirere deg til å teste ut Go selv, og at du også merker hvor gøy det er å jobbe med Go!-->

# Introduksjon
Go er laget av Robert Griesemer, Rob Pike og Ken Thompson med Google i ryggen. Den første offisielle versjonen ble sluppet i 2009 og språket er nå i versjon 1.2. Språket prøver å bygge en bro mellom effektive, statisk typede språk og produktive, dynamiske språk, og kommer med kraftige verktøy og et rikt standardbibliotek. Sammen med Go sin modell for parallellitet gjør dette at man enkelt skal kunne skrive applikasjoner med høy ytelse.

Go kompilerer til maskinkode, og resultatet er en binary som man kan kjøre på den plattformen man har kompilert programmet for. Videre har Go statisk linking av biblioteker, det vil si at alle bibliotekene programmet ditt bruker blir pakket sammen med den eksekverbare versjonen av programmet. Den store fordelen med dette er at man ikke har noen dynamiske avhengigheter som må være på plass på maskinen hvor programmet kjører.

Et av de viktigste målene utviklerne av Go hadde, var å lage Go "enkelt". I denne sammenhengen betyr enkelt at språket skulle ha få egenskaper, og de egenskapene språket fikk skulle være velbegrunnet og ikke bare være med fordi andre språk hadde de egenskapene. For eksempel har Go ingen muligheter for arv i tradisjonell forstand, ingen generics og bare én type løkker. Ved første øyekast kan språket muligens virke enkelt med få muligheter sammenlignet med andre språk. Men etterhvert som man jobber med det, vil man se at selv med begrensede egenskaper er språket veldig kraftig.

Noen av de viktigste egenskapene til Go kan oppsumeres som følgende punkter

* Concurrency er bakt inn i kjernen av språket, og ikke som et bibliotek.
* Go er statisk typet, men ved å støtte [Duck-typing](http://en.wikipedia.org/wiki/Duck_typing) og [automatisk typeinferens](http://en.wikipedia.org/wiki/Type_inference), føles språket nesten som et dynamisk språk.
* Rask kompileringstid selv på store prosjekter med mange filer. Det har vært et mål hele veien å holde kompileringstiden så kort som mulig.
* Garbage collection
* Ingen dynamiske avhengigheter som kreves på serveren programmet skal kjøre
* Rikt standardbibliotek og flere og flere eksterne pakker
* God dokumentasjon

# Installasjon
Go har støtte for de fleste plattformer og lastes ned fra [Go](http://golang.com.) sine sider. Etter installasjonen har man tilgang til kommandoen ```go```, og kan dermed utføre følgende kommandoer.

```
// Kompilerer og kjører programmet hello.go.
go run hello.go

// Kompilerer alle go filer i mappen du står og lager en eksekverbar fil
go install

// Formaterer alle go filer i mappen du står til å følge Go sin kodestandard
go fmt

// Installerer den eksterne pakke Martini som ligger på GitHub
go get github.com/codegangsta/martini

// Starter en webserver på port 6060 med tilgang til all Go-dokumentasjon
godoc -http=6060
```

# Syntaks
Så hvordan ser egentlig Go-kode ut? Syntaksen til Go er sterkt inspirert av C, og hvis du har jobbet litt med det språket vil du nok kjenne igjen en del ting. Eksempelet nedenfor viser et lite program som legger sammen to tall og skriver dette til konsollet.

```go
package main

import "fmt"

func add(x, y int) int {
	return x + y
}

func main() {
    fmt.Println(add(5, 3))
}
```

Noen umiddelbare forskjeller mellom Go og mer tradisjonelle språk er at typen til en variabel kommer *etter* navnet til variabelen. Det samme gjelder for parametre til funksjoner. En annen ting som gjør Go annerledes er måten det skilles mellom variabler som er tilgjengelig innenfor samme pakke, og variabler som er tilgjengelig utenfor pakken. Man bruker ikke nøkkelord (som *private* og *public* i Java), men variabler og funksjoner som starter med stor forbokstav er tilgjengelig utenfor pakken, mens variabler og funksjoner med liten forbokstav kun er tilgjengelig innenfor den pakken de er deklarert i.

I eksempelet over er dermed funksjonen *add* kun tilgjenlig innenfor pakken *main*. Vi legger også merke til at programmet importerer pakkeen *fmt*. *fmt* er en pakke som følger med i distribusjonen av Go og brukes her til å skrive til konsollet. Hvis vi lagrer programmet i filen *add.go* kan vi kjøre programmet med følgende kommando.

```
go add.go
```

# Structs og interface
Go har ingen klasser slik vi kjenner fra språk som Java og C#, men man bruker *structs* (kjent fra blant annet C) for å deklarere typer. Eksempelet nedenfor viser hvordan vi kan definere en type ```Page``` og en funksjon som kan kalles på denne typen ```appendToBody```. Siden Go ikke har klasser har man heller ikke metoder, men man kan definere funksjoner som kan kalles av en struct. Til slutt ser vi hvordan vi kan opprette en instans av ```Page``` med den innebygde funksjonen ```new```. Denne funksjonen vil allokere minne til typen og returnere en peker (addressen til hvor i minne objektet ligger) til objektet.

```go
type Page struct {
	Header  string
	Body    string
}

func (page *Page) appendToBody(message string) {
	page.Body.append(message)
}

page := new(Page)
page.appendToBody("my new message")
```

I tillegg til *structs* har Go *interface* hvor man definerer et sett med metoder som en type må ha for å implementere dette interfacet. I mange språk vil en klasse eller type eksplisitt implementere et eller flere interface (i Java gjennom nøkkelordet *implements*). I Go sier man derimot at så lenge en type B har de samme funksjonene (med samme signatur) som interface A har deklarert, implementerer B interface A uten at dette trenger å deklareres eksplisitt. Eksempelet nedenfor viser hvordan vi kan lage vår egen ```ConsoleWriter```som implementerer Go sitt innerbygde interface ```Writer```.

```go
// Go sitt interface Writer
type Writer interface {
  Write(p []byte) (n int, err error)
}

// Deklarerer typen ConsoleWriter med én funksjon Write() med samme signatur som interface Writer
// Go ser ved kompileringstid at ConsoleWriter implementerer Writer
// Legg merke til at det ikke er noen eksplisitt kobling mellom ConsoleWriter og Writer
type ConsoleWriter struct{}

func (cw *ConsoleWriter) Write(p []byte) (n int, err error) {
   fmt.Printf("%s", p)
   return
}

// Oppretter en ConsoleWriter
// Sender så referansen til metoden Fprintf som tar i mot alle Writer-objekter
// Fprintf vil igjen kalle Write på ConsoleWriter-objektet
func main() {
	cw := new(ConsoleWriter)
	fmt.Fprintf(cw, "Tester min egen ConsoleWriter")
}
```

# Kanaler og go-rutiner
I mange språk implementeres parallellitet ved å sikre tilgang til delte variabler, ofte i form av en lås. Dette kan virke enkelt ved første øyekast, men kan fort bli komplekst og kreve stor kunnskap for å få det helt korrekt. Go har derimot en annen tilnærming til denne problemstillingen og kommer med et konsept som kalles for go-rutiner (eng: go routines) og kanaler (eng: channels). Dette er inspirert av [Communicating sequential processes](http://en.wikipedia.org/wiki/Communicating_sequential_processes), første gang beskrevet av Tony Hoare. En go-rutine kan sees på som en lettvekts prosess og disse prosessene kommuniserer ved å sende de delte dataene mellom hverandre over en kanal. Dette fører til at en delt variabel aldri vil bli forsøkt aksessert av to prosesser samtidig (fordi det er kun én go-rutine som har tilgang på de delte dataene av gangen), som igjen fører til at det er mindre sannsynlighet for at race conditions eller deadlock-situasjoner kan oppstå. Go har utledet denne måten å tenke på til følgende slagord:

> *Do not communicate by sharing memory; instead, share memory by communicating.*

Så la oss se på hvordan vi kan bruke kanaler og go-rutiner.

```go
//Oppretter kanalen my_chan
my_chan := make(chan int)

// Sender en verdi på kanalen
my_chan <- 1

// Venter på svar og lagrer verdien i en variabel
a := <- my_chan

// Starter my_function som en go-rutine
go my_function()
```

Det å opprette en kanal og sende en melding på denne, er såkalte ikke-blokkerende operasjoner, det vil si at programmet vil utføre disse to operasjonene og så fortsette umiddelbart. Derimot så er det å vente på svar fra en kanal en blokkerende operasjon. Det vil si at programmet ovenfor vil blokkere helt til det faktisk ligger data på kanalen ```my_chan``` og først da vil programmet hente ut denne verdien og lagre den i variabelen ```a```. Det er også vært å legge merke til at ```my_function``` er en helt vanlig funksjon uten noen spesielle egenskaper, og hvis vi hadde fjernet ```go``` foran kallet til funksjonen hadde vi utført et vanlig synkront funksjonskall.

Kunnskapen om at en go-rutine vil blokkere når den venter på data fra en kanal, brukes ofte til å synkronisere en eller flere go-rutiner slik at de utfører en oppgave annenhver gang. Eksempelet nedenfor viser nettopp dette. 

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

Etter pakkedeklarasjonen og importering av noen pakker, starter vi med å definere en struct ```Ball```. I main-funksjonen oppretter vi en kanal ```table``` ved hjelp av den innebygde funksjonen ```make```. Kanaler i Go er typet og vi kan dermed sende objekter av typen ```Ball``` på denne kanalen. Så starter vi to go-rutiner ved å kalle ```go player("ping", table)```og ```go player("pong", table)```.

Funksjonen ```player``` tar i mot navnet på spilleren, samt kanalen ```table```. Funksjonen inneholder en evig løkke som vil utføres så lenge programmet kjører. Første linjen i løkka venter på at det kommer en melding på kanalen ```table``` og lagrer så denne verdien i variabelen ```ball```. Videre øker vi antall treff, og skriver ut status før vi sender en melding tilbake på ```table``` (programmet sover i 300 millisekunder kun for å få en bedre visuell effekt av programutskriften).

Begge go-rutinene vil først stå å vente på at det skal komme en melding på kanalen ```table```. I main-funksjonen starter vi selve spillet med å sende en melding på ```table```. Spilleren ```ping``` vil først hente ut denne meldingen, øke antall treff, før den til slutt legger ballen tilbake på bordet. Deretter vil ```pong``` stå å vente på en melding fra ```table``` og vil utføre samme operasjoner. ```ping``` og ```pong``` vil så kjøre annenhver gang.

I main-funksjonen sover vi så i et par sekunder (mens ```ping``` og ```pong``` spiller) før vi fjerner meldingen fra ```table``` og forkaster denne. Dermed vil programmet avsluttes. Utskriften fra programmet vil være som følger

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

Målet med Go er å abstrahere bort alle kompliserte detaljer rundt parallelle prosesser slik at utviklerne ikke trenger å forholde seg til unødige og komplisert detaljer. For eksempel så kan to go-rutiner kjøre på samme OS-tråd, men hvis en av de to go-rutinene blokkerer, vil den ene go-rutinen automatisk (og transparent for utvikleren) flyttes til en annen kjørende tråd. Poenget er at utvikleren ikke skal trenge å forholde seg til disse detaljene, men opprette go-rutiner i programmet og la Go selv håndtere disse på en best mulig måte.

# HTTP-backend
I introen til bloggposten nevnte jeg at jeg skulle vise hvordan man kan skrive en HTTP-backend i Go og deploye denne til Heroku. Som eksempel skal jeg ta utgangspunkt i en veldig enkel RSS-leser, det vil si vi skal lage backenden til denne RSS-leseren i Go. Jeg kommer ikke til å vise alle detaljer fra programmet, men for de som er interessert er kildekoden tilgjengelig på https://github.com/henriwi/gorss.

Det første vi trenger er en mekanisme for å ta i mot HTTP-kall og route disse videre nedover i programmet. Go kommer med en innebygd http-pakke med god støtte for å sette opp en web-server. Men i dette eksempelet skal vi bruke en ekstern pakke [Martini](https://github.com/codegangsta/martini) som er en utvidelse av Go sin http-pakke, med blant annet bedre støtte for REST. Nedenfor viser vi hvordan Martini kan brukes til å lage et endepunkt for å hente alle feedsene til brukeren på URL-en ```/api/feed``` før vi starter en webserver som lytter på port 8080.

```go
m.Get("/api/feed", FetchFeeds)

http.ListenAndServe(":8080", m)
```

```FetchFeeds``` er en funksjon med en signatur som gjør at den kan bli sendt inn som argument til ```Get```. Som vi ser støtter Go å sende inn funksjoner som argument til andre funksjoner, såkalt [First class function](http://en.wikipedia.org/wiki/First-class_function). *FetchFeeds* ser slik ut (med litt pseduokode). Dette er egentlig alt

```go
func FetchFeeds(writer http.ResponseWriter, request *http.Request) {
	// Hent alle feeds
	results := <hent og oppdater alle feeds>

	// Serialiserer resultatet til json og skriv resultatet til responsen
	jsonResult, _ := json.Marshal(result)
	fmt.Fprintf(writer, string(jsonResult))
}
```

Vi skal gå litt mer i detalj på funksjonen ```FetchFeeds``` og delen ```hent og oppdater alle feeds```. Denne er ansvarlig for å hente alle feedsene som brukeren har lagret, oppdatere disse og sende resultatet tilbake til klienten. Tanken bak implementasjonen er å se gjennom alle feeds vi skal oppdatere. For hver feed setter vi i gang en go-rutine som oppdaterer feeden og returnerer den oppdaterte feeden, eller en feil hvis noe har gått galt under hentingen. Hvis brukeren for eksempel har lagret 100 feeds vil programmet starte opp 100 go-rutiner som vil jobbe i parallell for å hente innholdet. Men hvordan kan vi synkronisere disse rutinene, og returnere resultatet til klienten når alle er ferdig? Til dette kan vi bruke en kanal.

Hver enkelt go-rutine vil legge resultatet av forespørselen på én kanal. Dermed kan vi lytte på denne kanalen og hente ut meldingene etterhvert som de ulike go-rutinene legger meldinger på denne kanalen. Når vi har fått like mange meldinger tilbake som antall go-rutiner vi startet, vet vi at vi er ferdig og vi kan returnere hele resultatet (alle feedene) til klienten. Nedenfor vises den aktuelle delen av koden som utfører dette. En ```select case``` fungerer på samme måte som en ```switch case```, hvor vi velger det caset der det ligger en melding på kanalen.

```go
ch := make(chan *HttpResponse)
responses := []*HttpResponse{}

for _, feed := range feeds {
	go func(f *rss.Feed) {
		feed, err := rss.Fetch(f.UpdateURL)

		if err != nil {
			ch <- &HttpResponse{f, err}
		} else {
			ch <- &HttpResponse{feed, err}
		}

	}(feed)
}

for {
	select {
	case r := <-ch:
		responses = append(responses, r)
		if len(responses) == len(feeds) {
			return responses
		}
}
```

Det er selvfølgelig mange mulige implementasjoner av løsningen ovenfor, og jeg har jeg utelatt litt feilhåndtering for å gjøre koden enklere. Samtidig synes jeg løsningen presentert har en del interessante sider som kan være verdt å ta med seg videre. Spesielt hvordan vi etter å ha satt i gang oppdateringen av alle feedene brukeren en kanal til å hente ut en og en feed.

# Deploy til Heroku eller Google App Engine
Nå som du har sett hvordan vi kan lage en HTTP-backend i Go, er du forhåpentligvis interessert i å deploye dette slik at man kan ta det i bruk. Både Heroku og Google App Engine har støtte for å deploy Go-applikasjoner (Heroku gjennom en ekstern [buildpack](https://github.com/kr/heroku-buildpack-go)). Selv om det er mulig å deploye til begge plattformer, er det en del [forskjeller](http://james-eady.com/blog/2013/08/06/hosted-golang-heroku-vs-google-app-engine) hvor den viktigste er at Google App Engine krever at du må bruke flere egne biblioteker tilpasset App Engine for å deploye applikasjonen din, mens man på Heroku kan bruke Go sine standard bibliotek. 

Personlig vil jeg anbefale å følge denne [guiden](http://mmcgrana.github.io/2012/09/getting-started-with-go-on-heroku.html) for en stegvis guide for å deploye til Heroku. Hvis du er interessert i å se på deploy til Google App Engine anbefaler jeg Google sin egen [dokumentasjon](https://developers.google.com/appengine/docs/go/).

# Oppsummering
Gjennom denne bloggposten har jeg vist hvordan man laster ned og installerer Go, kjører Go programmer og vist flere eksempler skrevet i Go. Jeg har også vist et bruksområde jeg mener Go egner seg godt til, nemlig som en HTTP-backend. I dag ser vi flere og flere eksempler på arkitekturer bestående av såkalte mikrotjenester, små tjenester/applikasjoner som gjør én ting. I motsetning til tidligere hvor det var vanligere med monolittiske applikasjoner som skulle utføre "alt". I denne konteksten mener jeg at Go er et interessant språk som det er vært å følge med på utviklingen til. Samtidig skiller Go seg en del ut fra mer tradisjonelle språk, og det kan være vært å lære seg Go av den enkle grunn at man vil lære seg litt nye måter og tenke på, som man kanskje til og med kan ta med seg inn i sitt "eget" språk i hverdagen.

Hvis du har lyst til å lære mer om Go, og kanskje prøve Go selv så anbefaler jeg følgende ressurser:

* Golang tour (test Go rett i nettleseren): http://tour.golang.org
* Go sin hjemmeside med god dokumentasjon av selve språket, samt mye linker til andre ressurser: http://golang.org
* An Introduction to Programming in Go: http://www.golang-book.com/assets/pdf/gobook.pdf
* Go Concurrency Patterns (Google IO 2012): http://www.youtube.com/watch?v=f6kdp27TYZs
* Advanced Go Concurrency Patterns (Google IO 2013): http://www.youtube.com/watch?v=QDDwwePbDtw
* Se hvem som bruker Go: https://code.google.com/p/go-wiki/wiki/SuccessStories