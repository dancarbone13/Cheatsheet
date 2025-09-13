# Cheatsheet

## Comment voir si linux integre un agent zabbix?

1. dpkg -l | grep zabbix-agent
2. systemctl status zabbix-agent
3. ps aux | grep zabbix_agentd

## Logs agent zabbix : /var/log/zabbix/zabbix_agentd.log

## Surveillance du processus GitLab Runner

Clé : proc.num[gitlab-runner]
Valeur attendue : 1 (ou plus, selon le nombre d'instances).
Type d'information : Numeric (unsigned)
Utilisation : Crée un trigger si la valeur est 0 (runner arrêté).
Exemple de trigger : {Template_GitLab_Runner:proc.num[gitlab-runner].last(0)}=0
Description : "Le processus GitLab Runner ne tourne pas."
## Surveillance du service systemd de GitLab Runner

| Élément                | Détails                                                                                     |
|------------------------|---------------------------------------------------------------------------------------------|
| **Clé Zabbix**         | `systemd.unit.info[gitlab-runner.service,ActiveState]`                                      |
| **Valeur attendue**    | `active`                                                                                   |
| **Type d'information** | Text                                                                                       |
| **Utilisation**        | Créer un trigger si la valeur n'est pas `active`.                                           |
| **Exemple de trigger** | `{Template_GitLab_Runner:systemd.unit.info[gitlab-runner.service,ActiveState].str("active")}=0` |
| **Description**        | "Le service GitLab Runner est inactif ou en échec."                                         |
| **Séverité suggérée**  | High                                                                                       |
| **Actions suggérées**  | Vérifier le statut du service avec `systemctl status gitlab-runner.service`, redémarrer si nécessaire. |

## Surveillance du fichier de log de GitLab Runner

| Élément                | Détails                                                                                     |
|------------------------|---------------------------------------------------------------------------------------------|
| **Clé Zabbix**         | `vfs.file.time[/var/log/gitlab-runner/gitlab-runner.log,modify]`                             |
| **Type d'information** | Numeric (timestamp)                                                                         |
| **Utilisation**        | Créer un trigger si le timestamp n'a pas changé depuis X minutes.                           |
| **Exemple de trigger** | `{Template_GitLab_Runner:vfs.file.time[/var/log/gitlab-runner/gitlab-runner.log,modify].time()} < {$LOG_MODIFIED_MAX_AGE}` |
| **Description**        | "Le log GitLab Runner n'a pas été mis à jour depuis {$LOG_MODIFIED_MAX_AGE} secondes."       |
| **Macro recommandée**  | `{$LOG_MODIFIED_MAX_AGE}` (ex: `300` pour 5 minutes, `600` pour 10 minutes)                  |
| **Séverité suggérée**  | Warning (alerte précoce) ou High (si critique)                                              |
| **Actions suggérées**  | Vérifier l'activité des jobs, l'état du service, ou redémarrer le service GitLab Runner.    |

## Surveillance de l'utilisation CPU/Mémoire
Tu peux surveiller l'utilisation des ressources par le processus gitlab-runner :

## Triggers pour la surveillance de GitLab Runner

| Catégorie      | Trigger                                      | Séverité  | Description                          |
|----------------|----------------------------------------------|-----------|--------------------------------------|
| Disponibilité  | `proc.num[gitlab-runner].last(0)}=0`         | High      | Processus GitLab Runner arrêté       |
| CPU            | `proc.cpu.util[gitlab-runner].avg(5m)}>80`   | Warning   | Utilisation CPU élevée (>80% sur 5 min) |
| CPU            | `proc.cpu.util[gitlab-runner].last(0)}=0`    | High      | Processus GitLab Runner inactif      |
| RAM            | `proc.mem[gitlab-runner,,,rss].last(0)}>1G`  | Warning   | Utilisation mémoire élevée (>1 Go)   |
| RAM            | `proc.mem[gitlab-runner,,,rss].last(0)}=0`   | High      | Processus ne consomme pas de mémoire |

## Crée des triggers si l'utilisation est à 0 (processus inactif) ou trop élevée (problème de performance).

## Triggers pour la surveillance des ressources

| Catégorie      | Trigger                                           | Séverité  | Description                          |
|----------------|---------------------------------------------------|-----------|--------------------------------------|
| Disponibilité  | `proc.num[gitlab-runner].last(0)}=0`              | High      | Processus arrêté                     |
| Disponibilité  | `systemd.unit.info[...].str("active")}=0`          | High      | Service inactif                      |
| CPU            | `proc.cpu.util[...].avg(5m)}>80`                   | Warning   | Utilisation CPU élevée               |
| CPU            | `proc.cpu.util[...].last(0)}=0`                    | High      | Processus inactif                    |
| RAM            | `proc.mem[...].last(0)}>1G`                        | Warning   | Utilisation mémoire élevée           |
| RAM            | `proc.mem[...].last(0)}=0`                         | High      | Processus ne consomme pas de mémoire |
| Disque         | `vfs.fs.size[/,free].last(0)}<10G`                 | High      | Espace disque racine faible          |
| Disque         | `vfs.fs.size[/var/lib/docker,free].last(0)}<5G`    | High      | Espace disque Docker faible          |
| Log            | `vfs.file.time[...].diff()}=0`                     | Warning   | Log non mis à jour                   |

