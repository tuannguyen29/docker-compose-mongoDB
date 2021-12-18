# Docker Compose MongoDB

Install MongoDB with Docker Compose

## Run project

```
docker-compose up -d
```

MongoDB will run with port `27018`.

You can change the port however you want. Ex: 27019,..etc..

## Stop project

Without remove the container.

```
docker-compose stop
```

Remove the container.

```
docker-compose down
```

## MongoDB credential (for database admin)

- Username: root
- Password: rootpassword

## How to connect to MongoDB

Via mongo Shell

```
mongo admin -u root -p root123456
```