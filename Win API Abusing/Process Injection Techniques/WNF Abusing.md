### 1 - Introduction


La fonction de notification Windows est un mécanisme de notification user/kernel pubsub (Publisher/Subscriber), cette fonctionnalité a été ajouté pour servir
une base pour les notifications push basées sur les mobiles centrées sur les applications similaires à iOS / Android
Son principal différenciateur est qu'il s'agit d'un modèle aveugle (essentiellement sans enregistrement) qui permet des
abonnement vs publication


 Nous entendons par là qu'un consommateur peut souscrire à une notification avant même que la notification ait été publiée par son
producteur
 Et qu'il n'y a aucune obligation pour le producteur d'«&nbsp;enregistrer&nbsp;» la notification à l'avance
 En plus de cela, il prend également en charge les notifications persistantes ou volatiles, augmentant de manière monotone le tampon de changement unique
ID, charges utiles jusqu'à 4 Ko pour chaque événement de notification, un modèle de notification basé sur un pool de threads avec un groupe
la sérialisation et un modèle de sécurité basé à la fois sur la portée et implémentant la sécurité Windows Descripteurs
 via le mécanisme standard DACL/SACL [Run-On Sentence Pwnie Award]







**References:**
- https://www.youtube.com/watch?v=MybmgE95weo&ab_channel=BlackHat
- https://modexp.wordpress.com/2019/06/15/4083/
