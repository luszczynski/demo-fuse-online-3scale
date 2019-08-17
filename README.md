# Demo using Fuse Online and 3Scale

Demo showing Fuse Online and 3Scale. This guide was based on [Rodrigo Ramalho demo](https://gist.github.com/hodrigohamalho/a52dfb24383c89b4c6b1022123e195f2).

## Slides

* https://docs.google.com/presentation/d/1MNQHqjj4XZAe7WWRiByu5VTzKuSK3-s-ZBL5uP2Sp-8/edit#slide=id.g5892f11135_0_756

## Links

* 3Scale Dashboard Samples: https://rramalho-admin.3scale.net
* Apicurio: https://www.apicur.io/
* Microcks: http://microcks.github.io/
* Syndesis Extensions: https://github.com/syndesisio/syndesis-extensions

## Pre-req

### Integrately Environment

You need to create an integrately environment

### Creating database on Openshift

First, login to Openshift using a token.

![](imgs/07.png)

And then copy to clipboard your login command
![](imgs/08.png)

Now, let's create a new database in a project named `fuse-demo`:

```bash
# Create new project on Openshift
oc new-project fuse-demo

# Create a new postgresql database using a Openshift template
oc new-app --template=postgresql-persistent --param=POSTGRESQL_PASSWORD=redhat --param=POSTGRESQL_USER=redhat --param=POSTGRESQL_DATABASE=sampledb -n fuse-demo
```

When the pod is ready, run:

```bash
# Get postgresql pod name
POD_POSTGRESQL=$(oc get po | grep postgresql | awk '{print $1}')

# Create database
oc exec -it $POD_POSTGRESQL -- bash -c 'psql -U redhat -d sampledb -c "CREATE TABLE users(id serial PRIMARY KEY,name VARCHAR (50),phone VARCHAR (50),age integer);"'

# Populate the database
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Rodrigo Ramalho', '(11) 95474-8099', 30);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Thiago Araki', '(11) 95474-8099', 31);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Gustavo Luszczynski', '(11) 95474-8099', 29);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Rafael Tuelho', '(11) 95474-8099', 55);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Elvis is not dead', '(11) 95474-8099', 36);\""

# Make sure your data is saved
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"select * from users;\""
```

> If for some reason you need to reinstall the database, just run:

```bash
oc delete all -l app=postgresql-persistent -n fuse-demo
oc delete pvc postgresql -n fuse-demo
oc delete secret postgresql -n fuse-demo`
```

### Creating a Database Connection on Fuse Online

Open your tutorial page: https://tutorial-web-app-webapp.apps.latam-3a88.openshiftworkshop.com

> Update this url `https://tutorial-web-app-webapp.apps.latam-3a88.openshiftworkshop.com` according your environment

* Open Fuse Online

![](imgs/01.png)

* Click on `Connections`

![](imgs/02.png)

* Click on `Create Connection`

![](imgs/03.png)

* Then, select `Database`

![](imgs/04.png)

1. Fill the database configuration with the following values:

* url: `jdbc:postgresql://postgresql.fuse-demo:5432/sampledb`
* user: `redhat`
* password: `redhat`

![](imgs/05.png)

* Now, click on `Validate` to make sure everything is working as expected. If it is all good, click on `Next`.

![](imgs/09.png)

* The Connection Name is: `Users Database`. Then, click on `Create`

![](imgs/06.png)

Now you should see connection `Users Database` listed in the connections page.

![](imgs/10.png)

We are good to go for our API creation demo.

## Demo

### Create an API from Scratch

Back to our `Home` page, click on `Create Integration`

![](imgs/11.png)

Then select `API Provider` from the connections listed.

![](imgs/12.png)

Choose `Create from scratch`

![](imgs/13.png)

Click on `Add a data type`

![](imgs/14.png)

Give it a name like: `User`

![](imgs/15.png)

Paste the following json example and choose `REST Resource`. Then, click `Save`.

```json
{
    "id": 0,
    "name": "Rodrigo Ramalho",
    "phone": "11 95474-8099",
    "age": 30
}
```

Click `Save` again.

![](imgs/16.png)

Now, click on `Next`

![](imgs/17.png)

And give a name for our integration: `Users API`. Click on `Save and continue`

![](imgs/18.png)

#### Creating an API for `Get All Users` (GET)

Create now a flow for the GET Method that list all users:

![](imgs/19.png)

Add a step in our flow clicking on `+`:

![](imgs/20.png)

Now choose our `Users Database` connection created previously.

![](imgs/21.png)

Click on `Invoke SQL to obtain, store, update or delete data`:

![](imgs/22.png)

Fill the `SQL Statement` with: `select * from users` and then click `Next`

![](imgs/23.png)

Add a log step in our flow. Click again on the `+`:

![](imgs/24.png)

Then choose `Log`

![](imgs/25.png)

In the `Custom Text`, write `Loading users from database` and click `Done`.

![](imgs/26.png)

Now, let's add a data mapping to our flow. In the last step, click in the yellow icon and then go to `Add a data mapping step`.

![](imgs/27.png)

Expand both panel clicking on the arrows:

![](imgs/28.png)

Now, drag and drop the source fields matching with the target fields and then click on `Done`.

![](imgs/29.png)

Click now on `Save`.

![](imgs/30.png)

#### Creating API for `Create a users` (POST)

From the combobox `Operations`, choose `Create a users`:

![](imgs/31.png)

Repeat the same steps you did when `Creating an API for Get All Users (GET)`

When adding the Users Database, you need to click on `Invoke SQL to obtain, store, update or delete data` and add `INSERT INTO USERS(NAME,PHONE,AGE) VALUES(:#NAME,:#PHONE,:#AGE);` in the field `SQL statement`.

![](imgs/32.png)

Also, during the data mapping you won't need to associate the `id` field because it will be already generate by the postgres database.

![](imgs/32.png)

In the end, you should have something like:

![](imgs/34.png)

Now, click on `Save` and then on `Publish`

![](imgs/35.png)

Now, we need to wait Openshift build our container. When done, you should see `Published version 1` on the top of the page.

If you go to the `Home` page, we have 1 integration running.

![](imgs/37.png)

Our last step is to expose our integration on Openshift using `Route`s.

```bash
oc create route edge i-users-api --service=i-users-api -n fuse
```

### Testing your integration

You can check if your integration is working properly running:

```bash
curl https://$(oc get route -n fuse | grep i-users-api | awk '{print $2"/users"}')
```

Or you can try with [httpie](https://httpie.org/):

```bash
http https://$(oc get route -n fuse | grep i-users-api | awk '{print $2"/users"}')
```
