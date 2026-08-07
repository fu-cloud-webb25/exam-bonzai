# Gruppexamination: Bonz.ai Booking API

## Bakgrund

Bonz.ai är ett modernt hotell som strävar efter att ligga i framkant när det gäller användningen av teknik för att förbättra kundupplevelsen.

Ni har fått i uppdrag att utveckla hotellets nya **boknings-API**.

Bonz.ai har valt att använda en **serverless arkitektur i AWS**, vilket innebär att applikationen kan skalas efter behov utan att hotellet behöver hantera egna servrar.

API:t ska hantera:

* användare
* inloggning
* hotellrum
* tillgänglighet
* bokningar

För persistent lagring ska ni använda **Amazon DynamoDB**.

---

# Uppgiften

Ni ska i grupp utveckla och driftsätta ett komplett REST API för Bonz.ai.

Det färdiga systemet kommer i stora drag bestå av:

```text
Client
   │
   ▼
API Gateway
   │
   ▼
Lambda
   │
   ▼
DynamoDB
```

Ni väljer själva hur ni strukturerar era Lambda-funktioner, er kod och er DynamoDB-databas.

---

# Grupparbete

En gruppmedlem skapar ett GitHub-repository och bjuder in resterande gruppmedlemmar som collaborators.

Skapa ett GitHub Project där ni lägger in projektets User Stories.

Bryt sedan tillsammans ner era User Stories till mindre och mer tekniska tasks.

Arbetet ska ske genom:

```text
main
 │
 ├── feature/register
 ├── feature/login
 ├── feature/create-booking
 ├── feature/get-bookings
 └── ...
```

Använd **Pull Requests och Code Review** innan kod mergas till `main`.

Alla gruppmedlemmar ska bidra aktivt till repositoryt.

---

# Affärsregler

## Rum

Bonz.ai har totalt **20 hotellrum**.

Hotellet erbjuder tre olika rumstyper:

| Rumstyp   | Max antal gäster | Pris / natt |
| --------- | ---------------: | ----------: |
| Enkelrum  |                1 |      500 kr |
| Dubbelrum |                2 |     1000 kr |
| Svit      |                3 |     1500 kr |

Ni bestämmer själva hur hotellets 20 rum fördelas mellan de tre rumstyperna.

Fördelningen ska dokumenteras i projektets README.

Exempel:

```text
10 enkelrum
7 dubbelrum
3 sviter
```

---

# Bokningar

En bokning ska innehålla:

* användare
* ett eller flera rum
* antal gäster
* incheckningsdatum
* utcheckningsdatum
* totalpris

Exempel:

```json
{
  "checkIn": "2026-09-20",
  "checkOut": "2026-09-23",
  "guests": 3,
  "rooms": [
    {
      "roomId": "ROOM:12",
      "type": "double"
    },
    {
      "roomId": "ROOM:04",
      "type": "single"
    }
  ]
}
```

API:t ansvarar för att beräkna bokningens totalpris.

---

# Regler för antal gäster

De bokade rummens sammanlagda kapacitet måste kunna ta emot antalet gäster.

Exempel:

### 1 gäst

Kan exempelvis boka:

```text
1 × enkelrum
```

### 2 gäster

Kan exempelvis boka:

```text
1 × dubbelrum
```

eller:

```text
2 × enkelrum
```

### 3 gäster

Kan exempelvis boka:

```text
1 × svit
```

eller:

```text
1 × dubbelrum
+
1 × enkelrum
```

API:t ska neka bokningar där de valda rummens kapacitet är lägre än antalet gäster.

---

# Datum och tillgänglighet

Alla bokningar ska innehålla:

```text
checkIn
checkOut
```

Exempel:

```json
{
  "checkIn": "2026-09-20",
  "checkOut": "2026-09-23"
}
```

`checkOut` måste inträffa efter `checkIn`.

Ett rum får **inte vara dubbelbokat**.

Om exempelvis:

```text
ROOM:12
```

är bokat:

```text
20 september → 23 september
```

ska det inte kunna bokas av en annan användare under en period som överlappar dessa datum.

API:t behöver därför kunna avgöra vilka rum som är tillgängliga under en viss period.

---

# Pris

API:t ska själv beräkna bokningens totalpris.

Exempel:

