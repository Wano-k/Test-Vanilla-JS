# Install the project
npm install

# Run the project
npm start

# Notes de dévelopement

### Bug #1 (+ #4)
J'ai commencé par jeter un oeil au `index.html` pour repérer l'élément affichant le "loading" infini et commencer à voir l'organisation globale. J'ai ensuite inspecté la page pour voir s'il y avait des erreurs, et il y en avait bien une: `Uncaught (in promise) TypeError: self.tasks is undefined`. Je vais donc voir comment `tasks` est initialisé. Je vois que c'est fait par un setter dans `main.js`: `tasksTable.setTasks(tasks)`. Avant de voir si la valeur de tasks est `undefined`, je regarde si le setter est le problème. Et effectivement, je vois `this.tasks = tasks`, alors qu'il faudrait utiliser `self.tasks = tasks` pour assigner la valeur au bon scope (ici `this` correspond à un autre scope isolé dans la fonction).

Mais je remaque que des fois ça ne charge pas sans raison. J'ai vu que dans `getTranslations()` de `data.service.js` il y a un random qui peut faire rejeter la promise, mais je n'ai pas le droit d'y toucher et c'est visiblement pour simuler des erreurs de connexion au serveur. Pour contrer cela, il faudrait donc retenter de récupérer les traductions jusqu'à ce que ça soit un succès, tout en continuant d'afficher message "Loading...". Pour cela, j'ai simplement catché la promise avec `.catch(updateTable)` pour réexecuter la fonction en cas d'echec (après avoir lu l'énoncé du bug 4, je me rend compte que j'ai fixé le problème de cette manière).

### Bug #2

Pour la liste company qui ne s'affiche pas correctement, j'ai vu que `updateCompanies` était asynchrone et qu'on pouvait avoir besoin de ces informations dans `updateTable`. J'ai donc fait en sorte d'attendre que les companies soient chargées en faisant: `updateCompanies().then(updateTable)`. J'ai également, pour que le boutton refresh soit plus cohérent, ajouté une méthode load qui charge les companies et la table plutôt que de ne charger que la table.

### Bug bonus

Je vois dans la console une erreur de parsing. Je regarde d'abord dans `main.js` où se fait visiblement le parsing du XML. Je regarde très vite les fonctions globales. Je vois que Robert Jonhson dans la première page d'instruction apparait dans le pdf, mais pas chez moi. L'erreur de parsing doit se trouver là, alors j'isole l'xml:

````
<?xml version=\"1.0\" encoding=\"utf-8\"?><Item><TxtType type=\"System.String\">telework</TxtType><TxtCivilName type=\"System.String\">Robert Johnson</TxtCivilName><NumCompanyId type=\"System.Int32\">4</NumCompanyId><NumDaysRequested type=\"System.Int32\">1</NumDaysRequested><TxtDateRange type=\"System.String\">Friday 19 March 2021</TxtDateRange><TxtStatus type=\"System.String\">Hierarchy Validation</TxtStatus><TxtComment type=\"System.String\">To add in calendar Information & Digital Services</TxtComment></Item>
````

Dans le deboggueur, je vois bien que c'est cette partie qui renvoie un parseerror. Et je vois le symbole `&` qui avait déjà été problèmatique dans d'autres anciens projets pour le parsing. J'enlève manuellement le symbole pour voir si ça résout le problème, et oui ! Je ne peux pas éditer ce fichier directement, je vais donc modifier un peu la méthode de parsing dans le `main.js`: `xml = xml.replace(/&/g, "&amp;amp;")` pour que le parser fonctionne.

### Bug 3
Pour ce problème, j'ai regardé le composant `task-table`. Il faut modifier la méthode `aggregateFn` dans `self.columns` de number of days. La méthode renvoit le nombre à additioner pour chaque ligne pour obtenir la somme finale. Il y a un cas où `item['NumDaysRequested']` vaut `NaN`, ce qui pertube le calcul total. On peut juste rajouter `|| 0` pour avoir `0` par défaut lorsque le nombre vaut `NaN` ou `undefined`.

### Bug 5
Je vais directement voir de nouveau le composant `task-table`, et je repère très vite `getTableElement()` avec l'event `click`. Le problème est encore un problème de scope. Il faut que la fonction anonyme ait le `task` correspondant dans son scope. Pour cela, j'ai étendu la portée de `task` en utilisant `let` au lieu de `var`.
