Draft React-Redux
=================

React
------

React est une bibliothèque javascript permettant
de **construire des interfaces graphiques composables**. 

La modélisation des interfaces graphiques d’une application au cours du temps est un sujet
complexe. Il est, en effet, difficile de suivre l’état d’une application, après une série
d’interactions utilisateur et/ou externes.
Dans un SPA, nous ne pouvons plus compter sur des "rechargement totals" d'une page web pour garder une interface synchronisée et cohérente.

La solution de React est de décrire “à quoi” l’application doit
ressembler à n’importe quel instant donné → construction de
l’UI de manière **déclarative**. Ce fonctionnement nous donne
l’impression que React redessine entièrement l’interface à chaque update
(uniquement une impression → cf *DOM virtuel*) Cela rend le
développement d’application considérablement plus simple et nous permet
de garder très facilement l’interface à jour avec les données.
L’élaboration de ces interfaces se fait à base de **composants React**.

L’API des composants est très simple. Un composant *peut* posséder :
-   un **state**
-   des propriétés: les données d’entrées du composant →
    **props**
-   Une méthode **render** chargée du rendu du composant, appelée
    lorsque son *state* ou une de ses *props* changent.
-   Des méthodes liées au cycle de vie du composant componentDidMount,
    componentWillReceiveProps etc..)


#### Ecriture avec les classes ES6
Ci dessous un composant React ayant pour seule vocation à afficher la propriété *user*.
Si *user* change, React redessine la partie du composant ayant changé.
```javascript
class Bonjour extends React.Component {
  render() {
    return <div>Bonjour {this.props.user}</div>
  }
}
```

#### Ecriture sous forme de fonction 
Ce composant peut aussi être écrit sous la forme d'une
fonction appelée **Stateless functional component**. 
```javascript
const Bonjour = (props) => (
  <div>Bonjour {this.props.user}</div>
)
```

Ce type de composant **ne possède pas de state, pas d’instance ni 
de méthodes liées au cycle de vie** d’un composant React. Il ne s’agit que de
simples fonctions retournant un résultat ne dépendant que
des données d’entrées (les *props*)
Cette écriture étant plus consise, elle est à privilégier dans la mesure du possible.



#### Dessiner un composant dans un noeud du DOM
```javascript
ReactDOM.render(
  <Bonjour user="Vincent" />,
  document.getElementById('root')
)
```
Pour dessiner le composant dans le DOM il suffit d’appeler
la méthode `ReactDOM.render` avec ledit Composant et le noeud du DOM où
l’on souhaite le dessiner

#### JSX

Le code XML-like que retourne la méthode *render* s’appelle du **JSX**
et est un sucre syntaxique permettant de créer les noeuds React.
L’utilisation du JSX n’est pas obligatoire. Voici la correspondance du
code JSX :
```javascript
<Bonjour user="Vincent" />
```

avec le code javascript équivalent :
```javascript
React.createElement('Bonjour', {
  user: 'Vincent'
});
```


#### DOM Virtuel
React implémente un **DOM virtuel** qui est une représentation interne
en javascript du DOM. Voici un schéma illustrant le processus :


<p align="center" ><img src="https://www.dropbox.com/s/ovawut1f5t6yfae/react_batch.svg?dl=1" width="600"></p>
<h6 align="center" >React et son DOM virtuel</h6>


Lorsque les <span style="color: #D32F2F">données changent</span> la méthode *render* d’un composant renvoie 
un object correspondant à la représentation interne du DOM virtuel.
React compare ensuite ce nouveau DOM virtuel avec le précédent
(algorithme de diff interne), et met à jour le *vrai DOM* en appliquant un série d’opérations
optimisées. Ce DOM virtuel permet donc d’optimiser les accès au “vrai DOM”, les modifications sont appliquées
en une fois.

Architecture Flux : Redux
-------------------------

### Synoptique technique

Voici un schéma illustrant l'architecture d'une application redux.
Les différentes composantes de ce schéma sont expliquées dans la suite de ce document.

<p align="center"><img src="https://www.dropbox.com/s/eoqyq874z957s4i/synoptique.svg?dl=1" width="500"></p>
<h6 align="center" >Synoptique technique React-Redux</h6>

Ps: Redux est une bibliothèque dogmatique mettant en scène plusieurs concepts et patterns (immutabilités, flux unidirectionel etc..)
et ses principes sous-jacents peuvent parfaitement s'appliquer à d'autres architectures.


### Principe

*React* fournit seulement un moyen de dessiner de manière efficace des
composants en fonction de données d’entrées.

**Flux** est un style d’architecture qui garanti un flux de données
unidirectionnel (*one way databinding*) **Redux** est l’implémentation
la plus populaire.

*Redux*, met en scène 3 principes :

