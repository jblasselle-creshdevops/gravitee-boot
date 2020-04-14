# Poc Gravitee.io

# Gravitee start



## Installation

### Methode 1 : one comamnd quick start


```bash
curl -L http://bit.ly/graviteeio-am | bash -s 8080
```



### Methode 2 : Une installation testée qui donne un crash gravitee.io (buggé pas d'ouverture d'issue github pour l'instant)

* https://docs.gravitee.io/apim/1.x/apim_installguide_docker_compose.html
* https://docs.gravitee.io/am/2.x/am_installguide_docker.html#docker_compose
* version : https://github.com/gravitee-io/gravitee-gateway/releases/tag/1.30.8
* Edit the `/etc/hosts` of the machine on which you will run the gravitee installation, and append to it the following  :

```ini
# ---
#
# Gravitee.io Cresh
192.168.1.22    pegasusio.io apim.gravitee.io am.gravitee.io

```

* Now run :

```bash

URI_GIT=git@gitlab.com:bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio.git
git clone $URI_GIT ~/.poc.graviteeio
cd ~/.poc.graviteeio

# pour elastic search
sudo sysctl -w vm.max_map_count=262144
#

# (Optional step: pull to ensure that you are running latest images)
docker-compose pull

# And run...
docker-compose up -d && docker-compose logs -f


```

* upgrade/downgrade to version :

```bash
export GRAVITEE_VERSION=1.30.8
export DWNLOAD_URI=https://raw.githubusercontent.com/gravitee-io/gravitee-docker/master/apim/${GRAVITEE_VERSION}/docker-compose.yml
curl -L https://raw.githubusercontent.com/gravitee-io/gravitee-docker/master/apim/1.x/docker-compose.yml -o "docker-compose.yml"
```



* ajouter la ligne suivante dans le `/etc/hosts` :

```bash
# --- #
127.0.0.1       localhost apim.gravitee.io am.gravitee.io

````
* accéder à gravitee.io avec http://localhost:8077/am/ui (poru l(instant je n'arrive pas à modifier la configuration popur l'accès externe)
* faire le premier login avec :
  * le username `admin`
  * le mot de passe `adminadmin`
* https://docs.gravitee.io/am/2.x/am_userguide_authentication.html
*




#### Create a security domain

>
>  security domain is a series of security policies apply to a set of applications that all share common security mechanisms for authentication, authorization and identity management.
>

* From the homepage, go to user menu (top right) and click Global settings

* From the domains page, click (+) button

* Give your security domain a name, a description and press SAVE

* Last step, enable your domain by clicking on the banner click here link. (une notification dans la WebUI)

_**Même action avec les API token Gravitee**_

```bash
# create domain
curl -H "Authorization: Bearer :accessToken" \
     -H "Content-Type:application/json;charset=UTF-8" \
     -X POST \
     -d '{"name":"My First Security Domain","description":"My First Security Domain description"}' \
     http://GRAVITEEIO-AM-MGT-API-HOST/management/domains


# enable domain
curl -H "Authorization: Bearer :accessToken" \
     -H "Content-Type:application/json;charset=UTF-8" \
     -X PUT \
     -d '{"name":"My First Security Domain","description":"My First Security Domain description", enabled: true}' \
     http://GRAVITEEIO-AM-MGT-API-HOST/management/domains/:domainId
