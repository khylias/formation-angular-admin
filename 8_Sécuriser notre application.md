# Sécuriser notre application

PROBLEMATIQUE : Protéger notre application par un accès authentifié

Jusqu'à maintenant, si nous mettons notre application en ligne, n'importe qui sera en mesure d'y accéder et donc d'y créer ou supprimer des données. La prochaine étape consiste à protéger notre application dernière une connexion d'un utilisateur.

1. La page de connexion

Pour commencer, nous allons créer une page de connexion sur laquelle l'utilisateur pourra s'authentifier.

Créez un composant Login et initialisez un formulaire avec deux champs obligatoires: username et password. 
*N'hésitez pas à retourner consulter le TP 6_Ajouter du contenu*

Ajoutons également une route pour accéder à notre page.

```typescript
{
  	path: 'connexion',
    component: LoginComponent
}
```

2. S'authentifier à l'API

A partir de l'API, vous devez pouvoir accéder à la documentation sur l'url locale : 
http://localhost:3000/api/

Vous trouverez bon nombre d'informations dans cette documentation. Pour l'instant, concentrons-nous sur la ligne `POST /auth/login`. En cliquant sur cette ligne, vous allez dérouler le détail de la route d'API.

Nous pouvons trouver différentes informations : le type de requête, les paramètres, le body, le type de réponse possible.

En tenant compte de cette documentation, nous allons ajouter la méthode d'authentification à la suite de notre formulaire de connexion.

Commençons par créer un service User, nous y mettrons toutes les méthodes relatives à la gestion de l'utilisateur.

```bash
ng g service services/user
```

N'oublions pas de le rajouter aux tableaux des `providers` de notre AppModule.

Comme pour le ClientService, nous utiliserons le module http pour communiquer avec l'API.

```typescript
constructor(private httpClient: HttpClient) { }
```

Créons la méthode `connect` qui fera la requête à l'API. Cette fonction aura pour paramètre une variable data, qui représente le corps de la requête POST.

```typescript
connect(data) {
  return this.httpClient.post('http://localhost:3000/auth/login', data);
}
```

Utilisons cette méthode à l'envoi du formulaire de connexion.

```typescript
login() {
  this.userService.connect(this.form.value).subscribe(response => {
    console.log(response);        
  })
}
```

Nous pouvons désormais nous connecter avec l'identifiant / mot de passe suivant : admin@admin.com / admin.

Pour s'assurer de la bonne authentification auprès de l'API, vous pouvez voir en réponse de la requête précédente, un token sous la clé `access_token`. C'est ce token que nous devrons utiliser pour authentifier les requêtes de notre application.

Afin de réutiliser facilement ce token pendant toute l'utilisation de l'application par notre utilisateur, nous allons le stocker dans le Local Storage du navigateur.

Première étape, nous allons ajouter un package NPM pour gérer le Local Storage facilement.

```bash
npm install --save ngx-webstorage
```

À la manière des modules Angular Material, nous devons inclure le package dans notre application au niveau des imports de module de notre AppModule.

```typescript
import { NgxWebstorageModule } from 'ngx-webstorage';
...
imports: [
  ...
  NgxWebstorageModule.forRoot(),
  ...
]
```

Ensuite, récupérer le fichier `token-storage.service.ts` sur le gist : https://gist.github.com/khylias/1c5eaff1f2a4f2b8bc1120552fb6795e 
C'est un petit fichier utilitaire qui nous servira pour la gestion du token. Placez ce fichier dans votre dossier `services`. Sans oublier de l'ajouter à votre AppModule.

Au retour de la méthode `connect` de notre UserService, nous pouvons maintenant stocker le token d'authentification et rediriger l'utilisateur vers le dashboard.

```typescript
login() {
  this.userService.connect(this.form.value).subscribe(response => {   
    this.tokenStorageService.setToken(response['access_token']);   
    this.router.navigate(['/dashboard']);
  })
}
```

Une fois la réponse de la requête reçue, nous pouvons retrouver le token d'accès dans notre console navigateur, onglet Application > Storage > Local Storage.

3. Gérer l'authentification automatique

L'idéal est de gérer l'authentification de toutes les requêtes automatiquement sans devoir rajouter le token dans chaque méthode. Le framework met à disposition une entité appelée Interceptor, qui est chargée d'effectuer des actions sur toutes les requêtes entrantes et sortantes de l'application avant même qu'elles n'arrivent dans les services.
 Créons notre premier interceptor.

```bash
ng g interceptor services/app
```

Vous trouverez dans votre dossier services, un fichier `app.interceptor.ts` qui utilise également le décorateur Injectable, qui implémente une autre classe : HttpInterceptor.
Dans ce service, nous trouvons une unique méthode `intercept` dans laquelle nous pourrons agir sur les requêtes de notre application.
Commençons par ajouter notre TokenStorageService afin de récupérer le token.

