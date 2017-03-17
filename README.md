Draft React-Redux
=================

React
------

React est une bibliothèque javascript permettant
de **construire des interfaces graphiques composables**. 

La modélisation des interfaces graphiques d'une application au cours du temps est un sujet
complexe. Il est, en effet, difficile de suivre l'état d'une application, après une série
d'interactions utilisateur et/ou externes.
Dans un SPA, nous ne pouvons plus compter sur des rechargements entiers d'une page web pour garder une interface synchronisée et cohérente.

La solution de React est de décrire “à quoi” l'application doit
ressembler à n'importe quel instant donné → construction de
l'_UI_ de manière **déclarative**. Ce fonctionnement nous donne
l'impression que React redessine entièrement l'interface à chaque _update_
(uniquement une impression → cf *DOM virtuel*) Cela rend la conception et le
développement d'application considérablement plus simple et nous permet
de garder très facilement l'interface à jour avec un modèle de données.
L'élaboration de ces interfaces se fait à base de **composants React**.

On pourrait assimiler un **composant React** à une **fonction**.
C'est d'ailleurs une des 2 manières possibles d'écrire une composant React.

L'API des composants est très simple. Un composant *peut* posséder :
-   un **state**
-   des propriétés : les données d'entrées du composant →
    **props**
-   Une méthode **render** chargée du rendu du composant, appelée
    lorsque son *state* ou une de ses *props* changent.
-   Des méthodes liées au **cycle de vie** du composant `componentDidMount`,
    `componentWillReceiveProps` etc...)


#### Écriture avec les classes ES6
Ci-dessous un composant React ayant pour seule vocation d'afficher la propriété *user*.
Si *user* change, React redessine la partie du composant ayant changé.
```javascript
class Bonjour extends React.Component {
  render() {
    return <div>Bonjour {this.props.user}</div>
  }
}
```

#### Écriture sous forme de fonction 
Ce composant peut aussi être écrit sous la forme d'une
fonction appelée **Stateless functional component**. 
```javascript
const Bonjour = (props) => (
  <div>Bonjour {this.props.user}</div>
)
```

Ce type de composant **ne possède pas de _state_, pas d'instance ni 
de méthodes liées au cycle de vie** d'un composant React. Il ne s'agit que d'une
simple fonction retournant un résultat en fonction de ses arguments (les *props*)
Cette écriture étant plus concise, elle est à privilégier dans la mesure du possible.



#### Dessiner un composant dans un nœud du DOM
```javascript
ReactDOM.render(
  <Bonjour user="Vincent" />,
  document.getElementById('root')
)
```
Pour dessiner le composant dans le DOM il suffit d'appeler
la méthode `ReactDOM.render` avec ledit Composant et le nœud du DOM où
l'on souhaite le dessiner

#### JSX

Le code "XML like" que retourne la méthode *render* s'appelle du **JSX**
et est un sucre syntaxique permettant de créer les nœuds React.
L'utilisation du JSX n'est pas obligatoire. Voici la correspondance du
code JSX :
```javascript
<Bonjour user="Vincent" />
```

Avec le code javascript équivalent :
```javascript
React.createElement('Bonjour', {
  user: 'Vincent'
});
```

#### Distinguer 2 types de composants React