```text
1 dubbelrum
1000 kr / natt

3 nätter
```

ger:

```text
3000 kr
```

Om flera rum bokas ska samtliga rum räknas med.

Klienten ska alltså **inte själv kunna bestämma bokningens totalpris**.

---

# Registrering

En användare ska kunna skapa ett konto.

Exempel på endpoint:

```http
POST /auth/register
```

Request body kan exempelvis innehålla:

```json
{
  "username": "anna",
  "email": "anna@example.com",
  "password": "mySecretPassword"
}
```

Lösenordet får **aldrig lagras i klartext** i DynamoDB.

Använd exempelvis:

```text
bcrypt
```

eller motsvarande teknik för att hasha lösenordet innan användaren sparas.

---

# Inloggning

En registrerad användare ska kunna logga in.

Exempel:

```http
POST /auth/login
```

Vid korrekt email/användarnamn och lösenord ska API:t returnera en:

```text
JWT
```

Exempel:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

Denna token används sedan för att identifiera användaren vid skyddade API-anrop.

---

# Authorization

Endpoints som hanterar en användares bokningar ska vara skyddade.

Klienten skickar sin JWT via:

```http
Authorization: Bearer <token>
```

En användare ska endast kunna:

* hämta sina egna bokningar
* ändra sina egna bokningar
* avboka sina egna bokningar

En användare ska alltså inte kunna ange en annan användares ID och därigenom komma åt dennes bokningar.

---

# API

Ni bestämmer själva exakt hur era endpoints ska struktureras.

API:t måste däremot erbjuda funktionalitet för följande.

## Auth

```text
Registrera användare
Logga in användare
```

## Rum

```text
Hämta alla rum
Hämta tillgängliga rum för en viss period
```

## Bokningar

```text
Skapa en bokning
Hämta användarens bokningar
Hämta en specifik bokning
Ändra en bokning
Avboka en bokning
```

En möjlig struktur skulle exempelvis kunna vara:

| Method   | Endpoint           | Beskrivning                 |
| -------- | ------------------ | --------------------------- |
| `POST`   | `/auth/register`   | Registrera användare        |
| `POST`   | `/auth/login`      | Logga in                    |
| `GET`    | `/rooms`           | Hämta rum                   |
| `GET`    | `/rooms/available` | Hämta tillgängliga rum      |
| `POST`   | `/bookings`        | Skapa bokning               |
| `GET`    | `/bookings`        | Hämta användarens bokningar |
| `GET`    | `/bookings/{id}`   | Hämta specifik bokning      |
| `PUT`    | `/bookings/{id}`   | Ändra bokning               |
| `DELETE` | `/bookings/{id}`   | Avboka bokning              |

> Tabellen ovan är ett förslag. Ni får själva välja en annan endpoint-struktur så länge all efterfrågad funktionalitet finns.

---

# DynamoDB

All persistent data ska lagras i **DynamoDB**.

Ni väljer själva hur databasen ska designas.

Ni kan exempelvis använda:

* en tabell
* flera tabeller
* Partition Keys
* Sort Keys
* sammansatta nycklar
* Secondary Indexes

Databasdesignen ska utgå från hur applikationen behöver läsa och skriva sin data.

---

# Access Patterns

Innan ni börjar implementera databasen ska ni identifiera vilka **Access Patterns** systemet behöver.

Exempel:

```text
Hämta användare via email

Hämta alla bokningar för inloggad användare

Hämta en specifik bokning

Hämta alla bokningar för ett specifikt rum

Kontrollera om ett rum är bokat under en viss period
```

Dokumentera era viktigaste Access Patterns i README.

Beskriv även kort hur er valda DynamoDB-design gör det möjligt att lösa dem.

---

# Tekniska krav

Projektet ska använda:

* Serverless Framework
* AWS Lambda
* API Gateway
* DynamoDB
* Node.js
* Git och GitHub

Serverless Framework ska användas för att deploya applikationen.

Er AWS-infrastruktur ska i så stor utsträckning som möjligt beskrivas i:

```text
serverless.yml
```

---

# Validering

All data som kommer från klienten ska valideras innan den används.

Exempel på saker som ska kontrolleras:

* obligatoriska fält finns
* email har ett giltigt format
* lösenord uppfyller era krav
* antal gäster är ett giltigt nummer
* rum existerar
* rumskapaciteten räcker
* datum är giltiga
* `checkOut` inträffar efter `checkIn`

