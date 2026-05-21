---
title: "ORDS & APEX : Comment (enfin) vérifier proprement la présence d'un paramètre dans vos URL avec OWA_UTIL"
seoTitle: "OWA_UTIL : Gérer les paramètres d'URL sous ORDS et APEX"
seoDescription: "Comment intercepter et vérifier proprement la présence d'un paramètre HTTP dans vos URL ? Le guide pratique PL/SQL pour blindez vos API ORDS et APEX"
datePublished: 2026-05-21T21:59:06.675Z
cuid: cmpg19fap00aj2gmt22sbday4
slug: ords-apex-comment-enfin-v-rifier-proprement-la-pr-sence-d-un-param-tre-dans-vos-url-avec-owa-util
tags: plsql, ords, oracle-apex

---

# ORDS & APEX : Comment (enfin) vérifier proprement la présence d'un paramètre dans vos URL avec OWA_UTIL

Développer des API REST avec **ORDS (Oracle REST Data Services)** ou concevoir des applications dynamiques sous **Oracle APEX** implique de manipuler le protocole HTTP en permanence.

Un besoin revient souvent : **savoir si un paramètre spécifique est présent dans l’URL** pour adapter le comportement de votre code PL/SQL (un jeton `?token=`, un flag `?debug=true` ou un format `?output=pdf`).

Pour récupérer ce contexte web, le package historique **`OWA_UTIL`** reste un excellent couteau suisse. Mais attention : les chaînes issues du web cachent un piège invisible.

Voyons comment traiter cela de manière chirurgicale et robuste.
---

### 💡 Le saviez-vous ?

Un simple `INSTR(query_string, 'votre_parametre')` vous dirige assurément vers un bug invisible en production.

**Pourquoi ?** Imaginez que vous cherchiez à savoir si le paramètre `id` est présent dans votre URL. Si un utilisateur appelle votre endpoint ORDS ou votre page APEX avec l'URL suivante :

`.../v1/employees?valid=true&userid=999`

Votre code renverra `TRUE`, car le mot `id` est présent dans `valid` et `userid`.

**L'astuce de routier PL/SQL :** pour éviter ce faux positif, encapsulez toujours votre `QUERY_STRING` et la recherche avec des esperluettes (`&`) et un signe égal (`=`), comme ceci :

```sql
INSTR('&' || vva_query_string || '&', '&' || v_param || '=')
```
## La mise en pratique : le snippet robuste

Voici comment intégrer cette logique dans un bloc PL/SQL propre, prêt à l'emploi pour vos process APEX ou vos handlers REST ORDS :

```sql
DECLARE
    vva_query_string  VARCHAR2(4000);
    vva_param_to_find VARCHAR2(100) := 'id'; -- Le paramètre exact à chercher
    vbl_has_param     BOOLEAN := FALSE;
BEGIN
    -- 1. Récupération de la query string via le package Web Toolkit
    vva_query_string := OWA_UTIL.get_cgi_env('QUERY_STRING');

    -- 2. Application de la règle d'isolation
    IF vva_query_string IS NOT NULL THEN
        IF INSTR('&' || vva_query_string || '&', '&' || vva_param_to_find || '=') > 0 THEN
            vbl_has_param := TRUE;
        END IF;
    END IF;

    -- 3. Traitement business
    IF vbl_has_param THEN
        -- Le paramètre exact 'id' est bien présent
        apex_debug.info('Paramètre trouvé ! Traitement en cours...');
    ELSE
        -- Paramètre absent (ou faux positif évité !)
        apex_debug.info('Paramètre introuvable.');
    END IF;
END;
```
## Pourquoi cette logique est infaillible ?

En cherchant explicitement `&id=`, le moteur PL/SQL valide deux règles d'or :

- le paramètre commence forcément après un séparateur HTTP (`&` forcé au début ou existant dans la chaîne) ;
- le nom du paramètre s'arrête précisément avant le signe `=`.

## ORDS natif vs OWA_UTIL : le mot de la fin

Si vous développez un handler REST directement dans l'interface d'ORDS, vous pouvez également déclarer des paramètres de type URI pour mapper automatiquement vos variables.

Cependant, dès que vous basculez dans des packages PL/SQL globaux, des triggers ou du code générique qui n'a pas conscience du contexte ORDS, `OWA_UTIL` reste votre joker universel. La compatibilité est totale, c'est performant, et avec cette astuce d'encapsulation, votre code devient blindé contre les surprises du web.
