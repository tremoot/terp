# Terp — instructies voor AI-coding-agents

> Dit bestand is de enige bron van instructies voor alle coding-agents (Claude Code, Codex, Cursor, Copilot, …). `CLAUDE.md` verwijst hiernaar. Werk dit bestand bij, niet `CLAUDE.md`.
> Communiceer met mij in het Nederlands. Code, identifiers, API, commentaar en commit-berichten in het Engels.

## 1. Wat is Terp

Terp is een **sovereign cloud-platform** dat bedrijven op hun eigen servers of in hun eigen datacenter installeren, zodat ze daar een "cloud" van maken. Het platform wordt gebouwd in **.NET** en hergebruikt zoveel mogelijk bestaande open source software.

- **Eigenaar en beheerder: Tremoot.** Tremoot heeft het volledige auteursrecht op Terp.
- Terp is **source-available, géén open source** (zie §5.1). Noem Terp nergens "open source" (code, docs, README, website-teksten).

- Wij zijn zelf de eerste klant. Onze huidige apps draaien op Azure (AKS, Functions, App Service, Azure SQL, Blob Storage, Service Bus, Key Vault, Application Gateway/Front Door, Azure Firewall/NSG's, VPN Gateway, Private Endpoints/Private DNS).
- Hoofdfocus: gateways, netwerken, firewalls, mTLS, secrets en RBAC.
- Doelgroepen op termijn: IT-afdelingen, MSP's/hosters, overheid/zorg/onderwijs en softwareteams.
- Repository en CI staan op **GitHub**: `github.com/tremoot/terp`, pipelines via GitHub Actions.

## 2. Werkwijze voor agents

1. **Eerst plannen, dan bouwen.** Laat bij elke grotere taak eerst een kort plan zien en wacht op akkoord.
2. **Kleine, werkende stappen.** Elke stap moet compileren en de tests moeten slagen.
3. **Vraag bij twijfel.** Neem geen architectuurbesluiten die niet in dit bestand staan; stel ze voor en wacht.
4. **Geen nieuwe dependencies zonder akkoord**, en nooit met een licentie buiten het licentiebeleid (zie §5).
5. **Tests horen bij de wijziging**, niet erna.
6. **Werk dit bestand bij** als we samen een nieuw besluit nemen (sectie "Besluitenlog").
7. **Git:** werk op een feature-branch, kleine commits met duidelijke Engelse berichten. Push of open een PR alleen als daarom gevraagd is.

## 3. Architectuurprincipes

- **Platformkern + modules.** Eerst de kern, daarna modules: Config → Secrets → Certificates → Network → Apps → Database (volgorde van afhankelijkheden).
- **Container-first.** Onder water draait alles op Kubernetes. Klanten zijn **verplicht** hun applicaties als container (OCI-image) te deployen. Klanten krijgen nooit directe toegang tot Kubernetes; alles loopt via de Terp-API.
- **API-first.** Alles wat de portal kan, kan via de API. Portal, CLI en Terraform-provider gebruiken dezelfde API.
- **Declaratief.** Resources hebben een `spec` (gewenste staat) en een `status` (werkelijke staat). Reconcilers brengen de werkelijkheid in lijn met de spec.
- **Secure by default.** mTLS standaard aan, secrets nooit als platte tekst, nieuwe identiteiten hebben standaard nul rechten (least privilege).
- **Modulaire monoliet om mee te beginnen.** De API draait als één proces met elke module als eigen assembly. Reconcilers draaien als aparte workers. Modules raken nooit elkaars tabellen aan.
- **Eigen terminologie.** We gebruiken bewust géén Azure-termen in het product (zie §6).

## 4. Technologiestack

| Onderdeel | Keuze | Licentie |
|---|---|---|
| Runtime | .NET 10 (LTS), C# | MIT |
| Lokale orkestratie | .NET Aspire (AppHost) | MIT |
| Platformdatabase | PostgreSQL via Npgsql + EF Core | PostgreSQL / MIT |
| Authenticatie | Keycloak (OIDC) | Apache 2.0 |
| Autorisatie | OpenFGA (vanaf fase 2) | Apache 2.0 |
| Messaging | NATS (vanaf fase 2) | Apache 2.0 |
| Observability | OpenTelemetry | Apache 2.0 |
| API-contract | TypeSpec → OpenAPI (vanaf fase 2) | MIT |
| Clients | Kiota | MIT |
| Tests | xUnit, Testcontainers | Apache 2.0 / MIT |

Latere modules (ter context, nog niet bouwen): Kubernetes (Talos Linux), Kube-OVN, Envoy Gateway + Coraza WAF, Istio ambient, cert-manager, OpenBao, Harbor, Trivy, Cosign, Kyverno, KEDA, CloudNativePG, Rook-Ceph, PowerDNS, OpenCost.

**De platformdatabase is altijd PostgreSQL en is niet uitwisselbaar.** De Database-module voor klanten ondersteunt later meerdere engines; dat staat hier volledig los van.

## 5. Licenties

### 5.1 Licentie van Terp zelf

- De broncode is openbaar, zodat klanten Terp kunnen inzien en auditen.
- **Gratis voor iedereen:** testen, ontwikkelen en evalueren (niet-productiegebruik).
- **Productiegebruik vereist altijd een licentie van Tremoot**, voor iedereen.
- **Non-profits** krijgen een gratis productielicentie (ze moeten die wel aanvragen).
- Een versie wordt **nooit** automatisch open source (dus geen BSL-achtige omzetting).
- Licentietekst: `LICENSE` ("Terp License"). De huidige versie is een **concept** en moet nog door een IT-jurist worden gecontroleerd.
- **Forken is niet toegestaan** zonder schriftelijke toestemming. Een fork op GitHub mag alleen om de code te bekijken en pull requests in te dienen (GitHub-voorwaarden staan dat forken altijd toe bij publieke repositories).
- Bronbestanden beginnen met de header:
  ```csharp
  // Copyright (c) Tremoot. All rights reserved.
  // Licensed under the Terp License. See LICENSE in the repository root.
  ```
- **Uitzondering:** API-clients, CLI en Terraform-provider worden Apache 2.0, zodat klanten er vrij op kunnen bouwen.
- Externe bijdragen alleen met een getekende CLA.
- Licentiecontrole later via ondertekende licentiebestanden die **offline** worden geverifieerd (geen phone-home). Niet in fase 1.

### 5.2 Licenties van dependencies

- **Toegestaan:** MIT, Apache 2.0, BSD, MPL, PostgreSQL-licentie.
- **Alleen als apart proces (niet linken):** LGPL, GPL.
- **Niet zonder expliciet akkoord:** AGPL, SSPL, BSL, en elke commerciële licentie.
- Licentieteksten en NOTICE-bestanden van dependencies meeleveren bij distributie (`THIRD-PARTY-NOTICES`).
- **Niet gebruiken** (inmiddels commercieel): MediatR, AutoMapper, MassTransit, Duende IdentityServer, FluentAssertions v8+. Alternatieven: eigen dispatching of Wolverine, handmatige mapping, OpenIddict, Shouldly.

## 6. Terminologie (ubiquitous language)

> Werktermen, nog te bevestigen. Gebruik ze consequent in code, database, API en documentatie.

| Term | Betekenis |
|---|---|
| **Operator** | Wie het platform draait (wij, of een MSP) |
| **Organization** | Identiteitsgrens: gebruikers, groepen, service identities |
| **Space** | Grens voor facturatie en quota (bijv. "productie", "test") |
| **Bundle** | Groep resources die samen leven en samen verdwijnen |
| **Resource** | Een beheerd object van een module |
| **Role** | Een set rechten |
| **Grant** | Identiteit + role + scope |
| **Service identity** | Niet-menselijke identiteit (pipelines, apps) |
| **Trust** | Vertrouwensrelatie met een externe OIDC-uitgever (bijv. GitHub Actions, Azure DevOps) |
| **Invite** | Uitnodiging met vervaldatum |

**Vermijd** in namen: workspace, module (extern), namespace, cluster, node, stack, project, realm, domain, zone, en "group" voor iets anders dan gebruikersgroepen. Gebruik geen merkgebonden Azure-termen (VNet, NSG, Key Vault, App Service).

Naamgeving in code: namespaces `Terp.*`, Terraform-resources `terp_*`, CLI-commando `terp`.

## 7. Solution-structuur

```
src/
  Terp.AppHost/                 # .NET Aspire: start Postgres, Keycloak, API, workers
  Terp.ServiceDefaults/         # OpenTelemetry, health checks, resilience
  Terp.Contracts/               # Gedeelde DTO's en het resource-envelope
  Terp.Core/                    # Resource-engine, modulecontract, persistence, audit
  Terp.Identity/                # Organizations, spaces, bundles, users, roles, grants
  Terp.Api/                     # ASP.NET Core host; laadt alle modules
  Terp.Modules.Config/          # Eerste module (API-deel + reconciler)
  Terp.Workers.Config/          # Worker-host voor de Config-reconciler
tests/
  Terp.Core.Tests/
  Terp.Identity.Tests/
  Terp.Modules.Config.Tests/
  Terp.Api.IntegrationTests/    # Testcontainers met PostgreSQL
Directory.Build.props
Directory.Packages.props        # Central Package Management
.github/workflows/ci.yml        # GitHub Actions: build + test
AGENTS.md                       # Instructies voor coding-agents (dit bestand)
CLAUDE.md                       # Verwijst naar AGENTS.md
LICENSE                         # Terp License
README.md
```

Codeerregels: nullable aan, `TreatWarningsAsErrors`, file-scoped namespaces, central package management, async overal met `CancellationToken`, geen statische state.

## 8. Resource-model en API-regels

Elke resource heeft dezelfde envelope:

```json
{
  "id": "/orgs/acme/spaces/prod/bundles/web/resources/config/setting/feature-flags",
  "name": "feature-flags",
  "type": "config/setting",
  "metadata": {
    "generation": 3,
    "etag": "\"a1b2c3\"",
    "labels": { "team": "web" },
    "createdAt": "2026-11-01T10:00:00Z",
    "createdBy": "user:selim"
  },
  "spec": { },
  "status": {
    "phase": "Pending | Ready | Failed | Deleting",
    "observedGeneration": 3,
    "conditions": [ ]
  }
}
```

Pad (voorstel): `/orgs/{org}/spaces/{space}/bundles/{bundle}/resources/{module}/{type}/{name}`

API-regels (nodig voor Terraform):

- **PUT is idempotent** en maakt aan of werkt bij. Dezelfde aanvraag twee keer geeft hetzelfde resultaat.
- **GET geeft de volledige staat terug**, behalve secrets (write-only).
- **ETags** via `If-Match` voor gelijktijdige wijzigingen; mismatch geeft `412`.
- **Asynchroon:** PUT geeft direct de resource terug met `status.phase = Pending`; de reconciler zet hem op `Ready` of `Failed`. Elke spec-wijziging verhoogt `generation`.
- **Versiebeheer** via query-parameter `?api-version=2026-11-01`.
- **Foutmodel:** RFC 9457 Problem Details.
- **Stabiele ID's** en ondersteuning voor import.
- Elke mutatie schrijft een regel in de **audit-log** (wie, wat, wanneer, scope, resultaat). De audit-log is append-only.

## 9. Modulecontract

```csharp
public interface IPlatformModule
{
    string Name { get; }                 // "config"
    void Register(IModuleBuilder builder);
}

// Voorbeeld in de Config-module:
builder.AddResourceType<SettingSpec, SettingStatus>("setting")
       .WithPermissions("read", "write", "delete")
       .WithValidator<SettingSpecValidator>()
       .WithReconciler<SettingReconciler>();
```

- De **kern** regelt routing, validatie-pipeline, persistentie, ETags, generation, autorisatie, quota en audit.
- Een **module** levert alleen: resourcetypes (spec/status), validators, rechten, reconcilers en (later) verbruiksmeting.
- Persistentie in de kern: één tabel `resources` met `spec` en `status` als JSONB. Module-specifieke tabellen staan in een eigen databaseschema per module.
- Reconcilers krijgen wijzigingen via PostgreSQL `LISTEN/NOTIFY` (fase 1) en draaien met leader election via advisory locks.

## 10. Security by default

- Elk endpoint vereist authenticatie, behalve health checks.
- Autorisatie altijd via één abstractie (`ITerpAuthorizer`) in de kern; nooit losse checks in modules.
- Nieuwe identiteiten hebben **geen** rechten. De eerste gebruiker van een organization wordt Owner; dat is het enige automatische recht.
- Secrets nooit loggen, nooit teruggeven in GET-responses, nooit in exceptions.
- Tenant-isolatie: elke query is gescoped op organization. Voeg integratietests toe die aantonen dat organization A nooit data van organization B ziet.

## 11. Eerste opdracht — Fase 1: walking skeleton

**Doel:** een werkend skelet van kern + identity + Config-module, lokaal te starten met één commando.

1. Maak de solution-structuur uit §7 aan, met `Directory.Build.props` en `Directory.Packages.props`.
2. **AppHost (Aspire)** start PostgreSQL, Keycloak, `Terp.Api` en `Terp.Workers.Config`.
3. **Terp.Core:** resource-envelope, `IPlatformModule`, `IModuleBuilder`, generieke CRUD-endpoints, EF Core-persistentie (JSONB), ETags, generation, audit-log, `ITerpAuthorizer` met een eenvoudige implementatie op basis van grants in de database (OpenFGA volgt in fase 2).
4. **Terp.Identity:** CRUD voor organizations, spaces en bundles; roles en grants; JWT-validatie tegen Keycloak. Eerste gebruiker van een organization wordt Owner.
5. **Terp.Modules.Config:** resourcetype `setting` met een spec van key/value-paren; validator; reconciler die de status op `Ready` zet.
6. **Tests:** unit-tests voor de kern en de module; integratietests met Testcontainers voor CRUD, ETag-conflict (`412`), idempotente PUT, tenant-isolatie en audit-log.
7. **`.github/workflows/ci.yml`:** build + test (GitHub Actions).

**Expliciet nog niet in fase 1:** NATS, OpenFGA, TypeSpec, Kubernetes, portal, CLI, Terraform-provider.

**Acceptatiecriteria**

- `dotnet run --project src/Terp.AppHost` start alles zonder handwerk.
- Een gebruiker kan via de API een organization, space en bundle aanmaken en daarin een config-setting zetten, ophalen, wijzigen en verwijderen.
- Een tweede PUT met dezelfde body verandert niets (geen nieuwe generation).
- Een PUT met een verouderde ETag geeft `412`.
- Een gebruiker zonder grant krijgt `403`.
- Alle tests slagen in de pipeline.

## 12. Openstaande beslissingen

- Worden Windows-containers ondersteund (voor .NET Framework-apps), of alleen Linux?
- Definitieve terminologie (§6) en het API-pad (§8).
- Mag Terp zelf client-certificaten voor partners uitgeven, of brengen klanten altijd hun eigen CA mee?
- Licentie: definities van "non-profit" en "productiegebruik" staan als concept in `LICENSE`; nog te laten bevestigen door een IT-jurist.
- Licentie: voorwaarden voor MSP's/hosters die Terp aan hun eigen klanten aanbieden.

## 13. Besluitenlog

| Datum | Besluit |
|---|---|
| 2026-10 | Eigen platform in .NET, niet gebaseerd op Apache CloudStack |
| 2026-10 | Alles draait op containers (Kubernetes); klanten moeten containers gebruiken |
| 2026-10 | API's en Terraform-ondersteuning zijn vereist |
| 2026-10 | Eigen terminologie, geen Azure-termen |
| 2026-10 | Eerst platformkern (account, tenants, gebruikersbeheer), daarna modules |
| 2026-10 | Meerdere database-engines voor klanten, los van de platformdatabase |
| 2026-10 | Naam van het platform: Terp |
| 2026-10 | Tremoot is eigenaar en beheerder van Terp |
| 2026-10 | Terp is source-available, geen open source; productiegebruik vereist altijd een licentie |
| 2026-10 | Testen en evaluatie gratis voor iedereen; non-profits gratis productielicentie |
| 2026-10 | Versies worden nooit automatisch open source |
| 2026-10 | GitHub (`tremoot/terp`) voor repository en CI, niet Azure DevOps |
| 2026-10 | `AGENTS.md` is de enige bron van agent-instructies; `CLAUDE.md` verwijst ernaar |
| 2026-10 | Forken alleen met schriftelijke toestemming van Tremoot (concept-LICENSE) |
| 2026-10 | Non-profit (concept): ANBI of buitenlands equivalent; overheid, zorg, onderwijs/onderzoek en woningcorporaties uitgesloten |
| 2026-10 | Bevoegde rechter: Amsterdam (gelijk aan de algemene voorwaarden van Tremoot B.V.) |
