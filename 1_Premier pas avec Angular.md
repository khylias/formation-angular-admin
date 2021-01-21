## Module JS avancé : Angular

# Premier pas avec Angular

*Pré-requis: avoir sur votre machine Node (>=10.9.x)*

PROBLEMATIQUE : Créer facilement une application Angular

1. L'Interface en Ligne de Commande, la CLI

   La CLI est un outil qui vous aidera tout au long de votre projet pour automatiser différentes tâches. Commençons par l'installer sur votre machine à l'aide de NPM.

   ```bash
   npm install -g @angular/cli
   ```

   Vérifier ensuite que l'installation de la commande `ng` s'est bien déroulé.

   ```bash
   ng --version
   ```

   La liste des paquets Nodes d'Angular doivent être listés sur votre terminal à la suite de cette commande. 

2. Créer son projet

   Nous allons nous servir de la commande `ng` précédemment ajoutée à votre machine pour nous aider à créer la base du projet. Ouvrez un terminal dans un dossier de travail et créons le projet.

   ```bash
   ng new admin-miaw
   ```

   Le terminal vous demandera alors si vous souhaitez ajouter le module Routing, tapez `N`. Ensuite, il vous demandera dans quel langage de style vous voulez créer votre projet. Choississez selon vos préférences. (Je vous conseille SCSS parmi les pré-processeurs)

   Une fois la création de votre projet, rendez-vous dans le dossier en ligne de commande et lancez votre application Angular avec la CLI.

   ```bash
   ng serve
   ```

   Allez à l'url indiquée : http://localhost:4200

   

   Félicitations, vous venez de créer votre base de projet Angular ! 🎉

