EMGStores
Cette application ASP.NET Core MVC est destinée à gérer l’inventaire de voitures d’EMG. EMG achète des voitures, les répare, puis les revend avec bénéfice. L’application permet de :

Afficher la liste des voitures disponibles (public).
Ajouter, modifier, supprimer les annonces de voitures (réservé à l’Administrateur).
Marquer une voiture comme “vendue” ou “plus disponible”.
Protéger l’accès en écriture avec des rôles (via Microsoft Identity).

.NET 6 ou supérieur (SDK). Vous pouvez vérifier avec dotnet --version.
SQL Server localement (ou une autre base compatible EF Core, comme Azure SQL, PostgreSQL, etc.). Si vous utilisez une base différente, ajustez la chaîne de connexion dans appsettings.json.
Un IDE comme Visual Studio, Visual Studio Code ou Rider, ou la CLI .NET pour compiler/ouvrir le projet.
Structure du projet
rust
Copier
Modifier
EMGStores.sln               -> Fichier solution Visual Studio
|
+--- EMGStores/             -> Projet principal ASP.NET Core
|    ├── Controllers/       -> Contrôleurs MVC (HomeController, CarsController, etc.)
|    ├── Data/              -> DbContext (ApplicationDbContext.cs)
|    ├── Models/            -> Modèles de données (Car.cs, Make.cs, Model.cs, etc.)
|    ├── Services/          -> Services métier (CarService.cs, ICarService.cs, etc.)
|    ├── Views/             -> Vues Razor (Index.cshtml, Create.cshtml, etc.)
|    ├── wwwroot/           -> Fichiers statiques (CSS, JS, images)
|    ├── appsettings.json   -> Configuration (BD, JWT, etc.)
|    ├── appsettings.Development.json
|    ├── Program.cs         -> Point d'entrée de l'application (.NET 6)
|    └── EMGStores.csproj   -> Fichier projet .NET
|
+--- EMGStores.Tests/        -> Projet de tests unitaires
|    ├── EMGStores.Tests.csproj
|    ├── CarServiceTests.cs  -> Exemple de tests sur le service CarService
|    └── ...
|
└── README.md               -> Document d’explications (celui-ci)


Installation
Cloner ou télécharger ce dépôt :

git clone https://github.com/mbaye12018/EMGStores.git

cd EMGStores
Ouvrir la solution avec Visual Studio, VS Code ou un autre IDE :

Visual Studio : Double-cliquez sur EMGStores.sln.
VS Code : Lancez code . dans le dossier du projet.
Restaurer les dépendances :

Dans Visual Studio, la restauration se fait automatiquement.
En ligne de commande, vous pouvez faire :

json
Copier
Modifier
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=EMGStoresDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}


Utiliser SQL Server LocalDB/Express (Windows)
Vérifiez que Server=localhost (ou (localdb)\\MSSQLLocalDB) correspond à votre configuration.
Assurez-vous que la base EMGStoresDb n’existe pas encore, ou bien qu’elle est vide.

Option 2 : Utiliser SQL Server sur Azure ou un autre SGBD
Remplacez la valeur de "DefaultConnection" par votre propre chaîne de connexion.
Installez le provider EF Core adéquat (ex. Npgsql pour PostgreSQL) si nécessaire.
Créer la base et appliquer les migrations
Pour créer la base et appliquer les tables :


dotnet ef migrations add InitialCreate --project EMGStores
dotnet ef database update --project EMGStores
InitialCreate est un nom d’exemple pour la migration.
Le dossier Migrations/ sera créé dans le projet EMGStores si vous ne l’avez pas déjà.
Une fois cette étape terminée, la base EMGStoresDb est prête à être utilisée.

Lancement de l’application
Plusieurs possibilités :

1. Visual Studio
Choisissez le projet EMGStores comme projet de démarrage.
Cliquez sur le bouton Exécuter (IIS Express ou profil .NET).
L’application s’ouvrira dans votre navigateur à l’adresse https://localhost:5001 ou http://localhost:5000 (selon vos settings).
2. Ligne de commande (CLI .NET)
Dans le dossier racine, exécutez :


dotnet run --project EMGStores
Par défaut, l’application tournera sur https://localhost:7280 et http://localhost:5280 (ou d’autres ports dynamiques). L’URL exact sera affiché dans la console.

Fonctionnalités principales
Accueil (Home/Index)

Page d’accueil (Front-end avec bootstrap) présentant EMG.
Lien direct : https://localhost:5001/ (selon configuration).
Listing des voitures (Cars/Index)

Affiche toutes les voitures disponibles (Make, Model, Year, Price, etc.).
Accessible à tous (aucune authentification nécessaire).
Détails d’une voiture (Cars/Details/{id})

Affiche les informations détaillées sur une voiture.
Ajout, modification, suppression (Cars/Create, Cars/Edit, Cars/Delete)

Réservé à l’Admin grâce à [Authorize(Roles="Administrator")].
Permet de gérer l’annonce : description, photo, etc.
Marquer la voiture comme vendue

Action Cars/MarkAsSold/{id}, accessible à l’Admin.
Met la propriété IsSold = true.
Authentification et rôles
Le projet utilise Microsoft Identity :

ApplicationDbContext hérite de IdentityDbContext pour stocker les utilisateurs et rôles en base.
Par défaut, après la première migration, les tables d’Identity sont créées (AspNetUsers, AspNetRoles, etc.).
Les contrôleurs ou actions protégées utilisent [Authorize]
Tests unitaires

Le dossier EMGStores.Tests contient des tests unitaires, par exemple :

CarServiceTests.cs : Teste les méthodes CRUD du service CarService en utilisant une base InMemory.
CarsControllerTests.cs (si vous voulez tester le contrôleur).
Pour lancer tous les tests :


Visual Studio dispose aussi d’une fenêtre Test Explorer pour exécuter et visualiser les tests.

CI/CD
Vous pouvez ajouter un pipeline (Azure DevOps, GitHub Actions, GitLab CI, etc.) pour :

Restaurer les dépendances (dotnet restore).
Compiler le projet (dotnet build).
Exécuter les tests (dotnet test).
Exemple minimal avec GitHub Actions dans .github/workflows/ci.yml :


on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: windows-latest

    steps:
    - name: Check out code
      uses: actions/checkout@v3

    - name: Set up .NET
      uses: actions/setup-dotnet@v2
      with:
        dotnet-version: '6.0.x'

    - name: Restore dependencies
      run: dotnet restore EMGStores.sln

    - name: Build
      run: dotnet build EMGStores.sln --configuration Release --no-restore

    - name: Test
      run: dotnet test EMGStores.Tests/EMGStores.Tests.csproj --no-build --configuration Release
Contribution
Les contributions sont les bienvenues ! Pour proposer une amélioration ou corriger un bug :

