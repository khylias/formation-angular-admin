# Créer le dashboard

PROBLEMATIQUE : Créer le dashboard d'accueil de l'admin

1. Enlever le tutoriel

   Par défaut, le fichier `app.component.html` a un contenu pour vous guider dans vos premiers pas sur Angular. Vous pouvez supprimer ce code, il servira de point d'entrée de votre application, votre dashboard.

2. Afficher le titre Dashboard

   Dans votre `app.component.ts` , ajoutez une variable `title` qui a pour valeur "Dashboard".

   ```typescript
   export class AppComponent {
     title = 'Dashboard';
   }
   ```

   Vous pouvez afficher cette variable dans votre template HTML à l'aide des accolades `{{ }}` : 

   ```html
   <h1>
     {{ title }}
   </h1>
   ```

3. Ajouter un message de bienvenue

   Sous le titre, ajoutons message de bienvenue à l'administrateur. Commençons par créer une variable qui stocke les  informations de l'utilisateur.

   ```typescript
   user = {
     name: 'Jean'
   };
   ```

   Puis nous pouvons lui souhaiter la bienvenue dans le template HTML.

   ```html
   <p>
     Bienvenue {{ user.name }}, que faisons-nous aujourd'hui ?
   </p>
   ```

4. Afficher la liste des prestations de l'agence

   Pour que le futur client vous choississe en tant que prestataire, il doit avoir connaissance de votre savoir-faire et de vos prestations. Nous allons afficher sur le dashboard, les prestations déjà existantes dans l'admin.

   Créer une variable qui stocke sous forme de tableau toutes les prestations.

   ```typescript
   skills = [
     {
       name: 'Développement WordPress',
      	logo: 'wordpress.png'
     },
   	{
       name: 'Développement Front-end',
       logo: null
     },
     {
       name: 'Webdesign',
       logo: null
     }
   ];
   ```

   Ensuite, affichons cette liste dans le Dashboard. Nous utiliserons la directive `ngFor` pour parcourir le tableau `skills`

   ```html
   <ul>
     <li *ngFor="let skill of skills">
       {{ skill.name }}
     </li>
   </ul>
   ```

   Nous avons maintenant une liste de prestations sur notre page. Pour distinguer plus rapidement les lignes entre-elles, ajoutons le logo. L'écriture avec crochets `[]` nous permet d'injecter directement une variable sans passer par l'interpretation des accolades. 

   ```html
   <img [src]="skill.logo" [alt]="skill.name">
   ```

   Pour que le compilateur sache où chercher votre image, nous allons la mettre dans le dossier `src/assets` du projet. De manière générale, tout ce qui n'est pas du code ira dans ce dossier.

   Rectifions le chemin source des images.

   ```typescript
   skills = [
     {
       name: 'Développement WordPress',
      	logo: 'assets/wordpress.png'
     },
     ...
   ]
   ```

   Certaines prestations n'ont pas de logo et créé ainsi de l'HTML inutile. Nous allons conditionner l'affichage de la balise image en fonction de la présence de chemin source ou non à l'aide de la directive `ngIf`.

   ```html
   <img *ngIf="skill.logo" [src]="skill.logo" [alt]="skill.name">
   ```

5. Afficher la liste des clients

   Comme toutes agences Web, il nous faudra mettre en avant les clients pour lesquels nous avons travaillés. Affichons une liste récapitulative des clients déjà créés. De la même manière que les prestations, commencez par créer une variable qui stocke sous forme de tableau tous les clients et affichez-la sur le dashboard.