---
title: Serverless redirects
draft: false
subtitle: Eenvoudige serverless redirects met Terraform in AWS.
pubdate: '2023-05-18'
type: artikel
image: imgs/seed_565023_00002.png
categories:
  - aws
  - terraform
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

Het klinkt zo eenvoudig... Een website is verhuisd dus de oude locatie moet
doorverwijzen naar een nieuwe locatie. Technisch gezien is het ook niet zo heel
spannend, maar toch zijn er meerdere juist geconfigureerde componentjes nodig
om dit voor de gebruiker vlot te laten verlopen. In de oude wereld had je op
z'n minst een webserver nodig. Deze tutorial laat je zien hoe je dit met
[TechNative's Terraform url-redirect
Module](https://registry.terraform.io/modules/wearetechnative/module-url-redirect/aws/latest)
snel en eenvoudig in je AWS account kunt uitrollen.

Wat heb je er voor nodig:

- een beetje kennis van Terraform
- een AWS Account
- het oude domein moet als route53 zone in je AWS account aanwezig zijn.

Stel je website draaide eerst op https://eerste-locatie.amsterdam en inmiddels
draait je website op https://nieuwe-locatie.amsterdam. Eerdere bezoekers hebben
nog bookmarks naar de oude locatie en ook op talloze plekken zullen nog links
naar de oude locatie aanwezig zijn. The 'internet never forgets'

Om al deze oude links toch een eind bestemming bij jou nieuwe website te geven
willen we de situatie als volgt inregelen.

Als een eerdere bezoeker `nieuwe-locatie.amsterdam` in z'n browser invoert zal de
browser denken dat hij naar `https://eerste-locatie.amsterdam` moet. De browser
zal een DNS query uitvoeren en wordt doorgestuurd naar een IP-adres waar een
webserver naar port 443 luistert. Hier zal de browser vragen om het SSL
certificaat dat hoort bij `eerste-locatie.amsterdam`. Pas als dit certificaat
deze geldig is beschouwd accepteert de browser de response van de webserver. De
webserver geeft in z'n response de opdracht om in het vervolg direct de url
`https://nieuwe-locatie.amsterdam/we-zijn-verhuisd` te openen. Hier staat een
bericht dat de bezoeker uitlegt dat de website is voortaan op deze nieuwe plek
te bezoeken is.

Met AWS bouwstenen gaan we dit als volgt oplossen:

Een _Route53_ record in de zone van `eerste-locatie.amsterdam` gaat verwijzen
naar een _CloudFfront_ distributie. Deze CF distributie heeft een _ACM_ SSL
certificaat gekoppeld dat bewijst dat `eerste-locatie.amsterdam` daadwerkelijk
in beheer is door de eigenaar van deze CloudFront resource. De _CloudFfront_
service stuurt alle `http` en `https` verzoeken door naar een als webserver
geconfigureerde _S3_ bucket. Deze _S3_ webserver stuurt vervolgens alle
browsers en web-agents door naar
`https://nieuwe-locatie.amsterdam/we-zijn-verhuisd`

Samengevat gebruiken we deze AWS diensten:

- [Route53](https://aws.amazon.com/route53/)
- [CloudFront](https://aws.amazon.com/cloudfront/)
- [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/)
- [S3](https://aws.amazon.com/s3/)

Ik ga niet helemaal uitleggen hoe je jouwe AWS account met Terraform
configureert. Bekijk [de tutorials op de website van de makers van
Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started)
hoe je kunt beginnen. Als je een werkende setup hebt en je omgeving kan
aanpassen met `terraform apply` maak dan een nieuw bestand aan in je Terraform
codebase en voeg hier onderstaande code aan toe.


```hcl

module "redirect_technative_nl" {
  source  = "wearetechnative/module-url-redirect/aws"
  version = "0.1.1"
  domain                          = "technative.nl"
  route53_zone_name               = "technative.nl."

  providers = {
    aws.us-east-1: aws.us-east-1
  }

  redirect_target_url = "https://technative.eu"
}
```

Als je deze TF code uitrolt zal het even duren omdat vooral het aanmaken van
een CloudFront distribute even duurt. Ook het valideren van de SSL certificaten
neemt wat tijd in beslag. Bovenstaand code blokken kun je voor
verschillende oude domeinnamen zoveel herhalen als je wilt.

Bedankt voor het lezen van deze tutorial. Mocht tegen problemen aanlopen, dan
horen we dit graag via ons [ticketsysteem op
github](https://github.com/wearetechnative/terraform-aws-module-url-redirect/issues).