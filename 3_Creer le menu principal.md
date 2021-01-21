# Créer le menu principal

PROBLEMATIQUE : Ajouter le menu principale pour naviguer dans les pages de l'admin

1. Le routing dans Angular

Angular embarque un module pour créer une navigation dans notre application. Nous pouvons ajouter ce module en utilisant la CLI et sa commande `ng generate` .

```bash
ng generate module app-routing --flat --module=app
```

- ng generate : le nom de la commande
- module : le type d'entité que l'on veut créer (https://angular.io/cli/generate)
- app-routing : le nom de l'entité générée
- --flat --module=app : des options à la commande de base generate.

Nous nous retrouvons donc avec un fichier `app-routing.module.ts` à la racine de notre `/app`. C'est ici que nous configurerons la navigation dans notre admin.
Commençons par créer le tableau qui stockera toutes nos routes.

```typescript
const routes: Routes = [];
```

`Routes` fait référence à l'interface du module de routing d'Angular. Très certainement, votre IDE vous signale une erreur à son niveau. Pour régler ce problème, il nous faut définir l'interface Routes dans notre module.

```typescript
import { Routes } from '@angular/router';
```

*De nombreux IDE vous propose l'auto-import de références en autocomplétion* 

Plus bas dans notre fichier, nous pouvons trouver le décorateur `@NgModule` avec des paramètres. Nous allons ajouter le module de Router d'Angular.

```typescript
@NgModule({
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule]
})
```

Sans oublier l'import du `RouterModule` également.

```typescript
import { RouterModule, Routes } from '@angular/router';
```

Dernière étape pour notre configuration de notre navigation de l'application, l'importer dans le module principale. Pour cela, nous l'ajoutons dans les imports de notre `app.module.ts`

```typescript
import { AppRoutingModule } from './app-routing.module';
...

@NgModule({
  ...
  imports: [
      AppRoutingModule,
  ]
})
```



2. Déclarer nos routes

Nous allons utiliser la variable `routes` pour définir les différentes URL de notre admin. Un bon début serai de préparer les URLs pour les listes complètes de nos compétences et de nos clients.

```typescript
const routes: Routes = [
    {
        path: 'competences',
    },
    {
        path: 'clients'
    }
];
```

3. Créer les composants Clients et Skills.

Les routes que nous avons définis précédemment doivent être assigner à un composant afin d'afficher ce dernier lorsque l'utilisateur se rends sur l'URL. Utilisons à nouveau la CLI pour générer nos composants automatiquement.

```bash
ng generate component skills && ng generate component clients
```

Nous avons désormais deux nouveaux dossiers qui se sont ajoutés à notre dossier app, skills et clients. Ces deux dossiers comportent les différents fichiers nécessaires aux composants.

- Un fichier `.component.ts` pour la partie JavaScript/TypeScript du composant
- Un fichier `.html` pour le template
- Un fichier `.scss` ou `.css` pour le style
- Un fichier `.spec` pour les tests

Maintenant que nous avons nos composants de destinations créées, nous pouvons les assigner aux routes.

```typescript
{
  path: 'competences',
  component: SkillsComponent
},
{
  path: 'clients',
  component: ClientsComponent
}
```

De la même manière que l'interface Routes, nous devons définir le chemin des composants dans notre `app-routing`.

```typescript
import { ClientsComponent } from './clients/clients.component';
import { SkillsComponent } from './skills/skills.component';
```

Par défaut, lors de la création d'un composant ou d'un module, la CLI va importer la classe dans votre AppModule.

```typescript
@NgModule({
  declarations: [
    AppComponent,
    SkillsComponent,
    ClientsComponent
  ],
  ...
})
```

4. La vue du router

Nous avons les routes, nous avons les composants, il nous faut désormais définir l'emplacement du router, l'espace où la navigation va s'effectuer.

A la suite de notre `app.component.html`, nous allons ajouter une balise spéciale du RouterModule.

```html
<router-outlet></router-outlet>
```

En nous rendant sur l'url http://localhost:4200/clients par exemple, nous y verrons maintenant le contenu de nos listes et un texte `clients works!`. Ce texte vient de la vue de notre ClientsComponent.

5. Créer des liens

Maintenant que nous avons nos URLs et nos composants, nous pouvons faire notre menu avec les différents liens qui mènent à nos listes de clients et compétences.

Pour déclarer des liens qui prennent en compte le routing d'Angular, il nous faut utiliser l'attribut `routerLink` à la place du traditionnel `href`

```html
<header>
    <nav>
        <ul>
            <li>
                <a routerLink="/clients">Clients</a>
            </li>
        </ul>
    </nav>
</header>
```

Faites-en de même pour les compétences.

Nous pouvons maintenant naviguer d'une page à l'autre sur notre application. Mais nous remarquons que le contenu, le mot de bienvenue, la liste des clients et des compétences, est persistant entre les pages.

Pour régler ce problème, nous allons faire de notre dashboard, un composant à part entière. Comme les étapes précédentes, créez un composant Dashboard, avec sa route personnalisée `/dashboard` et mettez-y le mot de bienvenue et nos listes récapitulatives.



À ce stade, nous devrions avoir un menu de navigation avec 3 liens, renvoyant vers nos 3 pages : le dashboard, la liste des clients et la liste des compétences.