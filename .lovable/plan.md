# ROUT — vervolgplan: Developer Console + Profile Hub Studio

Beide briefs zijn naast de huidige code gelegd. Sommige delen bestaan al (een begin van privéconcepten, de toegangscontrole van de console). Het werk staat hieronder in volgorde van urgentie. Elk blok is apart testbaar.

## Blok A — Developer Console-toegang (eerst, blokkeert jullie nu)
1. **Eerst onderzoeken, dan pas fixen:** de console laat nu alleen binnen als het profiel `verified`, `is_paid` of `is_early_believer` heeft. Daarom kijk ik eerst in de echte gegevens van jullie root-accounts waarom ze geweigerd worden. Mogelijke oorzaken: de verificatie staat op een andere plek (een goedgekeurde handle, een verificatieaanvraag, een rol), of de sessie verwijst naar een alias in plaats van naar het hoofdaccount. Deze oorzaak is nog niet bevestigd.
2. Eén centrale servercheck "mag de console gebruiken". Die telt mee: geverifieerde persoon/bedrijf, actief Pro/Verified en een geclaimde root-handle. Gratis `/u/`-aliassen blijven geweigerd. Elke consolefunctie en API-sleutelactie gebruikt deze check.
3. Een nette vergrendelde pagina voor wie nog niet geverifieerd is: uitleg waarom verificatie nodig is, met een knop "Start verificatie". Geen kale rode foutmelding meer.
4. De wizard "Nieuwe app" (4 stappen) van begin tot eind controleren: opslaan, foutmeldingen en terugkeren.

## Blok B — Developer Console-inhoud
1. Tabbladen Schema en AI Prompts: kant-en-klare SQL- en Prisma-sjablonen voor gebruikers met `rout_id` (sub) en een tabel `rout_linked_accounts` (Google, GitHub, Apple, Meta, ...). Daarbij logica voor automatisch herkennen van bestaande accounts: koppelen op geverifieerd e-mailadres of `rout_id`, zonder dubbele accounts.
2. Het token en de UserInfo-endpoint geven de badges `is_verified_person`, `is_verified_business` en `is_influencer`, plus samengevatte cijfers (projecten, volgers). Dat gebeurt alleen bij een toegekende scope én als de privacy-instellingen van de gebruiker het toestaan. Dit wordt servermatig afgedwongen en getest.
3. Nieuw tabblad "Architectuur": uitleg over soevereine, non-custodial identiteit, Bring-Your-Own-Identity en koppelen van meerdere providers.

## Blok C — Studio fase 1: concepten en publiceren afronden
- Conceptopslag bestaat al gedeeltelijk. Ik controleer en vul aan: autosave gaat alleen naar het concept, met de knoppen **Publiceren** en **Wijzigingen verwerpen**. Revisies beschermen tegen conflicten tussen tabbladen. Publiceren gebeurt in één keer, en undo/redo werkt alleen op het concept.
- Een faviconset (tot 512×512) maken van het bestaande logo, zonder het logo te veranderen.

## Blok D — Studio fase 2: gelaagde Design Studio
- De algemene Custom-vergrendeling verdwijnt. Elk onderdeel (achtergrond, knoppen, typografie, avatar/header, status, footer, interactie) krijgt drie niveaus: Kiezen, Aanpassen en Geavanceerd.
- Nieuwe instellingen met grenzen op de server (verloop, licht, textuur, blur, schaduw, glow, letterafstand, ...). De bestaande "dode" instellingen gaan echt werken.
- Preview en publiek profiel gebruiken exact hetzelfde weergaveonderdeel.

## Blok E — Fase 3: bibliotheek
De 152 kaders blijven, aangevuld met gelaagde topontwerpen. Er komen minstens 100 decoraties in categorieën en meer bezoekseffecten, met regelbare intensiteit en duur. Rekening houden met "minder beweging" en alles netjes opruimen. De verouderde test (24 vs 152) wordt hersteld.

## Blok F — Fase 4: publiek profiel en analytics
Breedte en uitlijning, de vCard-knop, aanwezigheidsbubbel, 50 footerstijlen, sociale verificatie per kaart en echte klikmetingen per onderdeel.

## Blok G — Fase 5: trust en telefoon (apart, wacht deels op de gateway)
Trusttabellen, upload van adresbewijs met een inbox voor beheerders, termijnen (adres maximaal 1× per 6 maanden, herverificatie na 30 maanden) en een cron-taak. De telefoongateway volgt één vast contract. De Android-koppeling **wacht op het API-contract en de sleutels van jullie Android-gateway**. Telefoonnummers en codes worden alleen versleuteld of als hash bewaard.

## Blok H — Harding en afwerking
Tests op rechten, datalekken, rate limits en grenswaarden. Visuele controle op mobiel en desktop. Bijwerken van `ENVIRONMENT.md`, `.env.example` en `AGENTS.md`.

## Extra verbeteringen die ik voorstel
- Een auditlog voor consoleacties (apps aanmaken, secret roteren, instellingen wijzigen).
- Per-app rate limits en een "test login"-knop in de console.
- Bij Publiceren een overzicht tonen van de wijzigingen ten opzichte van live.
- Een `roadmap.md` die per blok de status bijhoudt.

## Technische details
- Neon blijft de enige database. Migraties zijn idempotente `db/NN_*.sql`-bestanden vanaf het volgende vrije nummer, toegepast via `MIGRATION_URL`. Er bestaat een dubbele map `db/db/`; die rechttrekken we.
- De consolecheck wordt één helper in `provider.server.ts` die `assertVerified` en `consoleAccess` vervangt, met unittests per regel: verified ja, alias nee, Pro ja, root-handle ja.
- Scope- en privacyfiltering van claims gebeurt in de userinfo/token-builder, met tests per claim.
- Uitvoering per blok: A en B eerst (één beurt), daarna C → H.
