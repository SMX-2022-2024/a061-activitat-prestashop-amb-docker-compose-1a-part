<!-- https://josejuansanchez.org/iaw/practica-prestashop-docker/index.html -->

# Activitat **```a10u```**: **```PrestaShop```** amb **```docker compose```**

**Objectiu**: Instal·lació de **```PrestaShop```** utilitzant els contenidors **```Docker Compose```**

Creació de la carpeta que contindrà el sistema de contenidors.

* Comanda a executar:

```bash
sudo mkdir ~/c03-ps
cd ~/c03-ps
```

* Sortida:

<pre>
profe@docker-sxm:~$ sudo mkdir ~/c03-ps
cd ~/c03-ps
profe@docker-sxm:~/c03-ps$ 
</pre>

## **Pas 1**: Requisits de l'arxiu **```docker-compose.yml```**

### **1.1.** Imatges

Les imatges que farem servir són:

|Contenidor|Imatge|**```tag```**|
|---|---|---|
|**```prestashop```**|**```prestashop/prestashop```**|**```1.7```**|
|**```mysql```**|**```mysql```**|**```5.7```**|

* Comanda a executar:

```bash
sudo docker image list
```

* Sortida:

<pre>
profe@docker-sxm:~/c03-ps$ sudo docker image list
REPOSITORY                TAG            IMAGE ID       CREATED        SIZE
prestashop/prestashop     1.7            6e6ff1a2495e   2 hours ago    1.24GB
prestashop/prestashop     latest         9712f852ca81   3 months ago   1.42GB
prestashop/prestashop     1.7.8          fe5bfe08a491   4 months ago   1.24GB
...
mysql                     8              73246731c4b0   2 weeks ago    619MB
mysql                     5.7            bdba757bc933   2 months ago   501MB
...
profe@docker-sxm:~/c03-ps$ 
</pre>

Es pot mostrar la informació de les imatges segons un criteri de cerca.

**Exemple 1**: si volem veure només aquelles imatges amb el **nom** **```prestashop/prestashop```**

* Comanda a executar:

```bash
sudo docker image list --filter reference=prestashop/prestashop
```

* Sortida:

<pre>
profe@docker-sxm:~/c03-ps$ sudo docker image list --filter reference=prestashop/prestashop
REPOSITORY              TAG       IMAGE ID       CREATED        SIZE
prestashop/prestashop   1.7       6e6ff1a2495e   2 hours ago    1.24GB
prestashop/prestashop   latest    9712f852ca81   3 months ago   1.42GB
prestashop/prestashop   1.7.8     fe5bfe08a491   4 months ago   1.24GB
profe@docker-sxm:~/c03-ps$ 
</pre>

**Exemple 2**: si volem veure només aquelles imatges amb el **nom** **```mysql```**

* Comanda a executar:

```bash
sudo docker image list --filter reference=mysql
```

* Sortida:

<pre>
profe@docker-sxm:~/c03-ps$ sudo docker image list --filter reference=mysql

REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql        8         73246731c4b0   2 weeks ago    619MB
mysql        5.7       bdba757bc933   2 months ago   501MB
</pre>


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

### **1.4.** Polítiques de reinici de Docker

Caldrà fer ùs de la política de reinici per als dos contenidors per que es reiniciin cada vegada que es detenguin de forma inesperada.

> [!TIP]
> Es recomana consultar [la **documentació oficial** de l'opció **```restart```**](https://docs.docker.com/compose/compose-file/compose-file-v3/#restart).

### **1.5** Variables d'entorn

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
sudo vi ~/c03-ps/.env
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
sudo vi ~/c03-ps/docker-compose.yml
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