D'un point de vue architectural, nous pouvons très vite distinguer deux types de composants.
Redux (cf : suite de l'article) parle de **container component** (ou *smart component*) et de
**presentational component** (ou *dumb component*)
Si l'on se rapportait à une architecture _MVC_ plus traditionnelle, 
le premier correspondrait au **C**ontrolleur et le deuxième à la **V**ue.
**On sépare donc les composants responsables de la logique métier/orchestration des actions, de ceux reponsables de la vue**

### Exemple
Considérons un composant qui affiche une liste de pistes (_tracks_) provenant d'une api.

Le code ci-dessous est **moyen** 👿, en effet un même composant **ne devrait pas** être responsable à la fois :
- d'aller chercher les données de l'api et potentiellement les transformer  
- d'afficher et mettre en forme ces données 

Ce manque de séparation entre la vue et la logique métier peut très vite rendre le code difficile à maintenir lorsque ce dernier grossit.

#### ✘ Un "mauvais" composant :
```javascript
class TrackList extends React.Component {
  state = { tracks: [] }

  componentDidMount() {
    axios.get('/tracks')
      .then(response => response.data)
      .then(tracks => this.setState({ tracks }))
      .catch(handleError);
  }

  render() {
    return (
      <ul>
        {this.state.tracks.map(track => (
          <li>{track}</li>
        ))}
      </ul>
    )
  }
}
```

Nous pouvons le séparer en 2 composants, le premier étant un composant "container" et le deuxième un composant visuel.

#### ✔ Composant _Container_ :

```javascript
// LOgic is here!! 
// we have completely separated our logic and our view
class TrackListContainer extends React.Component {
  state = { tracks: [] }

  componentDidMount() {
    axios.get('/tracks')
      .then(response => response.data)
      .then(tracks => this.setState({ tracks }))
      .catch(handleError);
  }

  render() {
    // This is our view 
    // and the `tracks` props is like our ViewModel 
    return <TrackList tracks={this.state.tracks} />
  }
}
```


#### ✔ Composant _Presentational_ : 
```javascript
// here is our view
const TrackList = ({ tracks }) => (
  <ul>
    {this.state.tracks.map(track => (
      <li>{track}</li>
    ))}
  </ul>
)
```

#### DOM Virtuel
Lorsque nous écrivons un composant React, nous décrivons à quoi l'UI ressemble en fonction des *props*.
Même si React donne le sentiment au développeur de repeindre entièrement le DOM à chaque _update_,
il implémente, en réalité, un **DOM virtuel** qui est une représentation interne
en javascript du DOM. 
Voici un schéma illustrant le processus :


<p align="center" ><img src="https://www.dropbox.com/s/ovawut1f5t6yfae/react_batch.svg?dl=1" width="600"></p>
<h6 align="center" >React et son DOM virtuel</h6>


Lorsque le <span style="color: #D32F2F">modèle de données change</span> la méthode *render* du composant renvoie 
un objet correspondant à la représentation interne du DOM virtuel.
React compare ensuite ce nouveau DOM virtuel avec le précédent
(algorithme de diff interne), et met à jour le *vrai DOM* en appliquant un série d'opérations
optimisées. Ce DOM virtuel permet donc d'optimiser les accès au “vrai DOM”, les modifications sont appliquées
en une fois.

Architecture Flux : Redux
-------------------------

### Synoptique technique

Voici un schéma illustrant l'architecture d'une application redux.
Les différentes composantes de ce schéma sont expliquées dans la suite de ce document.

<p align="center"><img src="https://www.dropbox.com/s/eoqyq874z957s4i/synoptique.svg?dl=1" width="500"></p>
<h6 align="center" >Synoptique technique React-Redux</h6>

> Note : Redux est une bibliothèque dogmatique mettant en scène plusieurs concepts et _patterns_ (immutabilités, flux unidirectionnel etc...)
> et ces principes sous-jacents peuvent parfaitement s'appliquer à d'autres architectures.


### Principe

*React* fournit seulement un moyen de dessiner de manière efficace des
composants en fonction de données d'entrées.

**Flux** est un _pattern_ permettant de gérer **l'état d'une application** qui garanti un flux de données
unidirectionnel (*one way databinding*) **Redux** est l'implémentation
la plus populaire.

*Redux*, met en scène 3 principes :

-   **une seule source de vérité** : le **state** de l'application est
    maintenu dans une structure de données à l'intérieur d'un seul
    **store**

-   **le state est immutable** : La seule manière de modifier le
    *state* est via l'émission d'une **action**, un objet
    **décrivant** la modification à apporter. 
    Toutes les modifications sont centralisées et se produisent une
    à une, évitant ainsi les problèmes de concurrence

-   **les modifications sont effectuées à l'aide de fonction pures**
    appelées **reducers**


<p align="center"><img src="https://www.dropbox.com/s/yst3zq1o0uxtas7/flux.svg?dl=1" width="540"></p>
<h6 align="center">Architecture Redux</h6>

Le schéma illustre le flux unidirectionnel des données dans cette
architecture : des **actions** sont **“dispatchées”** et traitées par le
**reducer**, qui se charge de mettre à jour le **store**. Toutes les vues
(ici les composants react) abonnées au store se mettent à jour en
conséquence. Ces vues peuvent également de "dispatcher" des
actions et ainsi de suite.

### Actions et _Actions creators_

Les **actions** sont des paquets de données envoyés au *store*. Elles
sont la seule source d'information du store. Une action est envoyée au
store grâce à la fonction `store.dispatch`.

Voici un exemple d'action qui représente le
changement de nom d'une personne :

```javascript
{
  type: 'CHANGE_NAME',
  payload: 'Vince'
}
```

Si cette action se révèle être utilisée souvent, nous pouvons écrire une fonction qui se chargera de la créer.
```javascript
function changeName(name) {
  return {
    type: 'CHANGE_NAME',
    payload: name
  }
}
```

On appelle ces fonctions des **action creator**.  Elles rendent les actions réutilisables et facilement testables. 
Les actions peuvent être “dispatchées” avec : `dispatch(changeName('Vincent'))`

### Reducer

Les actions décrivent le fait que quelque chose s'est passé mais ne
spécifient pas la manière dont le store doit être modifié. C'est le rôle
du **reducer**. Le **reducer** est une **fonction pure** qui prend en
paramètre le *state*, une action, et retourne le nouveau *state*.

```
(previousState, action) => nextState
```

Le _reducer_ est une fonction pure, par conséquent il ne doit **jamais**:

-   modifier directement ses arguments

-   effectuer des opérations ayant des effets de bord tel que des appels
    à une api

-   appeler des fonctions impures telles que `Date.now()` etc...

Il est uniquement chargé de calculer le *nextState*.

#### ✘ Exemple: un _reducer_ incorrect : Mutation du _state_ INTERDITE
Le _state_ est muté.
La propriété du _state_ étant modifiée directement (l.4),
les composants abonnés à cette partie du _state_ ne se mettrons pas à jour et ignorerons cette modification.

```javascript
function user(state = {}, action) {
  switch (action.type) {
    case 'CHANGE_NAME':
      state.name = action.name // INTERDIT !!
      return state;
    default:
      return state;
  }
}
```

#### ✔ Exemple Un _Reducer_ correct 
```javascript
function user(state = {}, action) {
  switch (action.type) {
    case 'CHANGE_NAME':
      return {
        ...state,
        name: action.name,
      }
    default:
      return state;
  }
}
```

> Note : On utilise ici l'opérateur **object spread** (**...**),
> une syntaxe d'_ECMAScript_ 2016, qui permet de copier les propriétés d'un
> objet dans un nouvel objet d'une manière plus succincte.
> Nous pouvons également utiliser des bibliothèques qui garantissent l'immutabilité telles que *[immutable.js](https://github.com/facebook/immutable-js/)* développée par Facebook.

<!-- <h6>TODO: combineReducer pour réduire le boilerplate</h6> -->

### Store

Le **store** est un objet qui :

-   maintient le **state** de l'application

-   permet l'accès à ce *state* via `getState()`

-   permet de mettre à jour le *state* via `dispatch(action)`

-   permet d'abonner des composants via `subscribe(listener)` (composants notifiés lorsque le _state_ subit une modification)

### Async Actions
Afin d'orchestrer des flux asynchrones (par exemple, les appels réseaux) nous pouvons utiliser le _middleware_ *Redux-thunk*. 
Ce _middleware_ permet de traiter les **actions** étant des **fonctions** (appelées *thunk action*).
Une action *thunk* ne doit pas forcément être pure et peut avoir des effets
de bords. Les fonctions *dispatch* et *getState* du store lui sont passées en argument, ce qui lui donne la possibilité de *dispatcher* d'autres *actions* et d'accéder au *state*.

#### Exemple d'un _thunk action creator_ qui retourne une fonction :
```javascript
function whatIsMyName() {
  return async (dispatch, getState) => {
    dispatch(fetchNameRequest());
    try {
      const res = await fetch('http://vincent.cordobes/name');
      const name = await res.json();
      dispatch(fetchNameSuccess(name));
    } catch (err) {
      dispatch(fetchNameError(err));
    }
  }
}
```

L'exemple ci-dessus met en évidence une *action creator* qui retourne
une fonction. Des actions marquant le début, le succès ou une erreur de
l'appel (l.5) à l'API sont “dispatchées” (l.3, l.7, l.9) permettant de
mettre à jour le *store* en fonction de l'avancement de la requête.

> Remarques relativement au code ci-dessus : syntaxe
> avec les mots clés **async/await**. Cette syntaxe fait son apparition dans 
> ECMAScript 2017. En résumé, `await`
> permet d'attendre la résolution d'une promesse et ne peux être utilisé
> que dans une fonction préfixée par `async` (elle-même renverra à son tour une promesse)
> Il permet d'écrire le code asynchrone de javascript à la manière d'un code synchrone et ainsi
> éviter les *callback hell* et donc rendre le code plus lisible.
> Il permet également d'avoir une gestion d'erreur beaucoup plus
> agréable à l'aide des `try/catch`.

Composants "Container" et composants "visuels" 
-----------------------------------------------
La séparation "container"/"presentational" est d'autant plus vrai dans redux.
Ces deux termes proviennent, en l'occurrence, du créateur de redux.
![smart and dumb components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.xbauu6f3x)


### *Container* composants 
- Responsables de la manière dont **“les choses” fonctionnent**

- Sont souvent _stateful_ et servent de **sources de données**

- **“Dispatchent” les actions** flux

- **Transmettent** des **données et comportements** aux composants "presentational” **via leur _props_**

- Peuvent contenir des composants “présentation” et “container”

- **Ne contiennent pas** d'éléments du DOM ni de styles

- Peuvent être **générés par connect()**

### *Presentational* composants 
- Responsables de la manière dont **“les choses” apparaissent sur l'interface**

- Peuvent contenir des composants “présentation” et “container”

- **possèdent** souvent des **éléments DOM** et du **style**

- **Indépendants** du reste de l'application

- Ne spécifient pas la manière dont les données sont chargés ou modifiées

- **Reçoivent** les **données et les callback** exclusivement via leurs **props**

- Possèdent uniquement un _state_ si celui-ci concerne l'UI (et non des data)

- Souvent écrits sous forme de fonctions

Selectors
-----------

Afin de comprendre l'utilité des sélecteurs, prenons un exemple.
Considérons une liste de personnes, une recherche (par nom) et des filtres (sexe, age, etc...) sur ces personnes.

En suivant les principes *Redux*, le *store* contient les données et les critères de recherche.
À partir de ces éléments nous pouvons calculer la liste filtrée à afficher. 

Une bonne pratique, concernant le *state*, est de contenir seulement des *donnée minimisée*,
c'est-à-dire des données ne pouvant pas être obtenues à partir d'autres données.
Les états dérivés (calculés) ne doivent pas être présents dans le *state*.


<p align="center"><img src="https://www.dropbox.com/s/aj8zj878ivnf7n6/selectors0.svg?dl=1" width="260"></p>

Le "bon" endroit pour filtrer et afficher cette liste est donc la méthode _render_.
Ainsi, si  un critère de recherche ou si les données changent,
le composant exécute la méthode `render`, filtre les données et les affiche.
Il en résulte une _UI_ toujours synchronisée avec le _state_. 

Cette technique présente néanmoins un inconvénient.
Supposons qu'une _props_ **autre** que les filtres et la liste de personnes, change :
le filtrage de la liste se fera donc, **inutilement**, à chaque _update_ du composant.

La complexité de ce filtrage étant du 0(n), cela n'est pas très gênant si la taille des données à filtre reste modérée. 

Cependant, des listes de données potentiellement grandes ou même un calcul plus  complexe dégraderaient fortement les performances de l'application.

C'est ici qu'entrent en jeu les **selectors**:

<p align="center"><img src="https://www.dropbox.com/s/7gxp1rrntx2lhmj/selectors.svg?dl=1" width="300"></p>

Les *selectors* **calculent des données dérivées**. Ils permettent au *state* de ne stocker que les **données minimisée**.
Ils sont **efficaces** et ne sont **pas recalculés** si les arguments restent les mêmes → ils sont **mémoisés**.
Enfin ils sont *composables*, c'est-à-dire qu'ils peuvent être utilisés en
entrées d'autres *selectors*.
Ainsi toute la complexité est *déplacée* à l'exterieur et prise en charge par les **selectors**,

Les *selectors* jouent le rôle d'*api*, permettant un accès au _state_. 
Les composants React ne connaisse que cette interface.
Une conséquence directe est le *découplage* de ces composants vis-à-vis de la *forme* du _state_. 
Un autre bénéfice est la simplification du code des composants React.

#### Exemple avec la bibliothèque [reselect](https://github.com/reactjs/reselect)

##### Définition des _selectors_
```javascript
const getUsers = state => state.users
const getSearchTerm = state => state.searchTerm

// Memoized selector
const getFilteredUsers = createSelector(
  getUsers,
  getSearchTerm,
  (users, searchTerm) => users.filter(
    user => user.indexOf(searchTerm) > -1
  )
);
```


##### Définition du composant React

```javascript
// Composant React - liste d'utilisateurs 
const UserList = ({ filteredUsers }) => (
  <ul>
    {filteredUsers.map(user => <li>{user}</li>)}
  </ul>
);
```

##### Création du container

```javascript
export default connect(state => (
  filteredUsers: getFilteredUsers(state)
))(UserList);
```
> Note : Nous avons seulement besoin de transmettre la liste filtrée au composant _UserList_.
> `connect` suffit à créer le composant *container*.

> Note 1 : Le _pattern_ *mapStateToProps* étant récurrent, un sucre syntaxique serait :

```javascript
export default connect(createStructuredSelector({
  filteredUsers: getFilteredUsers,
}))(UserList);
```

### Structurer le State
TODO: ..


----------------------------
### Références
- *React* - https ://facebook.github.io/react
- *Redux doc* - http://redux.js.org
- *Smart and dumb component* - https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.h6rxn85kl
- *React and flux in production best practices* - https://medium.com/@delveeng/react-and-flux-in-production-best-practices-c87766c57cb6#.elbdrmo4f
- *Javascript callback hell* - http://callbackhell.com
- *Async await proposal* - https://tc39.github.io/ecmascript-asyncawait
