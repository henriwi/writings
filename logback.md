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
SLF4J er kun et api og ingen egen loggimplementasjon, men fungerer istedet som en abstraksjon for andre loggrammeverk. Isteden programmerer man mot ett felles API, men at man enkelt kan bytte ut loggimplementasjon kun ved å bytte ut en avhengighet til en annen loggimplementasjonen.

Videre eksisterer det ulike bindings hvor hver binding korresponderer til en loggimplementasjon. Figuren nedenfor viser SLF4J, en binding for log4j og selve log4j-implementasjonen henger sammen.

!(https://raw.github.com/henriwi/writings/master/img/slf4j-binding-log4j.png

Ved et slikt oppsett vil selve loggingen bli seendes helt lik ut som hvis man brukte log4j direkte.

# Logback
Er en videreføring av log4j som tilbyr alt log4j tilbyr i tillegg til mye mer. Logback har såkalt native støtte for slf4j, som betyr at logback implementerer interfacet til slf4j som gjør at det ikke er nødvendig å bruke en egen binding som trengs for å få log4j til å fungere med slf4j.

Figuren nedenfor viser et typisk oppsett med logback. Legg spesielt merke til at det ikke er nødvendig med en egen binding mellom slf4j og logback. Dette er fordi logback implementerer slf4j sitt Logger-interface, og det er derfor ikke nødvendig å ha en egen binding som trengs når log4j brukes.

!(https://raw.github.com/henriwi/writings/master/img/slf4j-binding-logback.png)

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
Hva trengs for å skrive om applikasjonen til logback?.