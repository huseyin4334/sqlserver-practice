# Sharding
- Sharding is a method for distributing data across multiple machines.
- It's like partitioning a table, but each partition is on a different machine.

## How it works
- We build a hash function that maps each row to a machine.
- We can manage this in database table, code or vs.
- We decide with this algorithm when we do any operation on the data, which machine to use.
- All databases have same structure, but they have different data.

## Example
- We will build an url shortener.

``` sql

-- init.sql file

create table urls (
    id serial primary key,
    url text not null,
    URL_ID int character(5) not null
);

```

``` dockerfile

from postgres:12

copy init.sql /docker-entrypoint-initdb.d/
# this will run the init.sql file when the container starts

```

``` bash

docker build -t url-shortener .

docker run -d -p 5432:5432 --name u1 url-shortener
docker run -d -p 5433:5432 --name u2 url-shortener
docker run -d -p 5434:5432 --name u3 url-shortener

docker run -d pgadmin4

```

``` java

// We will create a hash function to distribute the data across multiple machines.

@Component
public class ShardManager {
    
    @PersistenceContext(unitName = "shard0")
    private EntityManager shard0;
    
    @PersistenceContext(unitName = "shard1")
    private EntityManager shard1;
    
    @PersistenceContext(unitName = "shard2")
    private EntityManager shard2;
    
    private static final int SHARD_COUNT = 3;

    public int getShard(String url) {
        return url.hashCode() % SHARD_COUNT;
    }
    
    public EntityManager getEntityManager(String url) {
        int shard = getShard(url);
        
        switch(shard) {
            case 0:
                return Persistence.createEntityManagerFactory("shard0").createEntityManager();
            case 1:
                return Persistence.createEntityManagerFactory("shard1").createEntityManager();
            case 2:
                return Persistence.createEntityManagerFactory("shard2").createEntityManager();
            default:
                throw new IllegalArgumentException("Invalid shard: " + shard);
        }
    }
}

```

``` java
-- We will save, read and delete the data from the database.

@Service
public class UrlService {
    
    @Autowired
    private ShardManager shardManager;
    
    public void save(String url) {
        EntityManager em = shardManager.getEntityManager(url);
        
        em.getTransaction().begin();
        
        Url u = new Url();
        u.setUrl(url);
        
        em.persist(u);
        
        em.getTransaction().commit();
    }
    
    public Url read(String url) {
        EntityManager em = shardManager.getEntityManager(url);
        
        return em.find(Url.class, url);
    }
    
    public void delete(String url) {
        EntityManager em = shardManager.getEntityManager(url);
        
        em.getTransaction().begin();
        
        Url u = em.find(Url.class, url);
        
        em.remove(u);
        
        em.getTransaction().commit();
    }
}
```

## Pros and Cons

### Pros
- It's easy to scale.
- We can scale our system easily for data or memory.
- we can secure our data easily. We can give access to only some machines.
- Optimal And Smaller index sizes.

### Cons
- Complex client management.
- Transactions are complex across multiple machines.
- Rollback is complex.
- Schema changes are hard.
- Joins are hard.
- Updates are complicated. Because we can need to update multiple machines.