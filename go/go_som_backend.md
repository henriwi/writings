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
En av de mest spennende tingene ved Go er hvordan språket har innebygd 

> *Do not communicate by sharing memory; instead, share memory by communicating.*


```go
//Oppretter kanal
channel := make(chan int)

// Sender melding på en kanal
go channel <- 1

// Venter på svar og assigner til en variabel
svar := <- channel
```

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