-   **une seule source de vérité** : le **state** de l’application est
    maintenu dans une structure de données à l’intérieur d’un seul
    **store**

-   **le state est immutable** : La seule manière de modifier le
    *state* est via l’émission d’une **action**, un objet
    **décrivant** la modification à apporter. 
    Toutes les modifications sont centralisées et se produisent une
    à une, évitant ainsi les problèmes de concurrence

-   **les modifications sont effectuées à l’aide de fonction pures**
    appelées **reducers**


<p align="center"><img src="https://www.dropbox.com/s/yst3zq1o0uxtas7/flux.svg?dl=1" width="540"></p>
<h6 align="center">Architecture Redux</h6>

Le schéma illustre le flux unidirectionnel de donnée dans cette
architecture: des actions sont “dispatchées” et traitées par le
*reducer*, qui se charge de mettre à jour le *store*. Toutes les vues
(ici les composants react) abonnées au store se mettent à jour en
conséquence. Ces vues peuvent également dispatcher des
actions et ainsi de suite.

### Actions et Actions creator

Les **actions** sont des paquets de données envoyés au *store*. Elles
sont la seule source d’information du store. Une action est envoyée au
store grâce à la fonction `store.dispatch`.

Voici un exemple d’action qui représente le
changement de nom d’une personne :

```javascript
{
  type: 'CHANGE_NAME',
  payload: 'Vince'
}
```

Si cette action se révèle être utilisée souvent, nous pouvons écrire une fonction qui se chargera de la créer pour nous.
```javascript
function changeName(name) {
  return {
    type: 'CHANGE_NAME',
    payload: 'Vince'
  }
}
```

On appelle ces fonctions des **action creator**.  Elles rendent les actions réutilisables et facilement testables. 
Les actions peuvent être “dispatchées” avec : `dispatch(changeName(’Vincent’))`

### Reducer

Les actions décrivent le fait que quelque chose s’est passé mais ne
spécifient pas la manière dont le store doit être modifié. C’est le rôle
du **reducer**. Le **reducer** est une **fonction pure** qui prend en
paramètre le *state*, une action, et retourne le nouveau *state*.

```
(previousState, action) =&gt; nextState
```

Le reducer est une fonction pure, par conséquent il ne doit **jamais**:

-   modifier directement ses arguments

-   effectuer des opérations ayant des effets de bord tel que des appels
    à une api

-   appeler des fonctions impures telles que Date.now() etc..

Il est uniquement chargé de calculer le *nextState*.

##### ✘ Exemple: un reducer incorrect : Mutation du state INTERDITE
Le state est muté.
La propriété du state étant modifiée directment (l.4),
les composants abonnées à cette partie du state ne se mettrons pas à jour et ignorerons cette modification.

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

##### ✔ Exemple Un Reducer correct 
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

Ps : On utilise ici l’opérateur **object spread** (**...**),
une syntaxe d’ECMAScript 2016, qui permet de copier les propriétés d’un
object dans un nouvel object d’une manière plus succincte.
Nous pouvons également utiliser des bibliothèques qui garantissent l'immutalibilité telles que *[immutable.js](https://github.com/facebook/immutable-js/)* développée par Facebook.

<h6>TODO: combineReducer pour réduire le boilerplate</h6>

### Store

Le **store** est un objet qui:

-   maintient le **state** de l’application

-   permet l’accès à ce *state* via `getState()`

-   permet de mettre à jour le *state* via `dispatch(action)`

-   permet d’abonner des composants via `subscribe(listener)` (composants notifiés lorsque le state subit une modification)

### Async Actions
Afin d'orchestrer des flux asynchrones (par exemple, les appels réseaux) nous pouvons utiliser le middleware *Redux-thunk*. 
Ce middleware permet de traiter les **actions** étant des **fonctions** (appelées *thunk action*).
Une action *thunk* ne doit pas forcément être pure et peut avoir des effets
de bords. Les fonctions *dispatch* et *getState* du store lui sont passé en argument, ce qui lui donne la possibilité de *dispatcher* d'autres *actions* et d'accéder au *state*.

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
<h6 align="center">Exemple d’un thunk action creator qui retourne une function</h6>

L’exemple ci-dessus met en évidence une *action creator* qui retourne
une fonction. Des actions marquant le début, le succès ou une erreur de
l’appel (l.5) à l’API sont “dispatchées” (l.3, l.7, l.9) permettant de
mettre à jour le *store* en fonction de l'avancement de la requête.

