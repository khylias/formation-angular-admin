# Ajouter du contenu

PROBLEMATIQUE : Ajouter du contenu dynamiquement depuis l'administration

Maintenant que nous avons notre liste de clients et que nous savons comment récupérer la donnée, nous allons nous attaquer à l'ajout de client par un formulaire de création.

1. Le URL enfant

Pour le moment, nos clients ont simplement un nom. Commençons par ajouter un client avec cet unique champs.

Le futur formulaire sera sur l'URL `/clients/nouveau` avec le ClientFormComponent. Après avoir créée le composant, nous allons faire des petites modifications dans la configuration de routing. La route `/clients` étant déjà définie, nous devons ajouter la route enfant `/nouveau` avec la clé `children` de l'objet de type `Route`.

```typescript
{
  	path: 'clients',
    children: [
      {
        path: '',
        component: ClientsComponent
      },
      {
        path: 'nouveau',
        component: ClientFormComponent
      },
    ]
},
```

Ainsi, lorsque l'utilisateur accédera à l'URL `/clients`, il arrivera pas défaut sur le ClientsComponent et à l'inverse s'il navigue sur `/clients/nouveau`, il se trouvera sur notre formulaire. Sur cette dernière url vous devriez voir un message : `client-form works!`

Ajoutons un lien depuis la liste des clients pour que notre formulaire soit accessible facilement.

```html
<a routerLink="nouveau">Ajouter un client</a>
```

*Lorsque vous renseignez la route dans l'attribut `routerLink`, si vous ne mettez pas de `/` au début de la chaîne de caractère, celle-ci se concatènera à l'URL courante. Dans notre cas, étant déjà sur `/clients`, en renseignant `routerLink="nouveau"`, l'URL générée sera `/clients/nouveau`.*

2. Les champs simples

Pour utiliser un champ simple, il nous faut une variable dans notre composant pour stocker la valeur de l'utilisateur et un champ HTML.

```typescript
name = '';
```

```html
<input type="text" [(ngModel)]="name">

{{ name }} 
```

L'attribut `ngModel` est une directive du FormsModule que nous devons ajouter aux imports de notre AppModule. Elle sert à assigner et lire la valeur de la variable `name` dans le même temps.

Ajoutons la fonctionnalité de formulaire à notre application.

```typescript
import { FormsModule } from '@angular/forms';

@NgModule({
  ...
  imports: [
		... 
    FormsModule,
    ...
  ],
})
```

Nous pouvons maintenant taper une valeur dans le champ et celle-ci s'affichera juste à coté.

Poursuivons sur l'envoi de cette valeur à notre liste de clients.Avant d'ajouter le bouton de soumission du formulaire, préparons le ClientsService à mettre à jour sa liste. Créons une méthode `addClient` qui se chargera d'ajouter un item à la liste. Cette fonction va prendre en paramètre la donnée qui lui sera transmise par le composant.

```typescript
addClient(data) {
  this.clientsData.push(data);
}
```

Nous pouvons d'ors-et-déjà ajouter le type du paramètre `data` afin de s'assurer de l'homogénéité de la donnée dans le tableau `clientsData`.

```typescript
addClient(data: Client) {
  this.clientsData.push(data);
}
```

Retournons dans notre ClientFormComponent pour y ajouter notre bouton d'envoi du formulaire, celui-ci exécutera une méthode `add` de son composant au clic du bouton.

```html
<button (click)="add()">Ajouter le client</button>
```

Directement dans le composant ClientForm, il va nous falloir définir la méthode `add` et ajouter l'injection de notre ClientsService.

```typescript
constructor(private clientsService: ClientsService) { }

add() {
  // do some stuff
}
```

Il nous reste à utiliser la méthode du service que nous avons créée précédemment en lui passant la donnée de notre champ.

```typescript
this.clientsService.addClient({ name: this.name });
```

Une fois cela terminé, si vous remplissez votre champ texte, que vous cliquez sur le bouton du formulaire, en retournant sur la liste des clients, vous y trouverez une ligne supplémentaire.

2. Les formulaires plus complexes

Pour l'instant, nous avons vu qu'un champ est égale à une variable défini dans le composant. Pour des formulaires comportant plus de champs, cela ne va pas être optimal. C'est pourquoi, nous allons utiliser les Reactives Forms pour pouvoir gérer facilement des formulaire plus complexes.
Débutons par ajouter cette fonctionnalité à notre application.

```typescript
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

@NgModule({
  ...
  imports: [
		... 
    FormsModule,
    ReactiveFormsModule
    ...
  ],
})
```

De retour dans notre ClientFormComponent, nous disposons maintenant d'un service au travers du ReactiveFormsModule, qu'est le FormBuilder. C'est un service qui va nous permettre de gérer notre formulaire sous forme d'objet.

