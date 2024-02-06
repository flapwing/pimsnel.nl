---
title: Prometheus Alert Rules schrijven met Nickel
draft: false
subtitle: Eenvoudige tutorial om de nieuwe taal Nickel te leren kennen
pubdate: '2023-06-20'
type: artikel
image: imgs/nickel-golden-tinge.jpg
categories:
  - nickel
  - tutorial
  - nederlands
recept:
  tijd:
    bereidingstijd: ''
    werktijd: ''
  ingrediënten: []
  calculator:
    plusfactor: 0
    start: 0
    end: 0
    meervoud: ''
    enkelvoud: ''

---

In deze blog geef ik een korte demonstratie hoe je Prometheus Rules geschreven
in YAML omzet naar efficiënte Nickel-code die YAML kan genereren.

Een van onze klanten heeft ons de opdracht gegeven hun [Grafana
Cloud](https://grafana.com/products/cloud/) in te richten met tientallen
dashboards and Prometheus alert rules. Vergelijkbaar met het _Infrastructure as
Code_ principe hanteren we hier de _Dashboards as Code_ aanpak en voor de
Prometheus alerts, tja _Alerts as Code_.

De dashboards worden voor Grafana gedefinieerd in JSON, maar we schrijven de
dashboard definities in [jsonnet](https://jsonnet.org) en de
[grafonnet-library](https://grafana.github.io/grafonnet-lib). Dit voorkomt dat
we lange lappen code met veel redundantie moeten schrijven. Dit is de manier
die Grafana aanbeveelt.

De [Prometheus Alerts
Rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
worden aangeboden als YAML bestanden en ook hierin zit ontzettend veel
herhaling. Behalve dat dit veel schrijfwerk is, is het ook foutgevoelig om
zoveel code te schrijven en belangrijk is ook de _configuratie drift_ die snel
optreedt als je redundante configuratie code schrijft. Door code te genereren
werk je toe naar een [single source of
truth](https://en.wikipedia.org/wiki/Single_source_of_truth).

> "Configuration drift occurs when a standardized group of IT resources, be
> they virtual servers, standard router configurations in VNF deployments, or
> any other deployment group that is built from a standard template, diverge in
> configuration over time. … The Infrastructure as Code methodology from DevOps
> is designed to combat Configuration Drift and other infrastructure management
> problems."

- _Kemp Technologies on Configuration Drift_

Er zijn diverse manieren om YAML code te genereren, maar als [Nix en
NixOS](https://nixos.org) enthousiasteling besloot ik te gaan experimenteren
met de programmeertaal [Nickel](https://nickel-lang.org). Deze taal is
voortgekomen uit de Nix-community en de 1.0 versie van Nickel is in mei 2023
officieel aangekondigd door [Tweag](https://www.tweag.io/opensource/), het
bedrijf dat de Nickel ontwikkelt.

Aan de slag!

## De oude werkwijze

Hieronder staat een voorbeeld met 2 Prometheus alert rules zoals ze de oude
situatie zijn geschreven. We gaan in een paar stappen werken naar een versie in
Nickel die vergelijkbare, en zelf betere YAML genereert.

```yaml
groups:
  - name: mycompany
    rules:
      - alert: aws_documentdb_freeable_memory_low
        expr: |-
            16 / (aws_docdb_freeable_memory_average{dbinstance_identifier="tf-created-AABBCCDD"}
            / (1024^3)) * 100 < 25
        for: 10m
        labels:
          team: devops
          platform: aws
        annotations:
          description: 'DocumentDB: the instance has less then 25% freeable freeable memory.'
          summary: DocumentDB is has low freeable memory
          jira_ticket: aws-devops
          severity: 2

      - alert: aws_documentdb_cpu_usage_too_high
        expr: aws_docdb_cpuutilization_average > 90
        for: 10m
        labels:
          team: devops
          platform: aws
        annotations:
          summary: "DocumentDB is reaching its cpu limit"
          description: "DocumentDB: the instance are using more than 90% of the cpu assigned"
          jira_ticket: aws-devops
          severity: 2
```

Je ziet op het eerste gezicht al veel dubbele informatie. Sowieso alle
sleutelnamen, maar ook de meta informatie, de Jira tickets, de labels, etc...
Hierboven zie je twee alerts, maar in de echte situatie gaat het om tientallen,
zo niet honderden alerts definities. Hier valt echt wat te winnen.

We gaan nu in een paar stappen deze Prometheus YAML omschrijven slimmere Nickel
broncode.

## Stap 1: YAML naar platte Nickel

Laten we beginnen om de YAML domweg om te zetten naar Nickel zodat we deze
daarna weer kunnen exporten naar YAML.

```ncl
{
  groups = [
    {
      name = "mycompany",
      rules = [
        {
          alert = "aws_documentdb_freeable_memory_low",
          expr = m%"
            16 /
            (aws_docdb_freeable_memory_average{dbinstance_identifier="tf-created-AABBCCDD"} / (1024^3))
            * 100 < 25
          "%,
          for = "10m",
          labels = {
            team = "devops",
            platform = "aws"
          },
          annotations = {
            description = "DocumentDB = the instance has less then 25% freeable freeable memory.",
            summary = "DocumentDB is has low freeable memory",
            jira_ticket = "aws-devops",
            severity = "2",
            }
          },
          {
            alert = "aws_documentdb_cpu_usage_too_high",
            expr = "aws_docdb_cpuutilization_average > 90",
            for = "10m",
            labels = {
              team = "devops",
              platform = "aws"
            },
            annotations = {
              summary = "DocumentDB is reaching its cpu limit",
              description = "DocumentDB = the instance are using more than 90% of the cpu assigned",
              jira_ticket = "aws-devops",
              severity = 2
            }
          }
        ]
      }
    ]
  }
```

Hierboven staat een statische versie van de YAML in Nickel. Direct herkenbaar.
Een soort JSON maar dan met `=` ipv `:`.  Je ziet ook meteen een voorbeeld om
strings op meerdere regels te plaatsten met de `m%"Hello hello.\nText on a new
line"%` schrijfwijze.

Als je dit bestand opslaat als b01-prometheus-rules.ncl kun je het naar YAML
exporteren het commando:

```bash
nickel -f b02-prometheus-rules.ncl export --format yaml
```

Uiteraard kun je dit opslaan in een bestand door `> prometheus-rules.yml` aan de
commandoregel toe te voegen.

```bash
nickel -f b02-prometheus-rules.ncl export --format yaml > prometheus-rules.yml
```

We gaan nu grote stap maken door een functie te introduceren.

> Het je Nickel nog niet geïnstalleerd? Lees de [Getting Started pagina op
> nickel-lang.org](https://nickel-lang.org/getting-started/)
>
> Ben je een Mac-gebruiker? `brew install nickel`.


## Stap 2: Een functie voor de alert-definitie.

We maken een functie met als invoer de naam, de teksten die de problemen
omschrijven, de tijdsduur, de expressie en de severity. Vervolgens roepen we
de functie aan in het regels blok. Ook deze code kunnen we omzetten naar YAML
met het commando `nickel`.

```ncl
let alert_rule = fun name problem_txt_short problem_txt_long duration sev expression => {
  alert = name,
  expr = expression,
  for = duration,
  labels = {
    team = "devops",
    platform = "aws"
  },
  anotations = {
    summary = "%{problem_txt_short}",
    description = "%{problem_txt_long}",
    severity = std.string.from_number sev,
    }
} in

{
  groups = [
    {
      name = "mycompany",
      rules = [
        alert_rule "documentdb_freeable_memory_low" "DocumentDB freeable memory low" "DocumentDB: the instance has less then 25% freeable freeable memory." "10m" 2 m%"
          16 /
          (aws_docdb_freeable_memory_average{dbinstance_identifier="tf-created-AABBCCDD"} / (1024^3))
          * 100 < 25
        "%,

        alert_rule "documentdb_cpu_usage_too_high" "DocumentDB cpu usage too high" "DocumentDB: the instance are using more than 90% of the cpu assigned" "10m" 2 m%"
          aws_docdb_cpuutilization_average > 90
        "%
      ]
    }
  ]
}
```

Even puzzelen wat hier gebeurt. `let alert_rule = fun ...=> {} in` definieert
een functie, wijst deze toe aan de variabele `alert_rule`. De functie geeft in
dit geval een object of anders gezegd een dictionary terug. Vervolgens start de
daadwerkelijke opbouw van de uitvoer. In het blok `rules = [...]` wordt de
functie twee keer aangeroepen, elke alert definitie.

De oorspronkelijke YAML is 28 regels, de eerste Nickel conversie was 43 regels
en nu zijn we al weer terug naar 33 regels. Nog een of twee alert definities en
ons bestand is al ruim compacter dan oorspronkelijk uitgeschreven YAML.

Het is wel even wennen hoe je functies definieert in Nickel. Nickel is
geïnspireerd op talen zoals [Haskell](https://www.haskell.org/) en
[Nix](https://nixos.org/manual/nix/stable/) en dit zijn functionele talen.
Vooral mensen met gevoel voor wiskunde voelen zich snel thuis in functionele
talen. Persoonlijk ben ik een meer een alpha-mens en dol op talen met veel
syntactic sugar zoals mijn lievelingstaal [Ruby](https://www.ruby-lang.org/en/).

In de volgende stap maken we nog een laatste optimalisatieslag om het leven van
de devops'er echt makkelijker te maken.

## Stap 3: Puntjes op de i

Als je naar de teksten kijkt zie je dat de alertnaam de korte en de lange
omschrijving veel terugkerende woorden heeft. Eigenlijk zou de alertnaam
genereert kunnen worden op basis van een service-naam plus de korte
omschrijving. Ook in de korte en lange omschrijving kunnen de service-naam
hergebruiken.

```ncl
let normalize_string = fun instring => std.string.replace_regex "[%,.]+" "" (std.string.replace " " "_" (std.string.lowercase instring)) in

let alert_rule = fun service_name problem_txt_short problem_txt_long duration sev expression => {

  alert = "%{normalize_string service_name}_%{normalize_string problem_txt_short}",
  expr = expression,
  for = duration,
  labels = {
    team = "devops",
    platform = "aws"
  },
  annotations = {
    summary = "%{service_name} is unhealty. %{problem_txt_short}",
    description = "%{service_name} is unhealty. %{problem_txt_long} For at least %{duration}.",
    severity = std.string.from_number sev,
    }
} in

{
  groups = [
    {
      name = "mycompany",
      rules = [
        alert_rule "DocumentDB" "freeable memory low" "The instance has less then 25% freeable memory." "10m" 2 m%"
          16 /
          (aws_docdb_freeable_memory_average{dbinstance_identifier="tf-created-AABBCCDD"} / (1024^3))
          * 100 < 25
        "%,

        alert_rule "DocumentDB" "cpu usage too high" "The instance are using more than 90% of the cpu assigned." "10m" 2 m%"
          aws_docdb_cpuutilization_average > 90
        "%
      ]
    }
  ]
}
```

We hebben een nieuwe functie geïntroduceerd `normalize_string`. Deze gebruiken
we om de naam van de alert te genereren. We halen evt. verkeerde tekens eruit,
vervangen spaties met underscores en maken de naam lowercase. Vervolgens
hergebruiken we de _Service Naam_ `DocumentDB` in de probleemomschrijvingen.
Deze kleine verbeteringen zorgen voor efficiëntere functie aanroepingen en meer
consistente en bruikbaar Alert informatie.

## Conclusie

Door onze Prometheus Alerts in een paar stappen om te schrijven naar Nickel,
hebben we diverse verbeteringen bereikt.

- Onze broncode is compacter geworden en heeft vrijwel geen overbodige tekens of tekst.
- Dubbele broncode zijn we grotendeel kwijtgeraakt.
- Het schrijven van nieuwe alerts gaat veel sneller.
- De broncode is minder gevoelig voor typefouten die kunnen ontstaan door het kopieren en plakken van eerdere definities.
- Door gebruik te maken van een functie lopen we minder risico op configuratie drift.

We hadden ook Python of een andere algemene programmeertaal kunnen gebruiken.
Dit vereist echter veel boilerplate code, en vereist daarnaast behoorlijk wat
documentatie om anderen mee te laten draaien in de workflow.

Nickel is speciaal geschreven met als doel JSON en YAML configuratie-standen
slimmer te genereren. Conversie naar YAML vereist een enkel commando en kan
makkelijk worden opgenomen in een Makefile of Pipeline.

Uiteraard kunnen we nog meer verbeteren. We kunnen een pipeline maken met een
Nickel linter, we kunnen de functies importeren uit een gedeeld library bestand
zodat andere projecten deze ook kunnen gebruiken etc..

Wat ik graag wilde laten zien dat het in veel gevallen verstandig is YAML of
JSON configuraties te genereren met een taal zoals Nickel. En persoonlijk ben
ik na wat spelen erg gesteld geraakt op de syntax van Nickel. Ik er zeker meer
gebruik van maken.

Zie je een fout of wil je graag om een andere reden reageren. Gebruik het
formulier hieronder.

De bestanden uit deze tutorial kun je downloaden in de [git-repo]() die bij dit
artikel hoort.

## Update met code verbeteringen

Supercool! Yann Hamdaoui, de hoofdontwikkelaar van Nickel, gaf naar aanleiding
van dit blog artikel een reactie met aantal suggesties ter verbetering van de Nickel code
voorbeelden. Yann's suggesties laten geavanceerdere mogelijkheden van Nickel
zien, zoals het gebruiken van de _reverse application operator_ `|>`, _type
casting_, het omschrijven van functies naar _bare records_, _contracts_, en hij
stelt ook voor een _record_ als functie parameter te gebruiken in plaats van losse
parameters.

Hieronder toon ik een laatste refactoring waarbij ik de _reverse applicatie
operator_ toepas. Om dit artikel leesbaar voor instappers te houden verwijs ik
je voor de overige verbeteringen van harte door naar [Yann's reactie op de
nickel project pagina](https://github.com/tweag/nickel/discussions/1386#discussioncomment-6240266).

```ncl
# Using the "reverse application operator"
let normalize_string = fun instring =>
  instring
  |> std.string.lowercase
  |> std.string.replace " " "_"
  |> std.string.replace_regex "[%,.]+" ""
in

let alert_rule = fun service_name problem_txt_short problem_txt_long duration sev expression => {

  alert = "%{normalize_string service_name}_%{normalize_string problem_txt_short}",
  expr = expression,
  for = duration,
  labels = {
    team = "devops",
    platform = "aws"
  },
  annotations = {
    summary = "%{service_name} is unhealty. %{problem_txt_short}",
    description = "%{service_name} is unhealty. %{problem_txt_long} For at least %{duration}.",
    severity = std.string.from_number sev,
    }
} in

{
  groups = [
    {
      name = "mycompany",
      rules = [
        alert_rule "DocumentDB" "freeable memory low" "The instance has less then 25% freeable memory." "10m" 2 m%"
          16 /
          (aws_docdb_freeable_memory_average{dbinstance_identifier="tf-created-AABBCCDD"} / (1024^3))
          * 100 < 25
        "%,

        alert_rule "DocumentDB" "cpu usage too high" "The instance are using more than 90% of the cpu assigned." "10m" 2 m%"
          aws_docdb_cpuutilization_average > 90
        "%
      ]
    }
  ]
}
```


## Links

- [nickel-lang.org](https://nickel-lang.org) - The Nickel Home Page with general information and documentation.
- [Announcing Nickel 1.0](https://www.tweag.io/blog/2023-05-17-nickel-1.0-release) - The Nickel 1.0 release announcement with some code examples.
- [Nickel Q & A](https://github.com/tweag/nickel/discussions/categories/q-a) - This is the place to ask for help.