```


#### Create your client


>
> Before interact with the AM Authorization Gateway, you must create a client. The client represents your application and will give you the necessary information (like the **client ID/client Secret**) **for sign-in**, authorization, authentication and identity management.
>

>
> The application **can be** a native **mobile app**, a **single page front-end web app**, or a **regular web app** that executes on a server.
>

* D'abord, il faut "se positionner" sur le domaine de sécurité qui vient d'être définit (`creshdomain` pour mon exemple) :

![select domain](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_IO_SWITCH_DOMAINS_2020-04-14T15-56-13.767Z.png?inline=false)

* Ensuite, il faut aller au menu "Clients" et créer le client avec la WebUI (je l'ai appelé `creshapi_client`)

![create client 1](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_CRESH_API_CREATE_CLIENT_1_2020-04-14T16-06-27.507Z.png)

* From the resulting OAuth client create form, you can copy the Client ID and the Client Secret to start interacting with Gravitee.io AM endpoints and APIs :

![create client 2](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_CRESH_API_CREATE_CLIENT_2_2020-04-14T16-06-42.511Z.png)

_**Même action avec les API token Gravitee**_

```bash
curl -H "Authorization: Bearer :accessToken" \
     -H "Content-Type:application/json;charset=UTF-8" \
     -X POST \
     -d '{"clientId":"THE-CLIENT-ID"}' \
     http://GRAVITEEIO-AM-MGT-API-HOST/management/domains/:domainId/clients
```

#### Maintenant Testons

https://docs.gravitee.io/am/2.x/am_quickstart_register_app.html#test_your_application_with_oauth2

Le test proposé par la doc consuste à tester l'authentification application avec `OAuth2`

* Dans le process d'authentification OAuth2, comme la Docker Auth v2, on doit donc d'abord obtenir un token,
* puis utiliser ce token pour consommer le endpoint gravitee que l'on vien de déclarer

* Voici comment Obtenir le token OAuth2 :
  * d'abord, il faut définir une _redirect_url_ pour l'authentification OAuth2 à notre client `creshapi` :

![definir redirect_uri, etape 1](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_CLIENT_REDIRECT_URI_FOR_OAUTH2.png?inline=false)

  * ensuite, il faut activer le identity provider par défault:
  * puis, configurer l'authentification Oauth2 du client en créer un scope (ci-dessous le scope `lecture`) :


![config minimale OAuth2 client](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_CLIENT_CONFIG_CLIENT_MINIMALE_OAUTH2.png?inline=false)



```bash
export CLIENT_ID=creshAPI_ClientID_JBL
export CLIENT_SECRET=d8JKL9MX6d24xP376Y5q_SscLGs
export GRAVITEE_HOST=localhost:8077

export GRAVITEE_HOST=apim.gravitee.io
export GRAVITEE_HOST=am.gravitee.io

export CRESH_API_SECURTIY_DOMAIN=creshdomaine
export CRESH_API_GRAVITEE_CLIENT_SCOPE=lecture

curl -X POST \
  "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTIY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"


```

* tests :

```bash

jbl@poste-devops-jbl-16gbram:~$ curl -X POST  -i  'http://am.gravitee.io:8077/am/management/${CRESH_API_SECURTIY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=:${CLIENT_ID}&client_secret=:${CLIENT_SECRET}'
HTTP/1.1 401 Authentication Failed: No JWT token found
Server: nginx/1.16.1
Date: Tue, 14 Apr 2020 18:41:46 GMT
Content-Type: text/html;charset=iso-8859-1
Content-Length: 0
Connection: keep-alive
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Pragma: no-cache
X-Frame-Options: DENY
Cache-Control: must-revalidate,no-cache,no-store

jbl@poste-devops-jbl-16gbram:~$

```

* Ok, malgré ces essais infructueux, on peut lire ceci, dans le docker-compose :

```Yaml
environment:
  - MGMT_API_URL=http://localhost:${NGINX_PORT}/am
  - MGMT_UI_URL=http://localhost:${NGINX_PORT}/am/ui
```

* donc, pour moi, j'ai bien le bon appel à faire lorsque je fait :

```bash
$ curl -X POST  -i  'http://localhost:8077/am/${CRESH_API_SECURTIY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=:${CLIENT_ID}&client_secret=:${CLIENT_SECRET}'
HTTP/1.1 404 Not Found
Server: nginx/1.16.1
Date: Tue, 14 Apr 2020 18:56:50 GMT
Content-Type: text/plain
Content-Length: 43
Connection: keep-alive
X-Gravitee-Transaction-Id: f98f6056-0055-4ff7-8f60-5600559ff722

