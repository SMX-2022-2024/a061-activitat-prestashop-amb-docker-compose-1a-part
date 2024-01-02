<!-- https://josejuansanchez.org/iaw/practica-prestashop-docker/index.html -->

# Activitat **```a10u```**: **```PrestaShop```** amb **```docker compose```**

**Objectiu**: Instal·lació de **```PrestaShop```** utilitzant els contenidors **```Docker Compose```**


Creació de la carpeta que contindrà el sistema de contenidors.

```bash
sudo mkdir ~/c03-prestashop
cd ~/c03-prestashop
```

## **Pas 1**: Requisits de l'arxiu **```docker-compose.yml```**

### **1.1.** Xarxes

Els **serveis definits** a l'arxiu **```docker-compose.yml```** hauran d'utilitzar dues xarxes:

* xarxa frontend (**```frontend-network```**)
* xarxa backend (**```backend-network```**)

A la **xarxa ```frontend-network```** estaran els serveis:

* **```https-portal```**,

* **```prestashop```** i

* **```phpmyadmin```**.

I a la **xarxa ```backend-network```** només estarà el servei:

* **```mysql```**

Només els serveis que estan a la xarxa **```frontend-network```** exposaran els seus ports a l'**```host```**. Per tant, el servei **```mysql```** no haurà d'estar accessible des de l'**```host```**.

A continuació es mostra un diagrama amb les xarxes i els serveis que té que crear:

![diagrama.png](./images/diagrama.png)

### **1.2.** Polítiques de reinici de Docker

Caldrà fer ùs d'alguna política de reinici per als contenidors que es reinicien cada vegada que es detenguin de forma inesperada.

> [!TIP]
> Es recomana consultar [la **documentació oficial** de l'opció **```restart```**](https://docs.docker.com/compose/compose-file/compose-file-v3/#restart).

### **1.3** Variables d'entorn

Cal fer servir un fitxer **```.env```** per emmagatzemar totes les variables de l'entorn necessaries a l'arxiu **```docker-compose.yml```**.

> [!TIP]
> A la documentació oficial podeu trobar més informació sobre com fer l'ús de variables d'entorn a l'arxiu **```docker-compose.yml```** [**Substituïu-lo amb un fitxer ```.env```**](https://docs.docker.com/compose/environment-variables/set-environment-variables/#compose-file)

### **1.4**  Ordre en el que s'inicien els **serveis**

Cal indicar l'ordre en el que s'ha d'iniciar els serveis amb l'opció **```depends_on```**.

> [!TIP]
> Se recomana la lectura de l'article [**Controla l'ordre d'inici i apagat a ```compose```** - Control startup and shutdown order in Compose](https://docs.docker.com/compose/startup-order/)

Per garantir que el servei de **```MySQL```** està llest per acceptar connexions, haurà d'utilitzar l'opció **```healthcheck```** de l'arxiu **```docker-compose.yml```**.

> [!TIP]
> Es recomana la lectura de l'article [**```healthcheck```** - Compose file version 3 reference][https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck].

## **Pas 2**: Exemple d'arxiu **```docker-compose.yml```** utilitzant la imatge **```prestashop/prestashop```**

A continuació es mostra una possible solució de la pràctica utilitzant la imatge de **PrestaShop** **```prestashop/prestashop```** .


```bash
sudo vi ~/c03-prestashop/.env
```

* Contingut de l'arxiu **```.env```** que conté la configuració personal del sistema de contenidors:

```yml
# Configuració de l'acces a la base de dades
MYSQL_ROOT_PASSWORD=password
MYSQL_DATABASE=prestashop_db
MYSQL_USER=prestashop_user
MYSQL_PASSWORD=prestashop_password

# Configuració del PrestaShop
PS_DOMAIN=localhost
#PS_DOMAIN=iaw-test.ddns.net
ADMIN_MAIL=admin@mail.es
ADMIN_PASSWD=admin_password
```

```bash
sudo vi ~/c03-prestashop/docker-compose.yml
```

* Contingut de l'arxiu **```docker-compose.yml```**:

```yml
version: '3.3'

services:
  prestashop:
    image: prestashop/prestashop:1.7
    environment:
      - PS_INSTALL_AUTO=${PS_INSTALL_AUTO:-1}
      - DB_SERVER=${DB_SERVER:-mysql}
      - DB_USER=${MYSQL_USER}
      - DB_PASSWD=${MYSQL_PASSWORD}
      - DB_NAME=${MYSQL_DATABASE}
      - PS_DOMAIN=${PS_DOMAIN}
      - PS_ENABLE_SSL=${PS_ENABLE_SSL:-1}
      - PS_FOLDER_INSTALL=${PS_FOLDER_INSTALL:-install-dev}
      - PS_FOLDER_ADMIN=${PS_FOLDER_ADMIN:-admin-dev}
      - PS_COUNTRY=${PS_COUNTRY:-es}
      - PS_LANGUAGE=${PS_LANGUAGE:-es}
      - ADMIN_MAIL=${ADMIN_MAIL}
      - ADMIN_PASSWD=${ADMIN_PASSWD}
    restart: always
    volumes:
      - prestashop_data:/var/www/html
    depends_on:
      - mysql
    networks:
      - frontend_network
      - backend_network
  
  mysql:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    restart: always
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - backend_network
    security_opt:
      - seccomp:unconfined

  phpmyadmin:
    image: phpmyadmin:5
    restart: always
    ports:
      - 8080:80
    environment:
      - PMA_HOST=mysql
    depends_on:
      - mysql
    networks:
      - frontend_network
      - backend_network

  https-portal:
    image: steveltn/https-portal:1
    ports:
      - 80:80
      - 443:443
    environment:
      DOMAINS: 'localhost -> http://prestashop:80 #local'
      #DOMAINS: '${PS_DOMAIN} -> http://prestashop:80 #production'
    volumes:
      - ssl_certs_data:/var/lib/https-portal
    depends_on:
      - prestashop
    restart: always
    networks:
      - frontend_network

volumes:
  prestashop_data:
  mysql_data:
  ssl_certs_data:

networks:
  frontend_network:
  backend_network:
```