```typescript
import { FormBuilder } from '@angular/forms';
...
constructor(private clientsService: ClientsService, private fb: FormBuilder) { }
```

Ensuite, créons une constante qui stockera l'objet de notre futur formulaire, nous la typerons avec un nouveau modèle `FormGroup`.

```typescript
import { FormBuilder, FormGroup } from '@angular/forms';
...
form: FormGroup;
```

*Il existe 3 types de modèles pour les formulaires : `FormGroup` pour les objets, `FormArray` pour les tableaux et `FormControl` pour les champs simples.*

Nous allons créer un formulaire plus complexe pour enregistrer nos clients. Ajoutons les champs suivants : une description, une date et le type de projet réalisé.

Commençons par le partie basique HTML de notre formulaire.

```html
<form>
    <label for="name">Nom</label>
    <input type="text" name="name" id="name">

    <label for="description">Description</label>
    <textarea name="description" id="description" cols="30" rows="10"></textarea>

    <label for="date">Date du projet</label>
    <input type="date" name="date" id="date">

    <label for="type">Type de projet</label>
    <select name="type" id="type">
        <option value="site">Site Web</option>
        <option value="seo">Référencement</option>
        <option value="design">Webdesign</option>
    </select>

    <button (click)="add()">Ajouter le client</button>
</form>
```

Maintenant attaquons nous à l'instantiation de notre constante `form` pour contrôler ce formulaire. De manière à garder un code clair, toute action complexe doit se faire dans une méthode individuelle. `initForm` sera donc notre fonction qui exécutera l'initialisation du formulaire et nous la lancerons dès le cycle de vie `OnInit`.

```typescript
ngOnInit() {
  this.initForm();
}

initForm() {
  // do some stuff
}
```

Débutons par la racine du formulaire, le schéma global qui est un objet. Pour cela, nous utilisons le service FormBuilder précédemment ajouté.

```typescript
initForm() {
  this.form = this.fb.group({
		// some inputs
  });
}
```

*De manière identique aux modèles de formulaire, le FormBuilder (fb) permet de créer 3 types de champs : `.group({})`, `.array([])`, `.control()`*

Le premier champ est le nom du client avec la clé `name`. Il nous suffit d'ajouter cette clé à notre schéma de FormGroup.

```typescript
this.form = this.fb.group({
  name: ['']
});
```

*Bien que différente, la notation simplifiée `['']` équivaut à un `this.fb.control('')`*

Les champs suivants sont la description, la date et le type. Bien qu'ils soient de types de champs différents, la valeur attendue est unique ainsi la notation ne change pas.

```typescript
this.form = this.fb.group({
  name: [''],
  description: [''],
  date: [''],
  type: ['']
});
```

L'étape suivante consiste à "brancher" notre formulaire côté composant à celui de notre vue. Avec le ReactiveFormsModule, nous avons ajouté des attributs (des directives) possibles à nos champs, nous allons nous en servir de suite.

Tout d'abord, la balise `form` qui se voit ajouté deux attributs `formGroup` et `ngSubmit`. L'un sert à définir le modèle des champs inclus dans cette balise par rapport à notre constante, l'autre à exécuter une méthode lors de la soumission du formulaire, équivalent Angular à l'attribut natif `action`.

```html
<form [formGroup]="form" (ngSubmit)="">
<!-- some code -->
</form>
```

*Ici la notation `[]` fait référence directement à l'objet `form` de notre composant.*

Ensuite, nous devons définir les différents champs. Par exemple, pour le nom, nous ajoutons l'attribut `formControlName` qui fait référence à une clé présente dans notre objet `form`.

```html
<input type="text" formControlName="name" name="name" id="name">
```

Faites-en de même pour les autres champs.

L'action du formulaire se fera à sa soumission par l'utilisateur, c'est-à-dire à l'exécution de l'événement `ngSubmit`. Pour l'instant, nous ne lui avons pas désigné de méthode particulière. Nous pouvons donc lui assigner notre fonction `add` qui précédemment était lancé par notre bouton directement.

```html
<form [formGroup]="form" (ngSubmit)="add()">
	<button type="submit">Ajouter le client</button>
</form>
```

Il nous reste plus qu'à adapter notre méthode `add` pour envoyer le contenu de notre formulaire.

```typescript
add() {
  this.clientsService.addClient(this.form.value);
}
```

Votre client fraîchement créé se trouve à nouveau dans la liste des clients, pour l'instant seul son nom est toujours affiché, mais le reste des champs s'y trouvent également. Cependant, nous venons d'injecter un objet qui n'a pas le même schéma que notre modèle Client. Mettons-le à jour.

```typescript
export interface Client {
    name: String;
    description: String;
    date: String;
    type: String;
}
```

