<!-- https://josejuansanchez.org/iaw/practica-prestashop-docker/index.html -->

# Activitat **```a10u```**: **```PrestaShop```** amb **```docker compose```**

**Objectiu**: Instal·lació de **```PrestaShop```** utilitzant els contenidors **```Docker Compose```**

Creació de la carpeta que contindrà el sistema de contenidors.

```bash
sudo mkdir ~/c03-prestashop
cd ~/c03-prestashop
```

## **Pas 1**: Requisits de l'arxiu **```docker-compose.yml```**

### **1.1.** Imatges

Les imatges que farem servir són:

|Contenidor|Imatge|**```tag```**|
|---|---|---|
|**```prestashop```**|**```prestashop/prestashop```**|**```latest```**|
|**```mysql```**|**```mysql```**|**```5.7```**|

### **1.2.** Ports

Només cal exposar el **```port```** **```8089```** del **host** i redirigir-lo al **```port```** **```80```** del contenidor amb el nom **```prestashop```**.

|Contenidor|Port del<br>host|Port del<br>contenidor|
|---|---|---|
|**```prestashop```**|**```8089```**|**```80```**|

### **1.3.** Volums

Els **volums definits** a l'arxiu **```docker-compose.yml```** hauran de ser el següents:

|Contenidor|carpeta<br>host|carpeta<br>contenidor|
|---|---|---|
|**```prestashop```**|**```./data```**|**```/var/www/html```**|
|**```mysql```**|**```./db```**|**```/var/lib/mysql```**|

### **1.3.** Polítiques de reinici de Docker

Caldrà fer ùs de la política de reinici per als dos contenidors per que es reiniciin cada vegada que es detenguin de forma inesperada.

> [!TIP]
> Es recomana consultar [la **documentació oficial** de l'opció **```restart```**](https://docs.docker.com/compose/compose-file/compose-file-v3/#restart).

### **1.3** Variables d'entorn

Cal fer servir un fitxer **```.env```** per emmagatzemar totes les variables de l'entorn necessaries a l'arxiu **```docker-compose.yml```**.

> [!TIP]
> A la documentació oficial podeu trobar més informació sobre com fer l'ús de variables d'entorn a l'arxiu **```docker-compose.yml```** [**Substituïu-lo amb un fitxer ```.env```**](https://docs.docker.com/compose/environment-variables/set-environment-variables/#compose-file)

<!-- ### **1.4**  Ordre en el que s'inicien els **serveis**

Cal indicar l'ordre en el que s'ha d'iniciar els serveis amb l'opció **```depends_on```**.

> [!TIP]
> Se recomana la lectura de l'article [**Controla l'ordre d'inici i apagat a ```compose```** - Control startup and shutdown order in Compose](https://docs.docker.com/compose/startup-order/)

Per garantir que el servei de **```MySQL```** està llest per acceptar connexions, haurà d'utilitzar l'opció **```healthcheck```** de l'arxiu **```docker-compose.yml```**. 

> [!TIP]
> Es recomana la lectura de l'article [**```healthcheck```** - Compose file version 3 reference][https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck].-->

## **Pas 2**: Exemple d'arxiu **```docker-compose.yml```** utilitzant la imatge **```prestashop/prestashop```**

A continuació es mostra una possible solució de la pràctica utilitzant la imatge de **PrestaShop** **```prestashop/prestashop```** .

```bash
sudo vi ~/c03-prestashop/.env
```

* Contingut de l'arxiu **```.env```** que conté la configuració personal del sistema de contenidors:

```yml
# Configuració de l'acces a la base de dades
MYSQL_DATABASE=exampledb
MYSQL_USER=userps
MYSQL_PASSWORD=passps
MYSQL_ROOT_PASSWORD=12345
```

```bash
sudo vi ~/c03-prestashop/docker-compose.yml
```

* Contingut de l'arxiu **```docker-compose.yml```**:

```yml
version: "3"
services:
  prestashop:
    image: prestashop/prestashop
    volumes:
    - ./data:/var/www/html
    ports:
    - "8099:80"
    restart: always

  basedades:
    image: mysql:5.7
    restart: always
    environment:
    - MYSQL_DATABASE=pardodb
    - MYSQL_USER=usuarips
    - MYSQL_PASSWORD=motdepasps
    - MYSQL_ROOT_PASSWORD=motdepasroot
    volumes:
    - ./db:/var/lib/mysql
```


