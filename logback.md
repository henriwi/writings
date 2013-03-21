# Intro
Log4j har vært, og er fortsatt, loggrammeverket som benyttes for veldig mange Java-applikasjoner. Selv om log4j ikke lenger blir vedlikeholdt er det fortsatt mange som bruker dette og ikke har mulighet til å gå over til logback.

Logback har tatt over for log4j og tilbyr mange features som ikke er mulig å få til med log4j. Flere mener at logback vil bli brukt mer og mer, og at log4j gradvis vil bli faset ut. Det kan derfor lønne seg å skrive om applikasjonene som i dag bruker log4j til logback. Men hvordan gjøres dette? Hva er logback og hvor kommer SLF4J inne i bildet? I dette blogginnlegget vil jeg forklare hvordan man skriver om en applikasjon som bruker log4j til å bruke logback, og hvilke erfaringer vi har gjort oss med dette arbeidet.

<!-- Hva slags krav hadde vi til vår applikasjon - motivasjon for hvorfor vi ønsket å endre. -->
<!-- Zipping, tidsbasert rulling og ikke minst opprydning. -->

# Alternativer
Applikasjonen hadde allerede log4j som loggrammeverk. Derfor var det naturlig å se på hva vi kunne få til ved å bruke log4j eller versjoner av log4j hvor omskriving av applikasjonen ikke var nødvendig.

Log4j-extras kunne bidra med zipping og tidsbasert rulling, men kunne ikke rydde opp etter seg. Vi ønsket ikke å involere drift i denne saken og dermed skape en avhengighet til dem, og log4j extras ble derfor vurdert til å ikke være et alternativ.

Vi så da videre på at logback støtte de kravene vi hadde, nemlig; tidsbasert rulling, zipping og ikke minst opprydning av loggfilene. Den endelige løsningen ble dermed: SLF4J over logback.

# SLF4J
SLF4J er kun et api og ingen egen loggimplementasjon. Isteden er tanken at man programmerer mot ett felles API, men at man enkelt kan bytte ut loggimplementasjon kun ved å legge inn korrekt avhengighet på classpathen. 

Videre eksisterer det ulike bindings som man trenger i tillegg til den faktiske implementasjonen. slf4j-log4j12 vil route alle kall til slfj4-apiet til log4j og sluttresultatet vil bli helt likt som om man kun brukte log4j. 

Ved å kun bytte ut ulike jarfiler er det dermed mulig å endre loggimplementasjon uten å endre en eneste kodelinje!

# Logback
Er en videreføring av log4j. Tilbyr alt log4j tilbyr i tillegg til mye mer.
Har såkalt native støtte for slf4j. Logback implementerer interface til slf4j som gjør at det ikke er nødvendig å bruke en egen binding som trengs for å få log4j til å fungere med slf4j.

Kan konfigureres i xml eller til og med groovy.

--- Eksempelkonfigurasjon

# Omskriving til logback
Hva trengs for å skrive om applikasjonen til logback?.