Dans certains cas, il est possible que nous n'ayons pas ces informations, ce qui provoquera des erreurs en console ou dans le compilateur. Afin de se prémunir de ce problème, nous pouvons définir certaines clés comme optionnelles.

```typescript
export interface Client {
    name: String;
    description?: String;
    date?: String;
    type?: String;
}
```

3. Les validateurs

En suivant le modèle Client, nous pouvons en déduire que le nom du client est obligatoire. Adaptons notre formulaire pour rendre le champs Nom obligatoire lors de la saisie. Le FormsModule met à disposition des fonctions qu'on appelle [Validators](https://angular.io/api/forms/Validators) pour valider la valeur d'un champ. Pour l'instant, nous allons simplement rendre le champ obligatoire en ajoutant une option à la définition de notre champ dans le ClientFormComponent.

```typescript
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
...
this.form = this.fb.group({
  name: ['', Validators.required],
  description: [''],
  date: [''],
  type: ['']
});
```

Vérifions en temps réel la validation de notre formulaire. Nous pouvons par exemple afficher un message d'erreur sous le champ lorsque celui-ci n'est pas rempli.

```html
<label for="name">Nom</label>
<input type="text" formControlName="name" id="name">

<span *ngIf="form.get('name').invalid">
  Le nom est obligatoire.
</span>
```

L'instance d'un champ de ReactiveForms a plusieurs attributs et méthode, dont la clé `invalid/valid` qui permet de savoir à n'importe quel moment si le champ est valide. Si vous tapez quelque chose dans le champ, vous verrez le message d'erreur disparaître.

En se référant à la documentation, il existe d'autres validateurs: minLength, maxLength, min, max, email, etc... Nous allons utiliser le validateur minLength pour imposer une longueur minimale au nom d'un client. Il est possible d'assigner plusieurs validateurs à un même champs, sous forme de tableau.

```typescript
name: ['', [Validators.required, Validators.minLength(2)]],
```

Ainsi configurer, le nom du client devra avoir au minmum 2 caractères pour être valide. Ajoutons le message d'erreur corresponds à ce nouveau validateur.

```html
<span *ngIf="form.get('name').invalid">
  Le nom doit comporter plus de 2 caractères.
</span>
```

Le test des ngIf sur la même propriété `invalid` n'a plus vraiment de sens lorsque le champ a plusieurs validateurs. Nous pouvons faire la distinction des erreurs avec la clé `errors` de notre champ `name`.
Pour nous aider à y voir plus clair, nous pouvons temporairement afficher les valeurs de cette clé dans notre template pour comprendre ce qui s'y passe.

```html
{{ form.get('name').errors | json }}
```

En premier lieu, cette interprétation HTML affichera un objet `{ "required": true }` ce qui fait référence à notre validateur `.required`. Dans un second temps, si nous tapons un premier et unique caractère, nous verrons le résultat HTML changé pour un objet identifiant le validateur `.minLength` : `{ "minlength": { "requiredLength": 2, "actualLength": 1 } }`
En conclusion, nous pouvons faire la distinction des validateurs et donc des messages d'erreurs à afficher en fonction des clés présentes dans l'attribut `errors`. Nous avons à disposition une méthode `hasError()` qui permet de faire ce test.

```html
<span *ngIf="form.get('name').hasError('required')">
  Le nom est obligatoire.
</span>
<span *ngIf="form.get('name').hasError('minlength')">
  La description doit comporter plus de 2 caractères.
</span>
```

En fonction des cas, nous avons les différents messages qui s'affichent bien. 

Petit bémol, lorsque nous arrivons sur la page, le message d'erreur requis est déjà affiché. Or nous ne voulons pas déjà afficher des erreurs à l'utilisateur alors qu'il n'a pas encore commencé à remplir son formulaire. C'est pourquoi, nous allons de manière générale, encapsuler nos messages d'erreurs dans un ngIf plus haut qui vérifiera si l'utilisateur a rempli le formulaire sans ce champs là ou bien qu'il ai effacé sa saisie.

```html
<p *ngIf="form.dirty || form.touched">
  <span *ngIf="form.get('name').hasError('required')">
    Le nom est obligatoire.
  </span>
  <span *ngIf="form.get('name').hasError('minlength')">
    La description doit comporter plus de 2 caractères.
  </span>
</p>
```

4. Composants de formulaire

Avec l'aide d'Angular Material et de sa documentation, nous allons stylisé notre formulaire. Commençons par les champs simples. Ajoutons le module à notre application, nous pourrons ensuite l'utiliser dans notre vue.

```typescript
import { MatInputModule } from '@angular/material/input';
...
imports: [
  ...
  MatSidenavModule,
  MatListModule,
  MatInputModule
],
```

