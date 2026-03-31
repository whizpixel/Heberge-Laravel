# 🌐 SISR — Déploiement du projet Laravel (CRM Web)

> Projet InfoTools · BTS SIO 2024  
> Prérequis : serveur MySQL déjà configuré (voir `SISR-mysql.md`)

---

## 📦 Étape 1 — Installer les dépendances serveur

```bash
sudo apt update && sudo apt upgrade -y

# PHP 8.4 + extensions requises par Laravel
sudo apt install php8.4 php8.4-fpm php8.4-mysql php8.4-mbstring \
     php8.4-xml php8.4-curl php8.4-zip php8.4-bcmath php8.4-tokenizer -y

     # Nginx + Composer + Git
     sudo apt install nginx composer git -y
     ```

     Vérifier les versions :

     ```bash
     php -v
     nginx -v
     composer -V
     ```

     ---

     ## 📁 Étape 2 — Récupérer le projet

     ```bash
     cd /var/www
     sudo git clone https://github.com/whizpixel/web-ap.git
     sudo chown -R www-data:www-data web-ap
     sudo chmod -R 755 web-ap
     sudo chmod -R 775 web-ap/storage web-ap/bootstrap/cache
     ```

     ---

     ## ⚙️ Étape 3 — Configurer l'environnement Laravel

     ```bash
     cd /var/www/web-ap

     # Copier le fichier d'environnement
     sudo cp .env.example .env

     # Installer les dépendances PHP
     sudo composer install --no-dev --optimize-autoloader

     # Générer la clé d'application
     sudo php artisan key:generate
     ```

     Éditer le `.env` avec les infos de la BDD :

     ```bash
     sudo nano .env
     ```

     Modifier ces lignes :

     ```env
     APP_ENV=production
     APP_DEBUG=false
     APP_URL=http://<IP_DU_SERVEUR_WEB>

     DB_CONNECTION=mysql
     DB_HOST=<IP_DU_SERVEUR_BDD>
     DB_PORT=3306
     DB_DATABASE=infotools_crm
     DB_USERNAME=infotools
     DB_PASSWORD=MotDePasseForte2024!
     ```

     ---

     ## 🗃️ Étape 4 — Migrer la base de données

     ```bash
     sudo php artisan migrate --force
     sudo php artisan db:seed --force
     ```

     Optimiser pour la production :

     ```bash
     sudo php artisan config:cache
     sudo php artisan route:cache
     sudo php artisan view:cache
     ```

     ---

     ## 🌐 Étape 5 — Configurer Nginx

     Créer le fichier de configuration du site :

     ```bash
     sudo nano /etc/nginx/sites-available/infotools-crm
     ```

     Coller la configuration suivante :

     ```nginx
     server {
         listen 80;
             server_name <IP_DU_SERVEUR_WEB>;

                 root /var/www/web-ap/public;
                     index index.php index.html;

                         # Réécriture des URLs Laravel
                             location / {
                                     try_files $uri $uri/ /index.php?$query_string;
                                         }

                                             # Traitement PHP via PHP-FPM
                                                 location ~ \.php$ {
                                                         fastcgi_pass unix:/run/php/php8.4-fpm.sock;
                                                                 fastcgi_index index.php;
                                                                         fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
                                                                                 include fastcgi_params;
                                                                                     }

                                                                                         # Bloquer l'accès aux fichiers cachés
                                                                                             location ~ /\.(?!well-known).* {
                                                                                                     deny all;
                                                                                                         }

                                                                                                             # Logs
                                                                                                                 access_log /var/log/nginx/infotools-crm.access.log;
                                                                                                                     error_log  /var/log/nginx/infotools-crm.error.log;
                                                                                                                     }
                                                                                                                     ```

                                                                                                                     Activer le site et redémarrer Nginx :

                                                                                                                     ```bash
                                                                                                                     sudo ln -s /etc/nginx/sites-available/infotools-crm /etc/nginx/sites-enabled/
                                                                                                                     sudo nginx -t
                                                                                                                     sudo systemctl restart nginx
                                                                                                                     sudo systemctl restart php8.4-fpm
                                                                                                                     ```

                                                                                                                     ---

                                                                                                                     ## ✅ Étape 6 — Vérifier que tout fonctionne

                                                                                                                     Ouvrir un navigateur depuis le LAN et accéder à :

                                                                                                                     ```
                                                                                                                     http://<IP_DU_SERVEUR_WEB>
                                                                                                                     ```

                                                                                                                     La page de connexion Laravel doit s'afficher.

                                                                                                                     Pour tester la connexion BDD :

                                                                                                                     ```bash
                                                                                                                     sudo php artisan tinker
                                                                                                                     >>> App\Models\User::count()
                                                                                                                     ```

                                                                                                                     Si un nombre s'affiche, la BDD est bien connectée. ✅

                                                                                                                     ---

                                                                                                                     ## 🔧 Dépannage rapide

                                                                                                                     | Problème | Solution |
                                                                                                                     |----------|----------|
                                                                                                                     | Page blanche | `sudo php artisan config:clear` puis vérifier les logs : `sudo tail -f /var/log/nginx/infotools-crm.error.log` |
                                                                                                                     | Erreur 500 | `sudo tail -f /var/www/web-ap/storage/logs/laravel.log` |
                                                                                                                     | Permissions refusées | `sudo chown -R www-data:www-data /var/www/web-ap/storage` |
                                                                                                                     | PHP-FPM introuvable | `sudo systemctl status php8.4-fpm` |

                                                                                                                     ---

                                                                                                                     ## ⚙️ Configuration .env à transmettre à l'équipe SLAM

                                                                                                                     Une fois le serveur en place, communiquer ces infos :

                                                                                                                     ```env
                                                                                                                     APP_URL=http://<IP_DU_SERVEUR_WEB>
                                                                                                                     DB_HOST=<IP_DU_SERVEUR_BDD>
                                                                                                                     DB_DATABASE=infotools_crm
                                                                                                                     DB_USERNAME=infotools
                                                                                                                     DB_PASSWORD=MotDePasseForte2024!
                                                                                                                     ```

                                                                                                                     > ⚠️ Ne pas oublier de noter les IPs et mots de passe dans le fichier de mots de passe du dossier projet SISR.
                                                                                                                     