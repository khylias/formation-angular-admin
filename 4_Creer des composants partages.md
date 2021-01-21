# Créer des composants partagés

PROBLEMATIQUE : Utiliser des composants internes et externes

1. Les composants internes

Dans notre nouvelle page des clients, nous allons afficher la liste complète des clients de notre agence. Commençons par créer une variable qui stock nos clients dans le `clients.component.ts`.

```typescript
clients = [
	// Les objets Client
];
```

Côté HTML, nous allons reprendre le même HTML que nous avions sur le dashboard. Lorsque nous avons du code qui doit être dupliquer ainsi, il peut être astucieux de mutualiser le code en créant un composant intermédiaire qui s'occupera d'afficher un client. Attaquons ça de suite.

Avec la CLI, créons un composant ClientItem.

```bash
ng g component shared/client-item
```

*NB: nous mettrons ce composant dans un dossier `shared` car il sera utilisé communément par plusieurs composants parents* 

Nous pouvons d'ors-et-déjà injecter ce composant dans nos listes du dashboard et de la page des clients. Rien de plus simple, prenons la valeur de la clé `selector` dans le décorateur de notre ClientItemComponent et utilisons la comme une balise HTML.

```html
<ul>
    <li *ngFor="let client of clients">
        {{ client.name }}
        <app-client-item></app-client-item>
    </li>
</ul>
```

Pour chaque ligne client, nous verrons un texte `client-item works!`. Faites en de même sur le dashboard et la page des clients.

Transférons le nom de client dans notre vue du ClientItemComponent, en effaçant celui de la liste.

```html
{{ client.name }}
```

Votre IDE vous signalera une erreur comme quoi `client` est indéfini. Ayant changé de contexte de composant, pour l'instant la variable `client` n'est pas présente dans notre ClientItemComponent.

Pour récupérer notre variable, nous allons utiliser l'option `Input` pour transmettre la donnée à du composant parent au composant enfant.

Rendez-vous dans le ClientItemComponent, créons une variable `client`, préfixé par un décorateur.

```typescript
@Input() client = {
  name: ''
};
```

Ce décorateur `@Input` nécessite l'ajout de l'import dans la première ligne du composant.

```typescript
import { Component, Input, OnInit } from '@angular/core';
```

Maintenant, votre IDE ne devrait plus vous signaler d'erreur concernant la variable `client` de notre vue. Cependant, sur notre navigateur, rien ne s'affiche. Il nous reste une étape, celle de transmettre du composant parent, c'est-à-dire le DashboardComponent ou le ClientsComponent, la variable `client` à notre ClientItemComponent. 
Cela se passe au niveau de la balise HTML, nous devons ajouter un attribut `client`.

```html
 <li *ngFor="let client of clients">
   <app-client-item [client]="client"></app-client-item>
</li>
```

L'attribut `[client]` représente notre variable "Input" du ClientItemComponent et `"client"` corresponds à notre itération courante du `ngFor`.

Faites en de même avec le ClientsComponent.

Dorénavant, lorsque vous ferez une modification sur le template d'un client, elle sera présente sur toutes les listes des clients. 👍

2. Les composants externes

Pour l'instant, nous avons donc vu deux types d'utilisations de composant, en page indépendante et en injection directe dans le template. Nous pouvons maintenant utiliser des composants de librairies externes pour nous aider dans notre développement.

Nous allons dans le cadre de ce projet utiliser la librairie [Angular Material](https://material.angular.io/) pour nous fournir des solutions de composants clés en main. 

Suivons le Getting Started d'Angular Material pour l'ajouter à notre projet.

```bash
ng add @angular/material
```

La commande va nous demander de choisir un thème par 4 propositions, que vous pouvez retrouver sur https://material.angular.io/guide/theming#using-a-pre-built-theme et en prévisualisation sur l'icône haut à droite du site. Vous pouvez également accepter les configurations typographiques et les animations.

Nous allons pouvoir utiliser les composants d'Angular Material. Mettons toute suite en forme notre menu principale, rendez-vous sur la documentation du [Sidenav](https://material.angular.io/components/sidenav/overview). Pour commencer, dans l'onglet API, prenez la ligne d'import du composant et ajoutons-le dans notre AppModule. 

En ajoutant le MatSidenavModule dans les imports de notre AppModule, nous serons en mesure d'utiliser la Sidenav partout dans notre application.

```typescript
@NgModule({
  ...
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    MatSidenavModule
  ],
	...
})
```

Avec l'onglet Examples, nous pouvons avoir un aperçu des différentes possibilités avec ce composant. Pour l'instant nous allons l'utiliser de manière basique, c'est-à-dire, d'avoir notre menu principale dans cette sidenav.

En suivant la documentation du SidenavModule, nous voyons qu'il faut 3 balises HTML spécifiques pour obtenir la sidenav voulu. Nous la mettrons en place dans notre vue du AppComponent.

Nous devons en premier lieu encapsuler notre menu et notre contenu dans la balise `mat-sidenav-container`

```html
<mat-sidenav-container>
  <header>
    <nav>
      <ul>
        <li>
          <a routerLink="/dashboard" routerLinkActive="router-link-active">Dashboard</a>
        </li>
        <li>
          <a routerLink="/clients" routerLinkActive="router-link-active">Clients</a>
        </li>
        <li>
          <a routerLink="/competences" routerLinkActive="router-link-active">Compétences</a>
        </li>
      </ul>
    </nav>
  </header>
  <router-outlet></router-outlet>
</mat-sidenav-container>
```

Puis respectivement, le header avec la balise `mat-sidenav` et notre router par la balise `mat-sidenav-content`.

```html
<mat-sidenav-container>
    <mat-sidenav>
        <header>
            <nav>
                <ul>
                    <li>
                        <a routerLink="/dashboard" routerLinkActive="router-link-active">Dashboard</a>
                    </li>
                    <li>
                        <a routerLink="/clients" routerLinkActive="router-link-active">Clients</a>
                    </li>
                    <li>
                        <a routerLink="/competences" routerLinkActive="router-link-active">Compétences</a>
                    </li>
                </ul>
            </nav>
        </header>
    </mat-sidenav>
    <mat-sidenav-content>
        <router-outlet></router-outlet>
    </mat-sidenav-content>
</mat-sidenav-container>
```

Si on regarde dans notre navigateur, le menu a disparu... En suivant la documentation, nous pouvons remarquer qu'il y a des options pour la sidenav pour afficher en permanence le menu.

```html
<mat-sidenav mode="side" opened>
</mat-sidenav>
```

Notre menu et notre contenu s'affichent bien côte à côte, mais n'ayant pas encore beaucoup de contenu, le container global peut être tout petit. Ajoutons un peu de CSS pour fixer cela dans notre `app.component.css/scss`

```scss
mat-sidenav-container {
    height: 100%;
}
```

En tenant compte de la documentation Angular Material, remplacez les balises `ul` et `li` de notre menu par le composant List.