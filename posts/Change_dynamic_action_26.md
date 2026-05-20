# APEX 26.1 : Au-delà d'Apexlang, vos Actions Dynamiques changent aussi !

Depuis la sortie d'APEX 26.1, la communauté n'a un mot à la bouche : Apexlang. Si cette nouveauté liée à l'IA attire les projecteurs, d'autres changements plus discrets — mais importants pour votre quotidien de développeur — se sont glissés dans la version. Les Actions Dynamiques (DA) ont notamment reçu un sérieux lifting.

Que ce soit pour suivre les évolutions des navigateurs ou pour offrir davantage de puissance aux composants modernes, voici ce qui change réellement pour vos DA.

## 1. Ergonomie — Du changement dans le Page Designer

L'un des réflexes des développeurs APEX évolue : un clic droit sur un bouton dans l'arbre du Page Designer remplace l'option `Create Dynamic Action` par `Create Trigger Action`.

Pour créer une action dynamique classique désormais, définissez la propriété `Type` sur `Defined by Dynamic Action`, puis créez l'action comme d'habitude.

## 2. Composants à base de modèles — Modèles éditables côté client

Les régions de type Cards (Cartes) et Template Components (Composants de modèle) gagnent un modèle éditable côté client.

Avantage : vous pouvez retourner des valeurs dans ces colonnes en utilisant les actions dynamiques standards `Set Value` et `Execute Server-side Code`. Pour les Template Components, cela fonctionne dès qu'une ou plusieurs colonnes sont marquées comme `Available on Client`.

Piège à éviter : si vous manipuliez le DOM directement en JavaScript (classes, injection HTML) après le rendu de ces régions, ces modifications seront perdues — l'interface se rafraîchit automatiquement lorsque le modèle change.

## 3. Listes de valeurs (LOV) — Nouvelle syntaxe de retour

Dans les expressions HTML des Template Components et des colonnes LOV des Interactive Reports, vous pouvez désormais cibler la valeur de retour (RETURN) plutôt que la valeur d'affichage (DISPLAY).

La syntaxe à utiliser : `&\"COLUMN_ALIAS%RETURN\"`.

**Attention :** le nom de la colonne doit être entouré de guillemets doubles (`\"`). Si vous les oubliez, APEX renverra `NULL`.

## 4. Alerte technique — Adieu l'événement `unload`, place à `pagehide`

C'est le changement le plus critique pour la stabilité des anciennes applications : les navigateurs modernes abandonnent progressivement l'événement JavaScript `unload`.

Pour s'adapter, l'action dynamique Page Unload d'APEX écoute maintenant l'événement `pagehide`. Le comportement reste similaire dans la plupart des cas, mais sur les navigateurs mobiles ces événements de fermeture ne sont plus déclenchés de manière fiable à 100%.

### Comment vérifier vos applications

Passez en revue vos applications pour repérer les actions basées sur la fermeture de page. Exécutez la requête suivante dans votre espace de travail pour identifier les pages concernées :

```sql
select application_id,
       application_name,
       page_id,
       page_name,
       dynamic_action_name,
       when_event_name,
       when_event_internal_name,
       number_of_actions
from apex_application_page_da
where when_event_internal_name = 'unload'
order by application_id, page_id, dynamic_action_name;
```

En résumé : APEX 26.1 continue de moderniser l'architecture de ses composants tout en s'alignant sur les nouvelles normes du web. C'est bien beau de générer des apps avec l'IA, mais un petit audit de vos actions "Unload" s'impose avant la mise à jour pour s'assurer que l'existant continue de tourner rond !