```typescript
constructor(private tokenStorageService: TokenStorageService) { }
```

Nous sommes en mesure d'ajouter notre token aux en-têtes des requêtes, assurons-nous de le faire dans le cas où nous avons bel et bien un token.

```typescript
if (this.tokenStorageService.hasToken()) {
  request.headers = request
    .headers
    .set('Authorization', 'Bearer ' + this.tokenStorageService.getToken());
}
```

Problème, la clé `headers` de l'objet est une propriété en lecture seule. Nous pouvons contourner cette propriété en clonant la requête.

```typescript
if (this.tokenStorageService.hasToken()) {
  let newHeaders = request.headers;
  newHeaders = newHeaders
							 .set('Authorization', 'Bearer ' + this.tokenStorageService.getToken());
  request = request.clone({ headers: newHeaders });
}
```

L'authentification de l'API utilise un token Bearer, c'est pourquoi nous ajoutons l'en-tête Authorization avec une valeur `Bearer notreToken`.

Pour que notre application utilise ce service en tant qu'intercepteur, nous devons le déclarer dans notre AppModule de la façon suivante.

```typescript
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
...
providers: [
  ...
  {
    provide: HTTP_INTERCEPTORS,
    useClass: AppInterceptor,
    multi: true,
  }
],
```

En observant les requêtes de notre application, par exemple, la récupération de la liste de clients, nous pouvons voir dans les en-têtes un nouveau paramètre Authorization suivi de notre token.

4. Les Guards

Dernière sécurité que nous devons mettre en place, la restriction d'accès à nos différentes pages de l'administration si l'utilisateur n'est pas connecté. Pour cela, nous allons utiliser ce que l'on appelle des Guards, c'est une fonctionnalité qui va nous permettre de restreindre l'accès à des routes en fonction de booléen.

Nous avons la possibilité de créer un guard directement avec la CLI.

```bash
ng g guard guards/authentication
```

Votre terminal vous demandera de choisir parmi 4 choix : canActivate, canActivateChild, canDeactivate et canLoad. Nous choissirons pour ce TP l'option canActivate.

Par défaut, la classe du guard s'initialise avec une méthode `canActivate`, exactement comme l'interceptor, c'est ici que nous allons agir sur les permissions d'accès aux routes.
Nous pouvons par exemple, vérifier que l'utilisateur est bien connecté avec la présence du token d'authentification dans son Local Storage. Pour pouvoir utiliser notre TokenStorageService dans notre guard, il nous faut y ajouter un constructeur.

```typescript
constructor(private tokenStorageService: TokenStorageService) {}
```

En examinant le TokenStorageService, nous pouvons y trouver une méthode `isAuthenticate` qui retourne un booléen. Utilisons la dans notre guard.

```typescript
canActivate(
  next: ActivatedRouteSnapshot,
  state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
  return this.tokenStorageService.isAuthenticate();
}
```

Dans ce cas-là, si l'utilisateur n'est pas authentifié, il ne pourra pas accéder à une URL protégée. Cependant, il sera simplement bloqué. L'idéal est de le diriger vers le formulaire de connexion pour qu'il comprenne pourquoi il ne peut pas se rendre sur cette URL.
En utilisant le service Router d'Angular, nous pouvons le rediriger vers le LoginComponent.

```typescript
constructor(private router: Router, private tokenStorageService: TokenStorageService) {}

canActivate(
  next: ActivatedRouteSnapshot,
  state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
  
  if (this.tokenStorageService.isAuthenticate()) {
    return true;
  }

  this.router.navigate(['/connexion']);
  return false;
}
```

Il nous reste une dernière étape: ajouter le guard à la configuration des routes. Pour cela, nous devons ajouter la clé `canActivate` dans l'objet de chaque route que nous voulons protéger, puis d'y associer une valeur étant un tableau comportant notre guard.

```typescript
{
  path: 'competences',
  canActivate: [AuthenticationGuard],
  component: SkillsComponent
},
```

*Il est bien-sûr possible d'ajouter d'autres guards pour le type CanActivate, d'où la présence d'un tableau en valeur.*

Allez dans votre console navigateur, onglet Application > Local Storage et supprimer la clé `ngx-webstorage|auth_token` puis tenter de naviguer sur une URL telle que `/clients`. Vous êtes automatiquement rediriger vers le formulaire de connexion.
Petit bémol, le menu principale est toujours visible. Pas de problème, de la même manière que dans le guard, nous allons conditionner son affichage à la bonne connexion de l'utilisateur.

```typescript
<mat-sidenav mode="side" opened *ngIf="tokenStorageService.isAuthenticate()">
  ...
</mat-sidenav>
```

Réitérez la suppresion du token dans votre navigateur, et vous constaterez le masquage du menu principale.