Remarques relativement au code ci-dessus : une syntaxe
particulière avec les mots clés **async/await**. Cette syntaxe est une
proposition (stage 3) pour ECMAScript 2016. En résumé, `await`
permet d’attendre la résolution d’une promesse et ne peux être utilisé
que dans une fonction préfixée par `async`.
Il permet d’écrire le code asynchrone de javascript à la manière d’un code synchrone et ainsi
éviter les *callback hell* et donc rendre le code plus lisible.
Il permet également d’avoir une gestion d’erreur beaucoup plus
agréable à l’aide des `try/catch`.

2 types de composants React
---------------------------------------------------

D'un point de vue architectural, il est bon de distinguer deux types de composants React: 
les **container component** (ou *smart component*) et
**presentational component** (ou *dumb component*)
Si l'on se rapportait à une architecture MVC plus tradionnelle, 
le premier correspondrait au **C**ontrolleur et le deuxième à la **V**ue.
**On sépare donc les composants responsables de la logique métier de ceux reponsables de la vue**
(on ne melange pas les chèvres et les brebis)

### *Container* composants 

Les *container components* sont responsables de la manière dont **“les choses” fonctionnent**. 
Ils peuvent **communiquer directement avec le _store_** dans l’architecture Redux. 
Ils sont entièrement **spécifiques à l’application**.
Ils peuvent contenir aussi bien des *container components* que des *presentational components*.
Ils **transmettent** les **données du store** et les **comportements** (**callback**) **aux dumb components** **via leurs _props_**. 
Ils **déclenchent les actions Flux**(Redux).

### *Presentational* composants 

Les *presentational components* sont responsables de la manière dont **“les choses”
apparaissent sur l’interface.**
Ils sont complètement **indépendants** du reste de l’application (*store*, *actions*) 
et sont donc réutilisables, que ce soit à l’intérieur ou à l’extérieur de l’application.
Ils sont parfois appelés “dumb” car ils ne **connaissent que leur _props_.**
Ils **recoivent les données et les callback exclusivement via leurs props**.
Ils peuvent être la plupart du temps écrits sous la forme de fonction (cf figure 6)

### Exemple avec React (sans redux)
Considérons un composant qui affiche une liste de pistes (tracks) provenant d'une api.

Le code ci-dessous est **mauvais** 👿, en effet un même composant **ne devrait pas**:
- aller chercher les données de l'api et potentiellement les transformer 
- afficher et mettre en forme les données 

Il n'y a aucune séparation entre la vue et les traitements et ce type de code peut très vite s'avérer désordre.

##### ✘ Un "mauvais" composant :
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

Nous pouvons le séparer en 2 composants, le premier étant un **container component** et le deuxieme un **presentational component**.

##### ✔ Container component :

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


##### ✔ Presentational component : 
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

Selectors
-----------

### 
Afin de comprendre l'utilité des selecteurs, prenons un exemple.
Considérons une liste de personnes, une recherche (par nom) et des filtres (sexe, age, etc...) sur ces personnes.

En suivant les principes *Redux*, le *store* contient les données et les critères de recherche.
A partir de ces éléments nous pouvons calculer la liste filtrée à afficher. 

Une bonne pratique, concernant le *state*, est de contenir seulement des *donnée minimisée*,
c'est à dire des données ne pouvant pas être obtenues à partir d'autres données.
Les états dérivés (calculés) ne doivent pas être présents dans le *state*.


<p align="center"><img src="https://www.dropbox.com/s/aj8zj878ivnf7n6/selectors0.svg?dl=1" width="260"></p>

On a donc la méthode `render` chargée de filtrer la liste et de l’afficher. 
Ainsi, si  un critère de recherche ou si les données changent,
le composant execute la méthode `render`, filtre les données et les affiche.
Il en résulte une UI toujours synchronisée avec les données du state. 

L'inconvénient est que le filtrage se fait à chaque mise à jour du composant. 
Cela n'est pas très génant dans ce cas ci.
Cependant, des listes de données potentiellement grandes et des calculs complexes dégraderaient fortement les performances de l'application.


C'est là qu'entrent en jeu les **selectors**:

<p align="center"><img src="https://www.dropbox.com/s/7gxp1rrntx2lhmj/selectors.svg?dl=1" width="300"></p>

Les *selectors* **calculent des données dérivées**. Ils permettent au *state* de ne stocker que les **données minimisée**.
Ils sont **efficaces** et ne sont **pas recalculés** si les arguments restent les mêmes.
Enfin ils sont *composables*, c’est à dire qu’ils peuvent être utilisés en
tant qu’entrées à d’autres *selectors*.
Ainsi toute la complexité est *déplacée* à l'exterieur et prise en charge par les **selectors**,

Les *selectors* jouent le rôle d'*api*, permettant un accès au state. 
Les composants React ne connaisse que cette interface.
Une conséquence directe est le *découplage* de ces composants vis-à-vis de la *forme* du state. 
Un autre bénéfice est la simplification du code des composants React.


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
