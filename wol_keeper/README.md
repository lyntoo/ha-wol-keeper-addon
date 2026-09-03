# WoL Keeper

Garde le Wake-on-LAN (magic packet) actif sur une carte réseau, même après un arrêt propre de Home Assistant OS.

## Pourquoi

Sur beaucoup de systèmes (notamment avec des cartes réseau Realtek `r8169`), le pilote réseau **désactive** le mode Wake-on-LAN à chaque démarrage ou après un arrêt propre — même si l'option est activée dans le BIOS/UEFI. Résultat : un `hassio.host_shutdown` (ou tout arrêt propre) empêche ensuite de rallumer la machine à distance avec un paquet magique, ce qui est exactement le contraire de ce que le BIOS promet.

Cet add-on tourne en arrière-plan et réactive automatiquement le mode magic packet (`ethtool -s <interface> wol g`) dès qu'il détecte qu'il a été désactivé — au démarrage, et à intervalle régulier ensuite.

## Prérequis

- Le Wake-on-LAN doit être activé au niveau BIOS/UEFI (variable selon le fabricant — souvent sous *Power Management* ou *Integrated NIC*).
- Certains fabricants exigent aussi de désactiver un réglage de type *Deep Sleep* / *ErP* / *EuP* pour que la carte réseau garde son alimentation de veille à l'arrêt complet.

## Configuration

| Option | Description | Défaut |
|---|---|---|
| `interface` | Nom de la carte réseau (ex: `eth0`, `enp2s0`). Laisser vide pour une détection automatique de la première carte physique disponible. | `""` (auto) |
| `check_interval` | Intervalle en secondes entre chaque vérification/réactivation. | `300` |

## Limites connues

- Un arrêt **complet du courant** (pas juste le système éteint proprement) peut nécessiter que la carte réseau retrouve son alimentation de veille — ce n'est pas quelque chose que cet add-on peut contrôler, ça dépend du matériel/BIOS.
- Cet add-on ne peut pas activer le Wake-on-LAN si le BIOS/UEFI lui-même ne le permet pas au démarrage — c'est un complément au réglage BIOS, pas un remplacement.
