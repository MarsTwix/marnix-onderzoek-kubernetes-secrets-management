# Kubernetes Secret Management: Het Geheime Ingrediënt voor Sterke Applicatiebeveiliging
 
*Marnix Wildeman*, Oktober 2023
<hr>

<img align="right" style="width: 140px;" src="https://in4it.com/wp-content/uploads/2020/11/secret.jpg">
<img align="right" style="width: 140px;" src="https://b2750956.smushcdn.com/2750956/wp-content/uploads/2021/01/3-31510_svg-kubernetes-logo-hd-png-download-696x664.png?lossy=1&strip=1&webp=1">
 
In de steeds sneller evoluerende wereld van softwareontwikkeling is beveiliging niet langer een bijzaak. Het beschermen van gevoelige gegevens en de integriteit van applicaties is van essentieel belang. Deze blog duikt in de wereld van Kubernetes Secrets en laat zien hoe je deze geheime 'sleutels' kunt beheren om jouw applicaties aanzienlijk veiliger te maken. Ontdek de strategieën en best practices die je kunt toepassen om de beveiliging van jouw Kubernetes-gebaseerde applicaties te versterken.

**PERSOONLIJKER MAKEN**

## Wat is Kubernetes Secret Management?
Kubernetes Secrets zijn een ingebouwd Kubernetes-resourcestype dat wordt gebruikt om gevoelige gegevens te beheren en op te slaan. Dit kunnen wachtwoorden, tokens, sleutels, of andere geheime informatie zijn die nodig is door applicaties binnen een cluster. Secrets worden base64-gecodeerd opgeslagen, maar het is belangrijk op te merken dat dit geen echte beveiliging biedt - het is meer een vorm van "security through obscurity". Daarom is effectief beheer van Secrets essentieel.

### Soorten Secrets

#### Generic/Opaque Secrets
Dit zijn de meest voorkomende secrets en de standaard secret in Kubernetes. Ze kunnen willekeurige tekstgegevens opslaan, zoals gebruikersnamen en wachtwoorden, en worden vaak gebruikt om toegangsgegevens voor databases, API's en andere services op te slaan.

#### SSH Secrets
Deze secrets worden gebruikt om SSH-sleutels op te slaan die nodig zijn voor toegang tot andere servers. Ze kunnen worden gebruikt in pods om beveiligde verbindingen met externe servers tot stand te brengen.

#### TLS Secrets 
TLS (Transport Layer Security) Secrets worden gebruikt om SSL/TLS-certificaten op te slaan. Deze certificaten worden vaak gebruikt voor het beveiligen van communicatie met services, zoals HTTPS-verbindingen.

#### Service Account Tokens
Elke pod in een Kubernetes-cluster heeft standaard toegang tot een serviceaccount-token. Deze tokens kunnen worden gebruikt om toegang te krijgen tot de Kubernetes API-server en andere Kubernetes-resources, zoals Secrets. Ze zijn handig voor het automatiseren van taken binnen een cluster.

#### Docker Registry Secrets
Deze secrets worden gebruikt om inloggegevens voor Docker-registries op te slaan. Dit stelt pods in staat om Docker-images te halen van privéregio's die authenticatie vereisen.

## Aanmaken van Secrets

### Via command line

Maak username.txt en password.txt tekst bestanden.

```bash
echo -n 'root' > ./username.txt
echo -n 'Mq2D#(8gf09' > ./password.txt
```

en

```bash
$ kubectl create secret generic db-cerds \
  --from-file=./username.txt \
  --from-file=./password.txt


Output:

secret/db-cerds created
```

Lijst van secrets:

```bash
$ kubectl get secret/db-cerds


Output:

NAME       TYPE      DATA      AGE
db-cerds   Opaque    2         26s
```

Bekijken van secret:

```bash
$ kubectl describe secret/db-cerds


Output:

Name:         db-cerds
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password.txt:  11 bytes
username.txt:  4 bytes
```

### Via YAML bestand

The Secret bevat twee manieren: data en stringdata. Het data veld wordt gebruikt om willekeurige gegevens op te slaan, gecodeerd met base64. Nu gaan wij de secret data omzetten naar base64.

```bash
$ echo -n 'root' | base64
Output: cm9vdA==

$ echo -n 'Mq2D#(8gf09' | base64
Output: TXEyRCMoOGdmMDk=
```

Aanmaken van een YAML bestand genaamd `creds.yaml` met de secret data.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-creds
type: Opaque
data:
  username: cm9vdA==
  password: TXEyRCMojGdmMDk=
