# Module Contract

Chaque module expose un dictionnaire `MODULE`. Le chargeur lit ce contrat pour
construire l'interface et calculer la commande a executer dans le conteneur
toolbox.

## Champs

| Champ | Type | Obligatoire | Description |
| --- | --- | --- | --- |
| `name` | string | oui | Nom affiche dans l'interface. |
| `description` | string | oui | Resume court du module. |
| `category` | string | oui | Regroupement fonctionnel. |
| `schema` | list | oui | Definition des champs utilisateur. |
| `cmd` | callable | oui | Fonction qui transforme les parametres en commande. |
| `hidden_from_list` | bool | non | Masque un module technique de la page principale. |

## Exemple minimal

```python
import shlex

MODULE = {
    "name": "HTTP Headers Review",
    "description": "Collecte les en-tetes HTTP pour une cible web.",
    "category": "Web",
    "schema": [
        {
            "group_name": "Target",
            "fields": [
                {"name": "url", "type": "string", "required": True, "placeholder": "https://example.com"},
            ],
        }
    ],
    "cmd": lambda p: ["bash", "-lc", f"curl -I --max-time 10 {shlex.quote(p['url'])}"],
}
```

## Regles de qualite

- Valider et echapper toute entree utilisateur.
- Preferer une commande lisible et auditable a une chaine complexe.
- Eviter les secrets en argument de ligne de commande.
- Documenter les effets intrusifs potentiels dans la description.
- Tester le module en dry-run ou sur lab avant usage mission.

