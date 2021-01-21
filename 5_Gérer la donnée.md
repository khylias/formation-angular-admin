# Gérer la donnée

PROBLEMATIQUE : Unicité et partage de la donnée dans l'app

1. Les services

Vous l'aurez remarqué, nous avons dupliquer les tableaux de Clients et Skills pour qu'ils s'affichent sur le Dashboard et sur leur page respective, ce qui n'est pas du tout optimal. Pour remédier à ça et gérer efficacement la donnée, nous utiliserons les services dans notre application. Un service est une classe avec un décorateur particulier, `@Injectable`, qui la définit comme réutilisable dans les autres composants.

Commençons par créer notre premier service avec la CLI.

```bash
ng g service services/clients
```

Nous retrouverons notre ClientsService dans un dossier `services` dans lequel nous les stockerons.

En premier lieu, nous devons instancier notre service dans notre AppModule dans le tableau `providers`.

```typescript
@NgModule({
  ...
  providers: [
      ClientsService
  ],
})
```

Ensuite, dans le composant où nous souhaitons utiliser le service, dans notre cas le ClientsComponent, nous pouvons injecter le ClientsService.

```typescript
import { ClientsService } from '../services/clients.service';
... 
export class ClientsComponent implements OnInit {
	constructor(private clientsService: ClientsService) { }
	...
}
```

*L'attribut `private` peut également être `public`, il définit le contexte dans lequel peut être utiliser le service. Private pour uniquement le composant, public pour le composant et la vue HTML.*

*La bonne pratique veut qu'on défini la référence du service avec la même dénomination*

Maintenant, nous allons instancier la variable `clients` dans notre ClientsComponent comme étant un tableau vide afin de confier la gestion de donnée au service. Ce qui veut dire que nous pouvons commencer par préparer le retour de la donnée au travers du service.
Nous allons ajouter une méthode dédiée à cette tâche dans notre ClientsComponent.

```typescript
export class ClientsComponent implements OnInit {
    clients = [];
    constructor(private clientsService: ClientsService) { }

    ngOnInit(): void {
    }

    getClients() {
        // do some stuff
    }

}
```

Dans cette méthode `getClients`, nous allons assigné le retour de la future fonction de notre service à notre tableau de clients.

```
getClients() {
	this.clients = this.clientsService.getClients();
}
```

Dernière étape dans notre composant, exécuter la méthode pour récupérer la donnée. Pour cela, nous allons nous service du cycle de vie ([lifecycle hooks](https://angular.io/guide/lifecycle-hooks)) `ngOnInit` afin de lancer la fonction à l'initialisation du composant.

```typescript
ngOnInit(): void {
  this.getClients();
}
```

Votre IDE vous signale une erreur sur la méthode `clientsService.getClients`, pas de soucis, nous allons rectifier tout de suite en nous dirigeant vers notre ClientsService. Nous allons créer une variable qui stocke tous les clients nécessaires dans l'application et la méthode `getClients` qui est chargée de retourner notre tableau.

```typescript
export class ClientsService {

    clientsData = [
        {
            name: 'La Cantiine',
        },
        {
            name: 'Aquarium de La Rochelle',
        },
        {
            name: 'Le Bathyscaphe',
        }
    ];

    constructor() { }

    getClients() {
        return this.clientsData;
    }
}
```

Notre ClientsComponent n'est désormais plus en erreur ! Nous pouvons maintenant harmoniser les sources de données des clients entre le ClientsComponent et le Dashboard. Faites la même procédure pour le Dashboard.

2. Les modèles de données

Ce découpage de la logique de l'application entre des composants, des services et les vues apportent de l'abstraction quant à la donnée que l'on manipule tout au long du développement. Nous pouvons réduire cet aspect en ajoutant des interfaces, c'est-à-dire des modèles pour définir le schéma de donnée.
Commençons par le modèle Client avec la CLI.

```bash
ng g interface models/client
```

La notation au singulier est voulue car nous définirrons le schéma de donnée d'un client.

Dans cette interface, pour définir le modèle de donnée, il nous suffit d'indiquer les clés de l'objet Client ainsi que leur type: String, Number, Object, Array, ...

```typescript
export interface Client {
    name: String;
}
```

Désormais, toute variable étant typé par l'interface `Client` devra être un objet avec une clé `name`.

"Typer" une variable, c'est ajouter un attribut à une variable lors de sa définition. Prenons par exemple, le ClientsComponent et sa variable `clients`. *N'oublions pas l'import en haut de fichier*

```typescript
clients: Client = [];
```

Cependant, dans ce cas-là, nous avons besoin que `clients` soit un tableau de Client et non un seul objet.
Adaptons la syntaxe pour typer notre variable comme un tableau de Client.

 ```typescript
clients: Client[] = [];
 ```

Lors de nos prochains développements dans le ClientsComponent, nous serons certains d'avoir un tableau d'object Client.

La prochaine étape est de typer le tableau `clients` et le retour de la méthode `getClients` dans notre ClientsServices.

```typescript
clientsData: Client = [...]
```

Un retour de méthode peut être typer avec TypeScript, ce qui nous permet de connaître à l'avance le schéma de données renvoyé par cette fonction, ici un tableau de clients.

```typescript
getClients(): Client[] {
  return this.clientsData;
}
```

Faites-en de même pour le DashboardComponent.

Plus tard, nous verrons comment récupérer de la donnée depuis une API à travers ces services.