No security domain matches the request URI.
```

* Ah, ça y est (erreur bête de quotes) :

```bash
$ curl -X POST \
>   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTIY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "error" : "invalid_client",
  "error_description" : "Invalid client: client must at least have one grant type configured"
}
```

* à partir de là, j'ai dépilé les erreurs suivantes , et les ait toutes résolues en changeant ou ajoutant un paramètre de configuration du client dans la Web UI :

```bash

jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "error" : "invalid_client",
  "error_description" : "Invalid client: client must at least have one grant type configured"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$
jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$
jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTIY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "error" : "invalid_scope",
  "error_description" : "Invalid scope(s): read"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "error" : "invalid_scope",
  "error_description" : "Invalid scope(s): read"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "error" : "invalid_scope",
  "error_description" : "Invalid scope(s): read"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=scim&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "error" : "invalid_scope",
  "error_description" : "Invalid scope(s): scim"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=scim&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "access_token" : "eyJraWQiOiJkZWZhdWx0LWdyYXZpdGVlLUFNLWtleSIsImFsZyI6IkhTMjU2In0.eyJzdWIiOiJjcmVzaEFQSV9DbGllbnRJRF9KQkwiLCJhdWQiOiJjcmVzaEFQSV9DbGllbnRJRF9KQkwiLCJkb21haW4iOiJjcmVzaGRvbWFpbmUiLCJzY29wZSI6InNjaW0iLCJpc3MiOiJodHRwOi8vYW0uZ3Jhdml0ZWUuaW8vYW0vY3Jlc2hkb21haW5lL29pZGMiLCJleHAiOjE1ODY5MDAzMTgsImlhdCI6MTU4Njg5MzExOCwianRpIjoiM3o4TDdFeDVfM2VCVEhsY1VqdW5SeGQ3ckM4In0.mP2IX4KrUSDggjDCHqOMFtIIRa4NyE3fqpw5jXWGLbk",
  "token_type" : "bearer",
  "expires_in" : 7199,
  "scope" : "scim"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$

```

* les conf du client dans la WebUI :

  * Il faut "activer", mettre dans l'état "enabled", le client `creshapi_client` (bouton bleu un peu en haut à gauche dans le formulaire du client) :
![client dans l'état "enabled"](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/CRESH_API_CLIENT_CONFIG-CLIENT_ENABLED_2020-04-14T19-42-11.536Z.png?inline=false)

  * Il faut "activer",  mettre dans l'état "enabled", au moins un Identity Provider, pour le client `creshapi_client`,  (Ici j'ai configuré le provider  par défaut, mais ce pourrait être `Keycloak`) :

![client dans l'état "enabled"](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/CRESH_API_CLIENT_IDENTITY_PROVIDER_ENABLED_2020-04-14T19-47-33.445Z.pngs?inline=false)

  * Configuration d'un  `grant type` et d'au moins un `scope` (ci-dessous j'ai ajouté le scope `scim` et le scope `roles`) :
![config `grant type` et d'au moins un `scope ](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/CRESH_API_CLIENT_CONFIG-GRANT-TYPES_AND_SCOPES.png?inline=false)

