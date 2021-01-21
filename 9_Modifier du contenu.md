# Modifier du contenu

PROBLEMATIQUE : Mettre à jour un client

1. Accéder à une page d'un client

Avant de pouvoir modifier les informations d'un client, nous allons créer une étape intermédiaire qui permettra de voir les détails du client. Créez un nouveau composant Client, puis dans le AppRoutingModule, nous allons ajouter un objet de configuration qui sera enfant de la route `clients`

```typescript
{
  path: ':id',
  component: ClientComponent
}
```

*La notation `:id` signifie que le paramètre après les `:` sont dynamiques. Notre URL ressemblera donc à ceci : `localhost:4200/clients/12345`*

Dans notre ClientItemComponent, nous allons ajouter un lien "Voir" afin de nous rendre sur la page de détail qu'est notre ClientComponent.

```html
<a mat-raised-button color="default" [routerLink]="['../clients', client.id]">Voir</a>
```

La clé `id` de notre variable `client` sera inconnue pour l'instant. Il nous faut modifier le type de la constante dans notre composant par l'interface Client.

```typescript
@Input() client: Client;
```

Nous allons également ajouter la notion d'id dans l'interface du Client. Id qui sera généré automatiquement par l'API à la création du client.

Dès lors, nous avons un accès à une page dédiée à un client. Récupérons ses informations auprès de l'API.

Commençons par créer la méthode dans notre ClientsService.

```typescript
getClient(id: string): Observable<any> {
  return this.httpClient.get('http://localhost:3000/clients/' + id);
}
```

Puis utilisons là dans notre ClientComponent.

```typescript
getData() {
  this.clientsService.getClient().subscribe(response => {
    this.client = response;
  });
}
```

Il nous manque simplement de passer l'id du client à notre service. Id que nous avons à disposition dans l'URL, nous pouvons donc le récupérer facilement avec le router.

```typescript
import { ActivatedRoute } from '@angular/router';
...
constructor(private clientsService: ClientsService, private route: ActivatedRoute) { }
```

*Le service ActivatedRoute permet de récupérer toutes les informations relatives à la route courante.*

Une fois instancié, nous pouvons le récupérer de la façon suivante.

```typescript
getData() {
  this.clientsService
    .getClient(this.route.snapshot.paramMap.get('id'))
    .subscribe(response => {
    	this.client = response;
  });
}
```

Nous sommes en mesure désormais de récupérer les informations d'un client sur sa page de détails. Mettez les différentes informations en forme avec les ressources d'Angular Material. Par exemple avec le composant Card.

2. Le formulaire

Attaquons la partie qui nous intéresse, la modification des données du client. Sur la page du client, nous allons ajouter un lien qui pointera directement vers le formulaire pré-rempli des informations déjà existantes.

Débutons par créer une nouvelle route qui servira uniquement dans le cas de l'édition d'un client. Cette route sera également enfant de la route primaire `clients` et pointera vers notre ClientFormComponent car nous y avons déjà fait le formulaire, nous pouvons le réutiliser.

```typescript
{
  path: ':id/edit',
 	component: ClientFormComponent,
  data: {
    edit: true
  }
},
```

*Nous pouvons reprendre la notation `:id` et en y rajoutant `/edit`, le router saura faire la différence lorsque l'utilisateur souhaitera se rendre sur la fiche client ou le formulaire d'édition.*

L'attribut `data` permet d'ajouter des données personnalisées, dans notre cas, nous ajoutons une clé `edit` qui, une fois testée dans notre ClientFormComponent, permettra de déterminer si le formulaire doit être utiliser en création ou en modification.

Prochaine étape, récupérer cette clé `edit` dans notre composant de formulaire.

```typescript
constructor(... private route: ActivatedRoute) {}

ngOnInit() {
  console.log(this.route.snapshot.data.edit);
}
```

Sur la route `clients/nouveau`, le retour de votre console doit être `undefined` et `true` sur celle de la modification d'un client. Nous allons donc pouvoir en fonction de son état, récupérer les informations et pré-remplir le formulaire.
Comme pour la page du Client, nous récupérons ses informations, à la différence qu'à la réponse de la requête nous assignons les données au formulaire.

```typescript
getData() {
  this.clientsService
    .getClient(this.route.snapshot.paramMap.get('id'))
    .subscribe(response => {
    	this.form.patchValue(response);
  });
}
```

Nous devons être bien attentif à assigner les valeurs au formulaire, une fois que ce dernier est instancié. C'est pourquoi nous ferons l'appel à l'API après l'initialisation.

```typescript
ngOnInit(): void {
  this.initForm();
	if(this.route.snapshot.data.edit) {
  	this.getData();
	}
}
```

Pour le détail, changez le label de bouton d'envoi de `Ajouter le client` à `Modifier le client`.

Dernière modification à apporter au formulaire: la soumission du formulaire ne doit pas entraîner une création. Préparons le service pour faire le bon appel à l'API. Cette fois-ci, il s'agit d'une requête de type PUT.

```typescript
updateClient(id, client: Client): Observable<any> {
  return this.httpClient.put('http://localhost:3000/clients/' + id, client);
}
```

Ensuite, dans notre ClientFormComponent, exécutez la méthode `addClient` ou `updateClient` dans le bon contexte.

Nous sommes au point de pouvoir créer et modifier une donnée, il nous reste une dernière petite étape: la suppression. Comme précédémment, tout débute dans le service avec un appel à l'API.

```typescript
deleteClient(id): Observable<any> {
  return this.httpClient.delete('http://localhost:3000/clients/' + id);
}
```

Donnons la possibilité à l'utilisateur de supprimer le client directement depuis la fiche. Nous allons ajouter un bouton sur la page. Avec l'event `(click)`, nous sommes en mesure de récupérer le click de l'utilisateur sur ce bouton et ainsi exécuter la méthode `deleteClient`. Créez cette méthode dans votre ClientComponent.

```html
<button mat-button (click)="deleteClient()">
  <mat-icon>delete</mat-icon>
</button>
```

Une fois que l'API a répondu, redirigeons l'utilisateur sur la liste des clients.

```typescript
this.router.navigate(['/clients']);
```



Félicitations 👏 
Vous avez désormais une administration sécurisée pour gérer vos clients ! 
Vous avez pu avoir un ensemble des principes de bases du framework Angular et d'une première approche. Il reste bien des détails, fonctionnalités ou des améliorations à y apporter. C'est ce que vous pouvez appréhender dans les TPs suivants.