```

Aanmaken van de secret door `kubectl create`

```bash
$ kubectl create -f creds.yaml

Output: 

secret/database-creds created
```

Lijst van secrets:

```bash
$ kubectl get secret/database-creds

Output:

NAME             TYPE      DATA      AGE
database-creds   Opaque    2         1m
```

Bekijken van secret:

```bash
$ kubectl get secret/database-creds -o yaml

Output:

apiVersion: v1
data:
  password: TXEyRCMojGdmMDk=
  username: cm9vdA==
kind: Secret
metadata:
  creationTimestamp: "2023-10-05T17:45:17Z"
  name: database-creds
  namespace: default
  resourceVersion: "105750"
  uid: 26d540ba-6e44-43a0-ab3a-397f0aac46ff
type: Opaque
```

## Best Practices

**INTRO**

### Gebruik Namespaces
Gebruik namespaces om uw secrets te isoleren en de toegang daartoe te beperken. U kunt voor elk team of elke applicatie een aparte namespaces aanmaken en alleen toegang verlenen aan het noodzakelijke personeel.

### Gebruik encryptie
Gebruik encryptie om uw geheimen tijdens verzending en in rust te beschermen. Kubernetes versleutelt automatisch geheimen in rust, maar u kunt ook TLS gebruiken om ze onderweg te versleutelen.

### Gebruik RBAC
Gebruik Role-Based Access Control (RBAC) om de toegang tot geheimen te beheren. Gebruik RBAC om toegang te verlenen tot specifieke secrets of namespaces, en om de toegang tot gevoelige secrets te beperken.

### Gebruik extern geheimenbeheer 
Gebruik externe oplossingen voor geheimenbeheer, zoals Hashicorp Vault of AWS Secrets Manager, om uw geheimen extern te beheren. Met deze aanpak kunt u uw secrets veilig opslaan en kunt u Kubernetes gebruiken om ze op te halen wanneer dat nodig is.

### Verander secrets regelmatig 
Roteer uw secrets regelmatig, bijvoorbeeld elke 30 dagen, om ervoor te zorgen dat ze veilig blijven. Gebruik automatiseringstools om dit proces te automatiseren en vermijd handmatige tussenkomst.

## Populaire Secret Management tools

### HashiCorp Vault 
HashiCorp Vault is een gratis en open-source tool voor het veilig opslaan en beheren van gevoelige gegevens, zoals tokens, certificaten en API-sleutels. Het biedt een uniforme interface voor het beheren van geheimen en kan gebruikersaudits genereren. Vault ondersteunt traditionele en moderne methoden voor gebruikersauthenticatie en is een populaire keuze voor secrets management.

### Akeyless 
Akeyless Vault is een centrale oplossing voor geheimenbeheer, inclusief integratie met Kubernetes. Het gebruikt de Kubernetes Mutating Admission Webhook om geheimen in pods te injecteren en biedt functies zoals segregatie tussen Kubernetes-clusters, SSO-ondersteuning en verschillende authenticatiemogelijkheden.

### CyberArk Conjur 
CyberArk Secrets Manager is ontworpen voor Kubernetes en biedt geheimen- en referentiebeheer voor cloud-native en containeromgevingen. Het verwijdert hardgecodeerde geheimen uit broncode en biedt centraal beheer voor verschillende toepassingsworkloads.

### AWS Secrets Manager
AWS Secrets Manager is een AWS-service waarmee je snel database-inloggegevens, API-sleutels en andere wachtwoorden kunt roteren, beheren en ophalen. Met Secrets Manager kun je geheimen beveiligen, analyseren en beheren die nodig zijn om toegang te krijgen tot AWS Cloud-mogelijkheden, externe services en on-premises.

### Google Cloud Secrets Manager
Secret Manager is een dienst van Google Cloud voor het beheren van geheimen op clusters. Het biedt een veilig opslagsysteem voor API-sleutels, certificaten, wachtwoorden en andere gevoelige gegevens. Het biedt een centrale plek en een enkele bron om geheimen over Google Cloud te beheren, met functies zoals versiebeheer van geheimen, integratie met cloud IAM en audit logging.

### Azure Key Vault
Azure Key Vault is een Microsoft Azure-service waarmee gebruikers cryptografische sleutels kunnen opslaan en gebruiken. Voor klantspecifieke sleutels biedt Azure Key Vault verschillende sleuteltypen en algoritmen, evenals het gebruik van Hardware Security Modules (HSM).

**TABEL MET VERGELIJKING**

## Conclusie
**SOORT SAMENVATTING**

## Bronnen
- 