Ni väljer själva vilket valideringsverktyg ni använder.

---

# Felhantering

API:t ska returnera JSON tillsammans med relevanta HTTP-statuskoder.

Exempel:

### 400 Bad Request

Requesten innehåller felaktig data.

```json
{
  "message": "Invalid request body."
}
```

### 401 Unauthorized

Användaren är inte inloggad eller skickar en ogiltig token.

```json
{
  "message": "Unauthorized."
}
```

### 403 Forbidden

Användaren försöker utföra en operation som hen inte har behörighet till.

```json
{
  "message": "Forbidden."
}
```

### 404 Not Found

Resursen som efterfrågas finns inte.

```json
{
  "message": "Booking not found."
}
```

### 409 Conflict

En bokning kan inte genomföras på grund av systemets nuvarande tillstånd.

Exempelvis:

```json
{
  "message": "One or more rooms are unavailable."
}
```

### 500 Internal Server Error

Ett oväntat fel har inträffat på servern.

---

# Dokumentation

Projektets README ska innehålla dokumentation för API:t.

Dokumentationen ska minst innehålla:

* API:ts base URL
* samtliga endpoints
* HTTP-metod för varje endpoint
* vilka endpoints som kräver authentication
* förväntad request body
* eventuella path parameters
* eventuella query parameters
* exempel på responses
* relevanta felmeddelanden och statuskoder

Det ska vara möjligt för en annan utvecklare att förstå hur ert API används genom att endast läsa dokumentationen.

---

# DynamoDB-dokumentation

README ska även innehålla en kort beskrivning av er databasdesign.

Dokumentera:

* vilka tabeller ni använder
* Partition Keys
* Sort Keys
* eventuella Secondary Indexes
* viktiga Access Patterns

Ni behöver inte skriva en lång rapport.

Syftet är att ni ska kunna förklara **varför databasen är designad som den är**.

---

# GitHub och grupparbete

Arbetet ska ske gemensamt via GitHub.

Ni ska använda:

```text
feature branch
      ↓
Pull Request
      ↓
Code Review
      ↓
merge
      ↓
main
```

Alla gruppmedlemmar ska ha bidragit med kod till projektet.

Commits och Pull Requests kommer kunna användas för att följa gruppens arbetsprocess.

---

# Krav för Godkänt

För att examinationen ska bedömas som **Godkänd** ska följande krav vara uppfyllda:

* API:t är deployat till AWS och går att anropa.
* Serverless Framework används för deployment.
* API Gateway används för API:ts endpoints.
* AWS Lambda används för applikationens funktionalitet.
* DynamoDB används för persistent lagring.
* Användare kan registrera sig.
* Lösenord lagras hashade och inte i klartext.
* Registrerade användare kan logga in.
* En lyckad inloggning ger användaren en JWT.
* Skyddade endpoints kräver en giltig JWT.
* En användare kan endast hantera sina egna bokningar.
* Hotellets rum och rumstyper hanteras enligt kravspecifikationen.
* En användare kan söka efter tillgängliga rum.
* En användare kan skapa en bokning.
* En användare kan hämta sina bokningar.
* En användare kan ändra en bokning.
* En användare kan avboka en bokning.
* Ett rum kan inte dubbelbokas under överlappande datum.
* Antalet gäster får inte överstiga de bokade rummens sammanlagda kapacitet.
* Totalpriset beräknas av API:t.
* Inkommande data valideras.
* Relevanta fel hanteras med lämpliga HTTP-statuskoder.
* API:t är dokumenterat i README.
* DynamoDB-design och Access Patterns är kort dokumenterade i README.
* Alla gruppmedlemmar har deltagit aktivt i utvecklingen.

---

# Inlämning

Inlämning sker på **Moodle**.

Samtliga gruppmedlemmar ska lämna in en länk till gruppens GitHub-repository.

Repositoryt ska innehålla:

```text
Kod
serverless.yml
README.md
```

Kontrollera innan inlämning att den API-URL som finns dokumenterad i README fortfarande fungerar.

## Deadline

**Fredagen 18/9 kl. 23:59**

Alla gruppmedlemmar behöver göra en egen inlämning, även om ni lämnar in samma repository.
