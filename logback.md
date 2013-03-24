# Intro
Log4j har vært, og er fortsatt, loggrammeverket som benyttes av veldig mange Java-applikasjoner. Logback er ikke et spesielt nytt logrammeverk (release av versjon 0.1 ble gjort i 2006, og versjon 1.0 i 2011), men det er fortsatt mange nye applikasjoner som ikke bruker logback, men velger log4j. Selv om log4j fortsatt gjør nytten for mange, inneholder logback mange funksjoner som ikke støttes av log4j. I tillegg har logback innebgyd støtte for slf4j (mer om dette senere), noe som kort fortalt betyr at det er senere mulig å bytte ut logrammeverket ved kun å bytte en enkelt jar-fil med en annen.

Selv om i teorien er enkelt å skrive om en applikasjon fra log4j til logback kan man støte på flere hindringer som gjør det vanskeligere enn man først regnet med. Denne bloggposten viser hvordan man går frem for å skrive om en eksisterende applikasjon fra å bruke log4j til å bruke logback som loggrammeverk, og tar utgangspunkt i erfaringer vi har gjort oss på vårt prosjekt. Forhåpentligvis kan det være med å bidra til at andre ikke støter på de samme utfordringene som vi gjorde. I tillegg skal vi vise hvordan man kan sikre at logging fra rammeverk brukt av applikasjonen kan bli håndtert av det samme loggoppsettet som applikasjonen selv bruker. I tillegg skal jeg komme med noen tips og erfaringer som vi har gjort oss når vi har gjort denne jobben på vårt prosjekt.

<!-- Hva slags krav hadde vi til vår applikasjon - motivasjon for hvorfor vi ønsket å endre. -->
<!-- Zipping, tidsbasert rulling og ikke minst opprydning. -->

# Alternativer
Før jeg viser hvordan man går frem for å bytte ut log4j med logback, tenkte jeg å si litt om motivasjon og grunnen til at vi ønsket å gå fra log4j. Vi hadde en stor applikasjon som logget mye, og vi ønsket å få til en form for utvidet logging. Det vil si at vi ønsket at applikasjonen logget lenger tilbake, at loggfilene ble zippet etter hvert og ikke minst at applikasjonen selv skulle rydde opp og fjerne gamle loggfiler. Det siste punktet var spesielt viktig for oss da vi ikke ønsket å involvere drift i dette arbeidet.

Siden applikasjonen allerede hadde log4j som loggrammeverk, var det naturlig å se på hva vi kunne få til ved å bruke log4j eller versjoner av log4j hvor omskriving av applikasjonen ikke var nødvendig. Log4j hadde ikke mulighet til å oppfylle de kravene vi hadde, da log4j ikke kan zippe gamle loggfiler. Det eksisterer en ekstra "pakke" til log4j som heter log4j-extras og med denne pakken var applikasjonen i stand til å zippe gamle loggfiler, men klarte kunne å rydde opp etter seg. 

Vi ble derfor nødt til å se på andre alternativer og logback ble ganske raskt det beste alternativet. Logback hadde støtte for alle de kravene vi hadde, nemlig; tidsbasert rulling, zipping og ikke minst opprydning av loggfilene. Den endelige løsningen ble dermed: SLF4J over logback.

# SLF4J
SLF4J er et rammeverk for logging som er ment å fungere som en fasade eller en abstraksjon for andre loggrammeverk. Et viktig poeng er at slf4j ikke er en loggimplementasjon, x

 men fungerer istedet som en abstraksjon for andre loggrammeverk. Isteden programmerer man mot ett felles API, men at man enkelt kan bytte ut loggimplementasjon kun ved å bytte ut en avhengighet til en annen loggimplementasjonen.

Videre eksisterer det ulike bindings hvor hver binding korresponderer til en loggimplementasjon. Figuren nedenfor viser SLF4J, en binding for log4j og selve log4j-implementasjonen henger sammen.

![SLF4J over log4j](https://raw.github.com/henriwi/writings/master/img/slf4j-binding-log4j.png)

Ved et slikt oppsett vil selve loggingen bli seendes helt lik ut som hvis man brukte log4j direkte.

# Logback
Er en videreføring av log4j som tilbyr alt log4j tilbyr i tillegg til mye mer. Logback har såkalt native støtte for slf4j, som betyr at logback implementerer interfacet til slf4j som gjør at det ikke er nødvendig å bruke en egen binding som trengs for å få log4j til å fungere med slf4j.

Figuren nedenfor viser et typisk oppsett med logback. Legg spesielt merke til at det ikke er nødvendig med en egen binding mellom slf4j og logback. Dette er fordi logback implementerer slf4j sitt Logger-interface, og det er derfor ikke nødvendig å ha en egen binding som trengs når log4j brukes.

![SLF4J over logback](https://raw.github.com/henriwi/writings/master/img/slf4j-binding-logback.png)

Logback konfigureres i xml eller til og med groovy. Nedenfor vises en eksempelkonfigurasjon.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>

    <appender name="debugFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/app/logs/logback.log</file>

        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/app/logs/logback-%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>90</maxHistory>
        </rollingPolicy>

        <encoder>
            <pattern>%d [%-5p] %X{user} [%t] - %m -- %C%n</pattern>
        </encoder>
    </appender>

    <logger name="org.springframework" level="ERROR" />
</configuration>
```

# Omskriving til logback
Hva trengs for å skrive om applikasjonen til logback?

For å skrive om en applikasjon fra log4j sitt API til slf4j sitt API, trengs det en liten endring som vist nedenfor. I log4j vil en logger bli initalisert på følgende måte:

```java
Logger logger = Logger.getLogger(HelloWorld.class);
logger.info("Hello World");
```

I slf4j vil loggeren initialiseres på følgende måte:


```java
Logger logger = LoggerFactory.getLogger(HelloWorld.class);
logger.info("Hello World");
```

Bortsett fra under selve initaliseringen er API-et til slf4j ganske likt som log4j sitt API, og til "normalt" bruk vil man ikke merke spesielt stor forskjell.

