
# Spring Data REST
Spring Data Rest= Spring Data + REST API.
* Should use Spring Data reposiotry.

# Exposing Micro-services REST APIs
* Process
  1. finds all of the Spring Data Repositories
  2. Creates an Endpoint that matches the Entity Name
  3. Appends an S to th Entity Name in the Endpoint
  4. Expose Operations as RESTful APIs over HTTP
* Mapping
  * 200--> GET, 201-->post and put, 204-->Not content
## Configuring URL path: 
* Add annotation at class level or query method level
```
@RepositoryRestResource(path="bug")// Exposer resources at 808/bug/

public interface repository extends JpaRepository<entity,Datatype>{
    @RestResource(path="apps")//8080/bug/search/apps
    List<Ticket> findByAppId(Integer id);//query method defined. also expose 8080/bug/search/findByAppId(default)
    }
```
* All query method resources are expose under the resoure search
* use `RestResource` annotation above query method
* set `RepositoryRestResource(exported=false)` or  `@RestResource(exported=false)` : Hiding certain repo, query method, fields
* `@Override` amd `@RestResource(exported=false)` to turnoff method

## `PagingAndSortingRepository`
* `JpaRepository` (JPA related methods )extends `PagingAndSortingRepository`(Pagniation and sort records) which extends `crudRepositY`

* Two overloaded versions of `findAll()`
```
Page<T> findAll(Pageable pageable);
Iterable<T> finAll(Sor sort);
```
* Pageable: influences the page size and starting page number
  * query method: `Page`,`Size`,`Sort`
```
"page":{
      "size":1,
      "totalElements":80,
      "totalPages":30,
      "number":0
 }
 ```
* Add a sort URL parameter with anm of the property
## Implement custom API
* Standard CRUD functionality repositories usually have queires on the underlying data source
  * But with spring  data, delaring thos query methods become a multistep process.
* Custom Finder Method:  support GET only
  * for every query method declared in the repository, modify by @RestResource
  
## Leveraging complex relationships
* Separate ehach microservice into its own modual

### Relationships between entities



