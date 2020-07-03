
## Clone the repository and build Docker images:
``` 
git clone https://github.com/404man/famous-platform.git --recursive --jobs 3
```
```
cd famous-platform
```
``` 
docker-compose build 
```

## Apply database migrations, collect static assets:

``` 
docker-compose run --rm web python3 manage.py migrate 
```

``` 
docker-compose run --rm web python3 manage.py collectstatic --noinput 
```

## Optionally populate the database with sample data and create the admin user:
```docker-compose run --rm web python3 manage.py populatedb```

## Finally, create yourself an admin account:

``` docker-compose run --rm web python3 manage.py createsuperuser ```

# Running the services
``` docker-compose up ```