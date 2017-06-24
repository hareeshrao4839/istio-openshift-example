# Work in Progress

## Deploy all
### Deploy Products API
```sh
oc new-app mongodb -l app=mongodb --name=productsdb \
  -e MONGODB_ADMIN_PASSWORD=password  -e MONGODB_USER=app_user \
  -e MONGODB_DATABASE=store  -e MONGODB_PASSWORD=password
  
oc new-app https://github.com/i63/store-products --name=products

oc env dc products MONGO_USER=app_user MONGO_PASSWORD=password MONGO_SERVER=productsdb MONGO_PORT=27017 MONGO_DB=store \
mongo_url='mongodb://app_user:password@productsdb/store'
```
### Deploy Inventory Api
```sh
oc new-app mysql -e MYSQL_ROOT_PASSWORD=password
sleep 5
export MYSQL=$(oc get pods -l app=mysql -o jsonpath={.items[0].metadata.name})
cd ~/store-inventory
oc cp ./script.sql $MYSQL:/tmp/script.sql
oc rsh $MYSQL
mysql -h 127.0.0.1 -u root -p < /tmp/script.sql #inside container.

oc new-app https://github.com/i63/store-inventory --name=inventory
oc env dc inventory sql_db=store sql_host=mysql sql_user=root sql_password=password
```

## Cleanup
`oc label dc store  version=v1 istio=true`  
`oc delete all  -l app=products`  
`oc delete all  -l app=store`  
`oc delete all  -l app=inventory`  
`oc delete all  -l app=productsdb`  
`oc delete all  -l app=mysql`  
