# ROUT: app-wizard, telefoonverificatie, beveiliging en documentatie

Belangrijk vooraf: ROUT gebruikt Neon (niet Lovable Cloud) en Better Auth. Er bestaat al een wizard (`/console/apps/new`) en een tabel `oauth_clients` (db/42) met `client_id`, `secret_hash`, `name`, `homepage_url`, `redirect_uris[]`, `owner_user_id` en tijdstempels. We bouwen daarop verder in plaats van een tweede tabel te maken, want twee tabellen zouden de bestaande login via ROUT breken.

## 1. App-wizard (afronden)

Nieuwe stappenvolgorde in de bestaande wizard:

1. Naam, beschrijving en website (https verplicht).
2. Terug-adressen: meerdere toevoegen of verwijderen, met controle per regel (https verplicht, `http://localhost` alleen voor testen, geen `*`, geen fragment `#`, maximaal 10).
3. Sleutels aanmaken: de server maakt een willekeurige `client_id` en een geheim van 32 bytes aan. Alleen een hash (SHA-256 met pepper) gaat naar de database.
4. Overzicht: client-id, het geheim **één keer** zichtbaar met kopieerknop en waarschuwing, en het OIDC Discovery-adres `https://rout.be/.well-known/openid-configuration`.

Database: idempotente migratie `db/62_oauth_client_description.sql` voegt de kolom `description` toe aan `oauth_clients`. Geen nieuwe tabel.
Beveiliging: aanmaken, lezen en geheim vernieuwen alleen via server functions met een sessiecontrole. Er wordt altijd gefilterd op `owner_user_id = sessie-gebruiker`. Het geheim zelf wordt nooit opnieuw teruggegeven, enkel bij aanmaken of vernieuwen. Er geldt ook een limiet op het aantal nieuwe apps per uur.

## 2. Telefoonverificatie (voorbereid, nog niet actief)

- Een neutraal "SMS-afzender"-contract met drie aansluitpunten: Android-gateway (hoofdkeuze, volgens de bestaande projectregel), Twilio en Brevo. Zolang er geen sleutels zijn, meldt het onderdeel "niet geconfigureerd" en wordt er niets verstuurd.
- Een server-route om een code aan te vragen en een server-route om ze te controleren. De code bestaat uit 6 cijfers, wordt alleen gehasht bewaard, vervalt na 10 minuten en laat maximaal 5 pogingen toe. Er gelden limieten per nummer, per gebruiker en per IP.
- Database: migratie `db/63_phone_verification.sql`. De kolommen `phone_number_enc` (versleuteld), `phone_last4` (voor de gemaskeerde weergave) en `phone_verified` + `phone_verified_at` komen in de profieltabel, omdat ROUT geen aparte `users`-tabel voor profielgegevens gebruikt. Daarnaast komt er een tabel `phone_otp_codes`. Het telefoonnummer wordt alleen versleuteld bewaard en nooit leesbaar.
- Een strak invulscherm in de instellingen: landcode + nummer, knop "Code sturen", invulvak voor 6 cijfers met aftelteller, en daarna een groene status "Geverifieerd" met het gemaskeerde nummer (•••• 1234).
- In `.env.example` en `ENVIRONMENT.md` komen deze variabelen: `SMS_PROVIDER`, `SMS_API_KEY`, `SMS_SENDER_ID`, `SMS_GATEWAY_URL` en `PHONE_ENCRYPTION_KEY`.

## 3. Beveiliging en limieten

- Een gedeelde limietfunctie voor gevoelige routes: tokenuitwisseling (`/api/public/oauth/token`), aanmelden en wachtwoordherstel (`/api/auth/*`), verificatieaanvragen en SMS-codes. Te veel pogingen geven foutcode 429 met `Retry-After`. Die limiet wordt bijgehouden in de database (Neon), zodat ze over alle servers heen werkt en niet per server opnieuw begint. Migratie `db/64_rate_limits.sql`.
- Strengere beveiligingsheaders:
  - CSP met `frame-ancestors` (Lovable-editor toegestaan, verder alleen rout.be)
  - `Strict-Transport-Security`
  - `Cross-Origin-Opener-Policy`
  - CORS alleen op de publieke OIDC-adressen (discovery, jwks, token, userinfo), nooit op server functions
- Een controle dat alle sessiecookies HttpOnly, Secure en SameSite=Lax zijn.

## 4. Documentatie voor ontwikkelaars

Een nieuwe openbare pagina `/developers`, die de huidige `/api` vervangt (nu een doorverwijzing). Ze is eenvoudig en minimalistisch, met zijnavigatie en eigen titel en beschrijving voor zoekmachines.

- **Login met ROUT:** discovery-adres, stappen voor de code-flow met PKCE (S256), voorbeeldcode, scopes en de bijbehorende gegevens (claims).
- **Hoe verificaties werken:** Google/YouTube, GitHub, GitLab, Bluesky/AT Protocol en Mastodon/fediverse. Per dienst: wat bewezen wordt, en hoe (OAuth, link in de bio, DID).
- **API-overzicht:** alle endpoints, foutcodes en limieten.

Een link naar deze pagina komt in de console en in de voettekst. "delplanche.cloud" is een apart domein; de pagina leeft op rout.be en kan daar later naartoe verwezen worden.

## Tests

Er komen kleine tests voor:
- de controle van terug-adressen
- het hashen van het geheim en de eenmalige weergave
- de vervaltijd en het maximum aantal pogingen van de SMS-code
- de limietfunctie (429 na de grens)
- de headers

## Wat open blijft

- Echte SMS versturen: wacht op de keuze van de dienst en de sleutels.
- De wizard volledig testen met de database: wacht op `DATABASE_URL`.

## Technische details

- Bestanden: `src/lib/oauth/client-registration.{server,functions}.ts`, `src/lib/phone/{sms-provider,phone-otp}.server.ts` + `phone.functions.ts`, `src/components/settings/PhoneVerification.tsx`, `src/lib/rate-limit-db.server.ts`, `src/start.ts` (headers), `src/routes/developers.tsx`.
- Geheime hash: `sha256(pepper + secret)`, vergelijken in constante tijd. De pepper komt uit de bestaande auth-secret-afleiding.
- Telefoonversleuteling: AES-GCM via WebCrypto (werkt op workerd en Vercel).
- Regels in `AGENTS.md` toevoegen voor de SMS-contractlaag en de limieten in de database.
