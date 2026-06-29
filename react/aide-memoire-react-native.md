# Aide-mémoire React Native (pour un dev ReactJS)

## 1. Les composants de base (équivalents du HTML)

| Web (React) | React Native | Rôle |
|---|---|---|
| `<div>` | `<View>` | Conteneur générique, flexbox par défaut |
| `<span>`, `<p>`, `<h1>` | `<Text>` | **Tout texte doit être dans un `<Text>`** — impossible de mettre du texte brut dans un `<View>` |
| `<img>` | `<Image>` | `source={{ uri: '...' }}` ou `source={require('./local.png')}` |
| `<button>` | `<Pressable>` / `<TouchableOpacity>` | Pas de `<button>` natif ; `Pressable` est le plus moderne/recommandé |
| `<input>` | `<TextInput>` | Géré en `value`/`onChangeText` (pas `onChange` avec un event) |
| liste scrollable | `<FlatList>` / `<ScrollView>` | `FlatList` = virtualisée (perf, pour listes longues type tes interventions) ; `ScrollView` = tout rendu d'un coup |
| CSS externe | `StyleSheet.create({...})` | Objet JS, pas de fichier `.css`, pas de cascade |

**Règle d'or à retenir** : si ça plante avec "Text strings must be rendered within a `<Text>` component", c'est que du texte traîne directement dans un `<View>`.

## 2. Le style : ce qui change vraiment

```javascript
import { StyleSheet, View, Text } from 'react-native';

const styles = StyleSheet.create({
  conteneur: {
    flex: 1,              // flexbox actif PAR DÉFAUT (pas besoin de display:flex)
    flexDirection: 'column', // colonne par défaut (inverse du web où c'est 'row' par défaut)
    padding: 16,
    backgroundColor: '#fff',
  },
  titre: {
    fontSize: 18,          // jamais d'unité (pas de 'px', juste un nombre = density-independent pixels)
    fontWeight: 'bold',
  },
});

function MonEcran() {
  return (
    <View style={styles.conteneur}>
      <Text style={styles.titre}>Salon</Text>
    </View>
  );
}
```

Points à retenir :
- Pas de sélecteurs CSS, pas d'héritage de style entre composants (sauf `Text` imbriqués qui héritent entre eux)
- `flexDirection` par défaut = `'column'` (web : `'row'`) — piège classique au début
- Combiner plusieurs styles : `style={[styles.base, styles.variante]}` (tableau, pas de classes multiples comme en CSS)
- Pas d'unité sur les nombres (`fontSize: 18`, pas `'18px'`)

## 3. Navigation (React Navigation)

Pour ta hiérarchie Salon → Jours → Interventions, c'est un **Stack Navigator** classique :

```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Salons">
        <Stack.Screen name="Salons" component={EcranSalons} />
        <Stack.Screen name="Jours" component={EcranJours} />
        <Stack.Screen name="Interventions" component={EcranInterventions} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

Naviguer et passer des paramètres (ex: l'ID du salon choisi) :

```javascript
// Dans EcranSalons, au clic sur un salon :
navigation.navigate('Jours', { salonId: salon.id });

// Dans EcranJours, pour récupérer le paramètre :
function EcranJours({ route }) {
  const { salonId } = route.params;
  // ...
}
```

C'est conceptuellement proche de React Router (`useNavigate`, `useParams`), mais l'API est différente — pas d'URL, pas de `<Route path="...">`.

## 4. Hooks : (presque) tout pareil qu'en React

`useState`, `useEffect`, `useCallback`, `useMemo`, contexte (`useContext`) — **identiques** à ce que tu connais en ReactJS. C'est un vrai confort de la migration : ta logique d'état et tes hooks custom (par exemple pour la synchronisation API) se portent quasiment sans changement.

Ce qui change : les hooks spécifiques à la navigation (`useNavigation()`, `useRoute()`) et certains hooks liés au cycle de vie de l'écran (`useFocusEffect` de React Navigation, utile quand tu reviens sur un écran après être passé sur un autre — par exemple recharger la liste des interventions du jour quand on revient de l'écran détail).

## 5. Appels API (ta synchronisation)

`fetch` fonctionne nativement, exactement comme sur le web — ta logique de synchronisation avec l'API Symfony est probablement la partie qui se portera le plus facilement, presque sans changement :

```javascript
async function recupererInterventions(salonId, jour) {
  const reponse = await fetch(`https://ton-api.fr/interventions?salon=${salonId}&jour=${jour}`);
  return await reponse.json();
}
```

## 6. Stockage local (remplace localStorage de Cordova)

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';

await AsyncStorage.setItem('derniere_sync', new Date().toISOString());
const valeur = await AsyncStorage.getItem('derniere_sync');
```

Asynchrone par nature (toujours des Promises), contrairement à `localStorage` qui est synchrone.

## 7. Spécifique à ton projet : la signature

`react-native-signature-canvas` (composant qu'on va utiliser à la place de SurveyJS) fonctionne par callback retournant l'image en base64 :

```javascript
import SignatureScreen from 'react-native-signature-canvas';

function EcranSignature({ onSignatureCapturee }) {
  const handleOK = (signatureBase64) => {
    // signatureBase64 est une chaîne data:image/png;base64,...
    onSignatureCapturee(signatureBase64);
  };

  return <SignatureScreen onOK={handleOK} />;
}
```

À envoyer ensuite vers ton API Symfony comme tu envoies probablement déjà une image en base64 ou un fichier.

## 8. Outils pour voir le résultat en live

- **Expo Go** (app sur ton téléphone) : scanne un QR code généré par `npx expo start`, et l'app se recharge en live sur ton téléphone à chaque sauvegarde — le plus rapide pour itérer sans repasser par un build natif complet
- Recharge à chaud activée par défaut, comme le Hot Reload web

## 9. Pièges fréquents à connaître à l'avance

- **`<ScrollView>` avec une `<FlatList>` à l'intérieur** : ne pas imbriquer, ça casse le scroll. Une seule liste virtualisée par écran scrollable.
- **`Platform.OS`** pour gérer les différences iOS/Android quand nécessaire : `Platform.OS === 'ios' ? ... : ...`
- **Les `key` dans les listes** : toujours nécessaires comme en React, mais `FlatList` utilise `keyExtractor` plutôt qu'un `key` sur chaque élément JSX.
- **Pas de `window`/`document`** : tout ce qui était basé sur le DOM (querySelector, etc.) côté Preact n'a pas d'équivalent — normal, il n'y a pas de DOM.