Dans les exemples de la documentation, ils englobent chaque champs par une balise `mat-form-field`, nous allons suivre la même démarche et ainsi importer ce module également. Ce module nous permettra de gérer le style d'affichage des erreurs simplement.

```typescript
import { MatFormFieldModule } from '@angular/material/form-field';
...
imports: [
  ...
  MatSidenavModule,
  MatListModule,
  MatInputModule,
  MatFormFieldModule
],
```

Modifions notre formulaire HTML avec les différents composants de la librairie.

```html
<mat-form-field>
  <mat-label for="name">Nom</mat-label>
  <input type="text" matInput formControlName="name" id="name">
</mat-form-field>
```

Petit particularité sur la balise `input` qui se voit attribué une directive `matInput` au lieu d'un nouveau balisage. Toute suite, notre champ a un meilleur rendu visuel. Faites-en de même pour la description.

Pour l'affichage des erreurs, en suivant la documentation, nous pouvons remarquer qu'il y a également un sous-composant `mat-error` qui s'intègre dans le `mat-form-field`.

```html
<mat-error *ngIf="form.dirty || form.touched">
  <span *ngIf="form.get('name').hasError('required')">
    Le nom est obligatoire.
  </span>
  <span *ngIf="form.get('name').hasError('minlength')">
    La description doit comporter plus de 2 caractères.
  </span>
</mat-error>
```

Petit plus du composant MatError, il gère de base l'affichage ou non de son contenu. Nous pouvons donc retirer le ngIf présent sur la balise.

Attaquons nous aux autres champs. Avec un rapide coup d'oeil à la documentation, Angular Material propose un composant pour le champ date, Datepicker, et un composant pour le select, Select.
Même procédure que les composants précédants, nous ajoutons l'import du module dans notre application puis nous pouvons utiliser leur balise dans notre vue.

```typescript
import { MatSelectModule } from '@angular/material/select';
import { MatDatepickerModule } from '@angular/material/datepicker';
import { MatNativeDateModule } from '@angular/material/core';
...
imports: [
  ...
  MatSidenavModule,
  MatListModule,
  MatInputModule,
  MatFormFieldModule,
  MatSelectModule,
  MatDatepickerModule,
  MatNativeDateModule
],
```

*Petite spécificité pour le MatDatepickerModule qui a besoin du MatNativeDateModule pour fonctionner.*

Pour le champ date, adaptons-le déjà avec le MatFormField.

```html
<mat-form-field>
  <mat-label for="date">Date du projet</mat-label>
  <input matInput formControlName="date" name="date" id="date">
</mat-form-field>
```

Puis nous pouvons y ajouter le composant du datepicker.

```html
<mat-form-field>
  <mat-label for="date">Date du projet</mat-label>
  <input matInput formControlName="date" name="date" id="date" [matDatepicker]="picker">
  <mat-datepicker-toggle matSuffix [for]="picker"></mat-datepicker-toggle>
  <mat-datepicker #picker></mat-datepicker>
</mat-form-field>
```

L'attribut `#picker` de la balise `mat-datepicker` sert à récupérer cette balise HTML et son contenu comme un sélecteur JS classique tel que `.getElementById()`. 

Notre champ date est maintenant pourvu d'un sélecteur de date que vous pouvez ouvrir en cliquant sur l'icône.

Au tour du sélecteur de type d'être mis à jour, le module offre deux composants : `mat-select` et `mat-option` .

```html
<mat-form-field>
  <mat-label for="type">Type de projet</mat-label>
  <mat-select formControlName="type" name="type" id="type">
    <mat-option value="site">Site Web</mat-option>
    <mat-option value="seo">Référencement</mat-option>
    <mat-option value="design">Webdesign</mat-option>
  </mat-select>
</mat-form-field>
```

Et enfin, le composant que nous utiliserons le plus probablement, le bouton de formulaire.

```typescript
import {MatButtonModule} from '@angular/material/button';
...
imports: [
  ...
  MatSidenavModule,
  MatListModule,
  MatInputModule,
  MatFormFieldModule,
  MatSelectModule,
  MatDatepickerModule,
  MatNativeDateModule,
  MatButtonModule
],
```

Dans la documentation et les exemples du ButtonModule, vous pouvez y trouver différents styles de bouton : raised, flat, classic, stroke, etc... Prenez celui qui vous convient.

```html
<button mat-raised-button color="primary">Primary</button>
```

Ici, point de MatFormField, car le bouton n'est pas un champ à proprement parler du formulaire. Il nous suffit d'ajouter la directive du style de notre choix et d'y associer l'attribut `color` voulu. Nous avons le choix entre plusieurs couleurs du thème : `primary`,  `accent`, `warn`, le style du bouton désactivé et aucune couleur définie.