# Go som backend til din neste webapplikasjon

Go er et programmeringsspråk som er utviklet av Google og har fokus på enkelhet, stabilitet og ytelse. Videre er Go et språk som prøver å bygge en bro mellom effektive, statisk typede språk og produktive, dynamiske språk. Go kommer med kraftige verktøy og et rikt standardbibliotek, og sammen med Go sin modell for parallellitet gjør dette at man enkelt skal kunne skrive applikasjoner med høy ytelse. I denne blogposten vil jeg vise hvordan man kan skrive en HTTP-backend i Go, deploye den til Heroku og forhåpentligvis inspirere deg til å teste ut Go selv. I tillegg er målet å vise deg hvor gøy det er å jobbe med Go.

# Kort intro til språket
- Kompilerer til maskinkode (en binary)
- Statisk linking (ingen dynamiske avhengigheter av biblioteker)
- Syntaks inspirert av C
- "Enkelt" - få features og de featuresene som er tatt med er virkelig gjennomtenkt
- Ingen klasser, arv, generics

# Hello world

Da er det på tide å vise litt Go-kode. Eksempelet nedenfor viser Go sin versjon av Hello World. Vi legger merke til at programmet ligger i pakken main og at vi importerer en annen pakke som heter "fmt". "fmt" er en pakke som følger med i distribusjonen av Go og brukes blant annet til å formatere og skrive ut output. Vi skal senere se hvordan vi kan importere eksterne biblioteker. Til slutt har vi en main-funksjon hvor vi skriver ut "Hello world!" til konsollet. 

Hvis vi lagrer programmet i en fil *hello.go* vil vi kunne kjøre programmet med følgende kommando:

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

I mange språk implementeres parallellitet ved å sikre tilgang til delte variabler. Dette er ofte komplekst og krever stor kunnskap for å få helt korrekt. Go har derimot en annen tilnærming til denne problemstillingen. Go har et konsept som kalles for go-rutiner (eng: go routines) og kanaler (eng: channels). En go-rutine kan sees på som en lettvektsprosess og disse prosessene kommuniserer ved å sende data til hverandre over en eller flere kanaler. Dette fører til at en delt variabel aldri vil blir forsøkt aksessert av to prosesser samtidig. Dette fører igjen til at det er mye mindre sannsynlighet for race conditions eller deadlock-situasjoner kan oppstå, sammenlignet med mer tradisjonelle språk. Go har utledet denne måten å tenke på til følgende slagord:

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

Kunnskapen om at en go-rutine vil blokkere ved en slik operasjon kan vi bruke videre til å lage et større eksempel. Eksempel har som mål å vise en måte vi kan synkronisere to go-rutiner til å utføre en oppgave annenhver gang. 

```go
package main

import (
	"fmt"
	"time"
)

// STARTMAIN1 OMIT
type Ball struct{ hits int }

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