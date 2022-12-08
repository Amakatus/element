# Procédure 4
## Sacha Bouton et Benoît Misplon

## 1) Element Web

On a choisit apache : Pourquoi ? Et pourquoi pas nginx?


sudo -E apt update
sudo -E apt install apache2
Désinstaller nginx si on utilise apache2:

sudo apt autoremove nginx --purge -y

Afin d'activer APACHE :
se placer dans /etc/apache2/sites-available
user@vm:/etc/apache2/sites-available $ sudo a2dissite 000-default.conf
user@vm:/etc/apache2/sites-available $ sudo a2dissite default-ssl.conf
user@vm:/etc/apache2/sites-available $ sudo cp 000-default.conf websites.conf
user@vm:/etc/apache2/sites-available $ sudo rm 000-default.conf
user@vm:/etc/apache2/sites-available $ sudo a2ensite websites.conf
user@vm:/etc/apache2/sites-available $ sudo systemctl reload apache2
user@vm:/etc/apache2/ $ sudo nano ports.conf Modif port 8080
user@vm:/etc/apache2/ $ sudo nano sites-available/websites.conf Modif port 8080
user@vm:/etc/apache2/sites-available $ sudo a2ensite websites.conf
user@vm:/etc/apache2/sites-available $ sudo systemctl restart apache2

Pour tester si notre serveur marche bien :
On peut modifier le fichier /var/www/html
Inserer image apache2

dl element:
https://github.com/vector-im/element-web

wget https://github.com/vector-im/element-web/releases/download/v1.11.16/element-v1.11.16.tar.gz
tar -xvzf element-v1.11.16.tar.gz 

Ensuite, config json faut le cp  


cd element-serv/
ls
cp config.sample.json config.json
ls
cd ..
ls
cp -R element-serv/ /var/www/
sudo cp -R element-serv/ /var/www/
cd /var/www/
ls
sudo rm -Rf html/
mv element-serv/ html
sudo mv element-serv/ html