ci-dessus, on voit, et c'est important :
* que j'ai configuré 2 _**scopes**_, au sens de gravitee, pour ce client creshapi : `scim` et `roles`
* et en conséquance, la requête à envoyer pour obtenir un `token` est à modifier en utilisant le paramètre `&scope=${VALEUR_DE_MON_SCOPE}` (`${VALEUR_DE_MON_SCOPE}` devra donc ici être d'une des deux valeurs `scim` ou `roles` ) :

```bash
jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=scim&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "access_token" : "eyJraWQiOiJkZWZhdWx0LWdyYXZpdGVlLUFNLWtleSIsImFsZyI6IkhTMjU2In0.eyJzdWIiOiJjcmVzaEFQSV9DbGllbnRJRF9KQkwiLCJhdWQiOiJjcmVzaEFQSV9DbGllbnRJRF9KQkwiLCJkb21haW4iOiJjcmVzaGRvbWFpbmUiLCJzY29wZSI6InNjaW0iLCJpc3MiOiJodHRwOi8vYW0uZ3Jhdml0ZWUuaW8vYW0vY3Jlc2hkb21haW5lL29pZGMiLCJleHAiOjE1ODY5MDAzMTgsImlhdCI6MTU4Njg5MzExOCwianRpIjoiM3o4TDdFeDVfM2VCVEhsY1VqdW5SeGQ3ckM4In0.mP2IX4KrUSDggjDCHqOMFtIIRa4NyE3fqpw5jXWGLbk",
  "token_type" : "bearer",
  "expires_in" : 7199,
  "scope" : "scim"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=read&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "error" : "invalid_scope",
  "error_description" : "Invalid scope(s): read"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$ curl -X POST   "http://${GRAVITEE_HOST}/am/${CRESH_API_SECURTY_DOMAIN}/oauth/token?grant_type=client_credentials&scope=roles&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}"
{
  "access_token" : "eyJraWQiOiJkZWZhdWx0LWdyYXZpdGVlLUFNLWtleSIsImFsZyI6IkhTMjU2In0.eyJzdWIiOiJjcmVzaEFQSV9DbGllbnRJRF9KQkwiLCJhdWQiOiJjcmVzaEFQSV9DbGllbnRJRF9KQkwiLCJkb21haW4iOiJjcmVzaGRvbWFpbmUiLCJzY29wZSI6InJvbGVzIiwiaXNzIjoiaHR0cDovL2FtLmdyYXZpdGVlLmlvL2FtL2NyZXNoZG9tYWluZS9vaWRjIiwiZXhwIjoxNTg2OTAxNDk0LCJpYXQiOjE1ODY4OTQyOTQsImp0aSI6IkI5Yk4teC1Wb21iS3hSRUYzU1JHR09oZDhjOCJ9.Wg_kvSDNrx9CrFj4CliUVGWBoek5AaF5WaK9z9kZXmQ",
  "token_type" : "bearer",
  "expires_in" : 7199,
  "scope" : "roles"
}jbl@poste-devops-jbl-16gbram:~/vite.gravitee.io$
```

* On voit ci-dessus que le scope `read` n'existe pas : parce que la doc gravitee prend cela comme exemple, mais il n'existe pas par défaut.
* Voici donc comment créer un scope customisé (c'est ce qui permet de définir les roles de la creshAPI, via le client) :


![creer scope custom , etape 1](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_DOMAIN_SCOPES_1.png?inline=false)

![creer scope custom , etape 2](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_DOMAIN_SCOPES_2.png?inline=false)

![creer scope custom , etape 3](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_DOMAIN_SCOPES_3.png?inline=false)

* j'ai une issue très proche de la mienne (ne pas arriver à la webui du management des API, pour créer des API) : https://github.com/gravitee-io/issues/issues/1694
* j'ai la suite à : https://docs.gravitee.io/am/2.x/am_quickstart_register_app.html

_**Conclusion Premier test**_

Ok, le test que nous venons de faire consiste à configurer l'accès et la méthode d'authentification à l'API cresh, avec le protocole `OAuth2`, avec le  _default Identity Provider_.

Pour OpenID Connect avec Keycloak, il faudra créer un nouveau identity provider dans gravitee, de type `OpenID Connect (OIDC)`, et le formulaire nous demandera les infos suivantes (hostname no de ports, client Id lcient secret etc...):


![keycloak integ.](https://gitlab.com/bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio/-/raw/master/documentation/images/api-mgmt/GRAVITEE_IO_ADD_IDENTITY_PROVIDER_KEYCLOAK.png)



### Références de documents étudiés

* https://docs.gravitee.io/am/2.x/am_quickstart_register_app.html

* https://medium.com/graviteeio/how-to-secure-an-api-with-graviteeio-api-platform-437cf2dc0699

### ANNEXE données techniques Gravitee.io

<table class="tableblock frame-all grid-all spread">
<colgroup>
<col style="width: 33.3333%;">
<col style="width: 33.3333%;">
<col style="width: 33.3334%;">
</colgroup>
<thead>
<tr>
<th class="tableblock halign-left valign-top">Component</th>
<th class="tableblock halign-left valign-top">URL</th>
<th class="tableblock halign-left valign-top">Expected</th>
</tr>
</thead>
<tbody>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock">API Gateway</p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="http://localhost:8082" class="bare">http://localhost:8082</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">Must return 404 status code, with a <code>No context-path matches the request URI</code> payload.</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock">Management API</p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="http://localhost:8083/management/apis" class="bare">http://localhost:8083/management/apis</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">Must return a 200 status code, with a <code>[]</code> payload.</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock">Management UI</p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="http://localhost:8084" class="bare">http://localhost:8084</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">Must return a 200 status code, with the Gravitee.io Portal</p></td>
</tr>
</tbody>
</table>


# Kong

https://dev.to/_mertsimsek/kong-microservices-api-gateway-with-docker-12g5


### Recette rapide Kong


* ccc :

```bash
# sur l'hôte de test local, disons qu'il ait le nom d'hôte réseau [pegasusio.io]
git clone git@gitlab.com:bureau1/pulumi-workshops/poc-api-gateway/basic-api-kong.git
cd basic-api-kong

docker run -d --name king-kong -p 8000:8000 -p 8443:8443 -p 8001:8001 -p 8444:8444 cresh/king-kong:latest

```


source info pour améliorer cette recette : https://docs.konghq.com/install/docker/?_ga=2.253834817.893832238.1586863912-1018594552.1586863912&_gac=1.182739474.1586863939.EAIaIQobChMI143Q6ejn6AIVybvVCh1RfAJ9EAAYASAAEgJu2vD_BwE


```bash

$ curl -i http://pegasusio.io:8000
HTTP/1.1 404 Not Found
Date: Tue, 14 Apr 2020 11:55:11 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
X-Kong-Response-Latency: 1
Server: kong/2.0.3

{"message":"no Route matched with those values"}
```


### Suite : https://docs.konghq.com/2.0.x/getting-started/quickstart/


(Learn Kong)



et le but final ce serait ça : (case créer  un API consumer pour ensuite derrière voir si on arrive à récupéere l'ID des  API consumers dans notre REST API )


https://docs.konghq.com/2.0.x/getting-started/adding-consumers/



# dev

```bash
export OPS_HOME=~/.poc.graviteeio
export COMMIT_MESSAGE=" feature (quickstart) : "
export COMMIT_MESSAGE="$COMMIT_MESSAGE provision la plus simple"


URI_GIT=git@gitlab.com:bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio.git
git clone $URI_GIT ${OPS_HOME}
cd ${OPS_HOME}


git add --all && git commit -m "$COMMIT_MESSAGE" && git push -u origin master

atom .

```

* cycle :

```bash
docker-compose down --rmi all && docker system prune -f --all && docker system prune -f --volumes && docker-compose pull && docker-compose up -d && docker-compose logs -f | tee launch.gravitee.io.logs
```


# Questions

### De l'expiration des secrets

* Comment est gérée l'expiration des token ? automatique ? Ou changement de secret
* Quelle est la technique utilisée par `Kong` ou par `Gravitee.io`.


Encore demain 15/04/2020 des tests avant de sstatuer sur le choix entre gravitee.io et `Kong`


# ANNEXE : Référecnes GRavittee.io

* deux articles :
  * partie 1 : https://medium.com/graviteeio/fastest-way-to-manage-and-secure-your-apis-6c35a7ebab47
  * partie 2 : https://medium.com/graviteeio/how-to-secure-an-api-with-graviteeio-api-platform-437cf2dc0699
