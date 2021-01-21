# Envoyer ses données

PROBLEMATIQUE : Enregistrer ses données dans une base de donnée

Pour l'instant, toutes nos données sont stockées en dur dans l'application. Il est temps de les dynamiser et de faire appel à une API externe pour stocker durablement nos données.

1. Récupérer l'API

Pour ce TP, nous allons intéragir avec une API que vous hébergerez en local sur votre machine. Récupérez-là sur le gitlab de l'Université.

```bash
git clone https://github.com/khylias/apimiaw.git
```

Ensuite, rendez-vous dans le dossier du projet et installez les dépendances.

```bash
cd api-miaw && npm install
```

Et enfin, lancez le script de l'API.

```bash
npm run start
```

Vous pourrez maintenant accéder à l'API par l'URL : `localhost:3000`.

2. Créer un service Http

Pour pouvoir faire appel à notre API depuis notre application, nous devons dans un premier temps ajouter le module [HttpClient](https://angular.io/api/common/http/HttpClientModule).

```typescript
import { HttpClientModule } from '@angular/common/http';
...
imports: [
  ...
  HttpClientModule,
],
```

Ensuite, nous allons ajouter le service [HttpClient](https://angular.io/api/common/http/HttpClient) qui est rendu disponible par ce module dans notre service Clients.

```typescript
import { HttpClient } from '@angular/common/http';
...
constructor(private httpClient: HttpClient) { }
```

3. Récupérer les données

Nous pouvons maintenant utiliser le service HttpClient pour faire appel à notre API et récupérer les informations de nos clients. En suivant la documentation du service, vous pouvez remarquer qu'il expose plusieurs méthodes, chacune correspondante à un type d'appel HTTP : `.get()`, `.post()`, `.put()`, `.delete()`.
Dans notre premier cas, nous allons utiliser la méthode `.get()`.

```typescript
getClients(): Client[] {
  return this.httpClient.get('http://localhost:3000/clients');
}
```

Votre IDE devrait vous signaler une erreur sur le type de la fonction qui est retourné. En effet, il s'agit maintenant d'une [Observable](https://angular.io/guide/observables-in-angular). 

```typescript
import { Observable } from 'rxjs';
...
getClients(): Observable<any> {
  return this.httpClient.get('http://localhost:3000/clients');
}
```

La prochaine étape consiste à mettre à jour l'utilisation de notre méthode `getClients ` dans les différents composants où elle est utilisé.

Une observable est un flux de donnée ouvert, pour récupérer son contenu, nous devons y "souscrire" et ainsi à chaque événement envoyé dans ce flux, nous pourrons récupérer les données. Commençons par le dashboard.

Pour rappel, nous avons un appel à nos clients comme ceci.

```typescript
getClients() {
  this.clients = this.clientsService.getClients();
}
```

Notre ClientsService est déjà instancié dans ce composant, il nous suffit donc de modifier l'appel à `.getClients()` pour récupérer les données de l'API.

```typescript
getClients() {
  this.clientsService.getClients().subscribe(response => {
    this.clients = response;
  });
}
```

Pour l'instant, si nous nous rendons sur le dashboard, nous n'avons plus aucun client dans notre liste. Cependant, en utilisant l'inspecteur et l'onglet "Network", nous pouvons bien voir l'appel à l'API, qui renvoit en réponse un tableau vide.

3. Envoyer les données

Nous savons comment recevoir nos données, maintenant nous allons faire le chemin inverse et envoyer des données à l'API. Notre formulaire de création de client s'y prête très bien.
Nous avions déjà prévu notre méthode `addClient` de notre service. Il nous suffit donc de modifier cette méthode pour qu'elle envoit les données à l'API et non les ajouter au tableau local.

```typescript
addClient(data: Client) {
  return this.httpClient.post('http://localhost:3000/clients', data);
}
```

En suivant la documentation, nous pouvons voir que le contenu de la requête doit être passé en deuxième paramètre de la méthode `post`. Il ne reste plus qu'à souscrire au retour de la requête dans notre composant.

```typescript
add() {
  this.clientsService.addClient(this.form.value).subscribe(response => {
    // do some stuff
  });
}
```

À la réponse de la reqûete, nous pouvons faire ce que l'on souhaite. Dans ce cas-là, il serait judicieux de retourner l'utilisateur sur la page des Clients.

Afin de renvoyez l'utilisateur vers une autre route, nous avons besoin d'injecter le service Router à notre composant.

```typescript
import { Router } from '@angular/router';
...
constructor(private clientsService: ClientsService, 
            private fb: FormBuilder, 
            private router: Router) { }
```

Nous pouvons ainsi utiliser la méthode `navigate` du service.

```typescript
this.clientsService.addClient(this.form.value).subscribe(response => {
  this.router.navigate(['/clients']);
});
```

Remplissez votre formulaire, et vous serez ainsi redirigé vers la page des Clients. Comme nous avons déjà adapté la réception des données clients, vous y trouverez votre client fraîchement créé.