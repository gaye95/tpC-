# TP — Boutique Diayma

## 1. Récupérer et lancer le projet

Le dépôt source de l’application est [devehod/BoutiqueDiayma2025](https://github.com/devehod/BoutiqueDiayma2025). La solution s’ouvre dans Visual Studio avec `Diayma.sln`. Dans Visual Studio, définir `Diayma` comme projet de démarrage, puis lancer avec **F5**. Le navigateur affiche la liste des produits à l’adresse locale indiquée par le profil de lancement.

Le projet a aussi été compilé et publié depuis la ligne de commande. Pour lancer la version Windows autonome fournie dans ce dépôt, extraire l’archive de la section 10 et exécuter `Diayma.exe`.

## 2. Projets de la solution

La solution contient **un projet** :

| Projet | Fichier | Type |
|---|---|---|
| `Diayma` | `P2FixAnAppDotNetCode/Diayma.csproj` | Application web ASP.NET Core MVC |

## 3–4. Framework cible et SDK .NET

Le fichier `P2FixAnAppDotNetCode/Diayma.csproj` cible `netcoreapp2.0` : l’application a été écrite pour **.NET Core 2.0**. Cette valeur est le *framework cible*, pas une version de SDK épinglée. Le dépôt ne contient pas de `global.json` pour fixer un SDK.

Le SDK présent sur la machine de réalisation est **.NET SDK 10.0.403 (x64)**. Il a servi à restaurer, compiler et publier le projet. L’installation du vieux SDK 2.0 a été tentée, mais l’installateur ne trouve plus ses fichiers : .NET Core 2.0 est hors support. La compilation d’origine signale également des avertissements de fin de support et des avis de vulnérabilité sur les dépendances historiques. Le binaire livré est autonome et n’a donc pas besoin que le runtime .NET Core 2.0 soit installé sur le PC Windows.

## 5. Dépôt GitHub de rendu

Le code et ce compte rendu sont publiés dans le dépôt [gaye95/tpC-](https://github.com/gaye95/tpC-).

## 6. Deux bugs observés

1. **Le total du panier ne tient pas compte des quantités.** Dans `Cart.GetTotalValue()`, le calcul additionne le prix de chaque ligne une seule fois (`Price`) au lieu de multiplier le prix par `Quantity`. Ajouter deux exemplaires du même produit fait donc afficher un total trop bas dans le récapitulatif du panier.
2. **Le panier est partagé entre les visiteurs.** `Startup.ConfigureServices` enregistre `ICart` avec `AddSingleton`. Une seule instance de `Cart` est alors réutilisée pour toutes les requêtes et tous les utilisateurs : les articles d’un visiteur peuvent apparaître dans le panier d’un autre et une commande peut vider le panier partagé.

Ces bugs sont signalés pour le TP ; ils ne sont pas corrigés dans le code source.

## 7. Points d’arrêt à placer

Dans Visual Studio, ouvrir le fichier demandé et cliquer dans la marge à la ligne indiquée (ou appuyer sur **F9** sur la ligne) :

| Fichier / classe | Ligne | Instruction arrêtée |
|---|---:|---|
| `Components/CartSummaryViewComponent.cs` — `CartSummaryViewComponent` | 12 | Affectation du panier injecté |
| `Controllers/ProductController.cs` — `ProductController` | 15 | Affectation du service de produits |
| `Controllers/OrderController.cs` — `OrderController` | 17 | Affectation du panier |
| `Controllers/CartController.cs` — `CartController` | 15 | Affectation du panier |
| `Startup.cs` — `Startup` | 20 | Affectation de la configuration |

Les lignes correspondent aux fichiers du dépôt source fourni.

## 8. Parcours avant l’affichage des produits

Le chemin ci-dessous décrit le traitement utile de la requête d’accueil (`GET /`), en laissant de côté les appels internes du framework ASP.NET Core.

1. **Démarrage de l’application** — `P2FixAnAppDotNetCode.Program.Main(args)` appelle `Program.BuildWebHost(args)`. Le démarrage configure `P2FixAnAppDotNetCode.Startup` : `Startup.ConfigureServices(IServiceCollection)` enregistre les services et contrôleurs, puis `Startup.Configure(IApplicationBuilder, IHostingEnvironment)` installe les middlewares.
2. **Routage de la requête** — `Startup.Configure` ajoute les fichiers statiques, la localisation et la session, puis `UseMvc` mappe la route par défaut `{controller=Product}/{action=Index}/{id?}`. La route `/` désigne donc `ProductController.Index`.
3. **Injection des dépendances** — MVC crée `P2FixAnAppDotNetCode.Controllers.ProductController`, avec un `IProductService` (implémenté par `P2FixAnAppDotNetCode.Models.Services.ProductService`) et un `ILanguageService`. À la première résolution du dépôt, `ProductRepository` construit les données en appelant `GenerateProductData()`.
4. **Chargement des produits** — `ProductController.Index()` appelle `ProductService.GetAllProducts()`. Cette méthode appelle `P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GetAllProducts()`, qui filtre les produits dont le stock est positif et les trie par nom. Le service convertit le tableau obtenu en `List<Product>`.
5. **Création de la réponse** — `ProductController.Index()` retourne `View(products)`. Razor rend `Views/Product/Index.cshtml`, qui parcourt la liste et produit une ligne HTML par produit. Le layout `Views/Shared/_Layout.cshtml` entoure la page et invoque aussi les composants `CartSummaryViewComponent.Invoke()` et `LanguageSelectorViewComponent.Invoke()` pour le panier et la langue.

### Pas à pas conseillé

- Utiliser **Pas à pas détaillé (F11)** depuis l’appel à `GetAllProducts()` dans `ProductController.Index` pour entrer dans `ProductService.GetAllProducts()`, puis dans `ProductRepository.GetAllProducts()`.
- Utiliser **Pas à pas sortant (Maj+F11)** pour revenir du dépôt au service, puis au contrôleur.
- Utiliser **Pas à pas principal (F10)** pour avancer sans entrer dans le code du framework MVC ou de Razor jusqu’à `View(products)`. Le rendu HTML de la vue se poursuit ensuite dans le moteur Razor.

## 9. Publication Windows

Commande ayant produit l’application autonome Windows x64 (à lancer depuis la racine du dépôt) :

```powershell
dotnet publish P2FixAnAppDotNetCode\Diayma.csproj `
  --configuration Release `
  --runtime win-x64 `
  --self-contained true `
  --output dist\win-x64
```

Le résultat est `dist\win-x64\Diayma.exe` accompagné de ses fichiers d’exécution. L’application a été démarrée localement et la page d’accueil a répondu **HTTP 200** avec les produits attendus.

## 10. Téléchargement de l’exécutable

Archive Windows autonome : [Télécharger Diayma pour Windows x64](https://github.com/gaye95/tpC-/raw/refs/heads/main/dist/Diayma-win-x64.zip).

Extraire l’archive, puis lancer `Diayma.exe`. Le serveur local écoute sur le port 5000 par défaut.
