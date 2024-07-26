## Ghid: Crearea unei Aplicații folosind Azure Cloud


### Pasul 1: Crearea Resource Group-ului "aplicatie-laborator" folosind Azure Cloud Shell

1. Deschide [Azure Portal](https://portal.azure.com/) într-un browser web.

2. În partea dreaptă sus a ecranului, click pe iconița de Cloud Shell (ar trebui să arate ca un terminal).

3. Dacă este prima dată când folosești Cloud Shell, va trebui să alegi între Bash și PowerShell. Pentru acest ghid, vom folosi Bash. Dacă ai deja configurat Cloud Shell, poți sări peste acest pas.

4. În Cloud Shell, execută următoarea comandă pentru a crea un resource group numit "aplicatie-laborator" în regiunea "westeurope":
```bash
#!/bin/bash

# Prefix for the resource group name
PREFIX="rg"
export location="westeurope"
export appServicePlan="plan-aplicatie-laborator"
export appService="aplicatie-laborator-service"
# Get the currently logged in Azure username
AZURE_USER=$(az ad signed-in-user show --query userPrincipalName -o tsv | cut -d'@' -f1)

# Generate the resource group name
GROUP_NAME="${PREFIX}-${AZURE_USER}"

# Set the resource group name in an environment variable
export resourceGroup=$GROUP_NAME

# Create the resource group in the specified location
az group create --name "$resourceGroup" --location $location

# Output the created resource group name
echo "Resource group created with name: $resourceGroup"
```
Explicație Comandă
- az group create este comanda Azure CLI pentru a crea un nou resource group.
- --name aplicatie-laborator specifică numele resource group-ului.
- --location westeurope specifică regiunea în care va fi creat resource group-ul.


 ### Pasul 2: Crearea unui Repository GitHub pentru Procesul de CI/CD


CI/CD (Continuous Integration/Continuous Deployment) este o practică DevOps esențială care permite echipelor de dezvoltare să livreze modificările codului mai frecvent și cu o mai mare fiabilitate. CI/CD implică două concepte principale:

- Continuous Integration (CI): Procesul de integrare continuă a modificărilor codului într-un repository partajat, unde fiecare modificare este testată automat pentru a asigura că nu cauzează erori.

- Continuous Deployment (CD): Procesul de implementare continuă a modificărilor validate automat într-un mediu de producție sau pre-producție, asigurând astfel o livrare rapidă și fiabilă a funcționalităților noi.

**Crearea unui Repository GitHub și Încărcarea Codului**

Pentru a crea un repository GitHub public și a încărca codul din /samples/azure-cloud-sample/cicd-github-actions-sample, urmează acești pași:


1. Accesează [GitHub](https://github.com/) și Autentifică-te:
2. Click pe butonul + din colțul din dreapta sus și selectează New repository.
3. Configurează Repository-ul:
    - Repository name: aplicatie-laborator-azure
    - Description: (opțional) Repository pentru procesul de CI/CD cu Azure
    - Public/Private: Selectează Public
    - Add .gitignore: Node
4. Inițializează Repository-ul Local:
- Deschide terminalul (local, din folder-ul **fii-cloud-computing-practice**) și execută următoarele comenzi pentru a inițializa repository-ul local și a adăuga fișierele din /samples/azure-cloud-sample/cicd-github-actions-sample:

- inlocuieste USERNAME cu username-ul din GitHub

```bash
# Clonează repository-ul nou creat
git clone https://github.com/<USERNAME>/aplicatie-laborator-azure.git
git clone https://github.com/matei97/fii-cloud-computing-practice.git

cd aplicatie-laborator-azure

git config --global user.email "user@email.com"
git config --global user.name "Your Name"

# Copiază fișierele din directorul sursă în repository
cp -r ../fii-cloud-computing-practice/samples/azure-cloud-sample/cicd-github-actions-sample/* .

# Adaugă fișierele la repository
git add .

# Fă un commit cu mesajul inițial
git commit -m "Initial commit with CI/CD sample code"

# Trimite modificările la repository-ul GitHub
git push origin main
```

 ### Pasul 3: Generarea unui Personal Access Token din Interfața Web GitHub

1. Mergi la [GitHub](https://github.com/) și autentifică-te în contul tău.
2. Accesează Setările de Securitate:
- Click pe poza ta de profil în colțul din dreapta sus al paginii.
- Selectează "Settings" (Setări) din meniu.
- În meniul din stânga, selectează "Developer settings" (Setări pentru dezvoltatori).
3. Navighează la Personal Access Tokens:

- În meniul din stânga, selectează "Personal access tokens".
- Click pe "Tokens (classic)" pentru a vedea token-urile existente și opțiunea de a genera unul nou.
4. Generarea unui Token Nou:
- Click pe butonul verde "Generate new token".
- Introdu un nume descriptiv pentru token-ul tău în câmpul "Note" (Notă), astfel încât să știi pentru ce l-ai creat.
5. Bifeaza toate permisiunile pentru token
6. Generarea și Salvarea Token-ului:
- După ce ai selectat permisiunile, click pe butonul verde "Generate token" (Generează token).
- GitHub va genera token-ul și îți va afișa o valoare unică. Notează-ți această valoare într-un loc sigur, deoarece nu vei mai putea să o vezi din nou după ce părăsești această pagină.
7. Executa comanda urmatoarea pentru a crea o variabial de mediu cu tokenul generat.

```bash
export token="<TOKEN>"
export gitUsername="<GIT_USERNAME>"
export gitRepo="aplicatie-laborator-azure"
```



 ### Pasul 4: Crearea unui App Service cu Opțiunea de Continuous Deployment Activată și Legată la Repository-ul GitHub

Pentru a crea un App Service în Azure cu opțiunea de continuous deployment activată și legată la repository-ul GitHub creat anterior, urmează acești pași:

1. Accesează Azure Cloud Shell:

2. Creează un App Service Plan:

Un App Service Plan definește regiunea, nivelul de scalare și prețul pentru aplicația ta. Creează un App Service Plan cu comanda:


```bash
az appservice plan create --name $appServicePlan --resource-group $resourceGroup --location $location --sku B1 --is-linux
```

- --name plan-aplicatie-laborator: Numele planului.
- --resource-group aplicatie-laborator: Numele resource group-ului.
- --location westeurope: Regiunea în care va fi creat planul.
- --sku B1: Nivelul de preț și scalare (Basic, nivel 1).

3. Creează App Service-ul:

```bash
az webapp create --name $appService --resource-group $resourceGroup --plan $appServicePlan  --runtime "NODE:20-lts"
```

- --name aplicatie-laborator-service: Numele aplicației.
- --resource-group aplicatie-laborator: Numele resource group-ului.
- --plan plan-aplicatie-laborator: Numele planului de App Service creat anterior.

4. Setează Configurația pentru Deploy:

```bash
az webapp deployment github-actions add --repo "$gitUsername/$gitRepo" -g $resourceGroup -n $appService --token $token -b "main"

echo Verifica pagina "https://github.com/$gitUsername/$gitRepo/actions/"
```


 ### Pasul 4: Stergerea resurselor

 **Această comandă va sterge resrouce group-ul si toate resursele aferente.**

```bash
az group delete --name $resourceGroup
```
