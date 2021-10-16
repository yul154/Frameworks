# Catalog
1. Using Spring’s JdbcTemplate
2. Inserting data with SimpleJdbcInsert
3. Declaring JPA repositories with Spring Data
---
## 3.1 Reading and writing data with JDBC
### 3.1.1 Adapting the domain for persistence
When persisting objects to a database it’s generally a good idea to have one field that uniquely identifies the object. 

### 3.1.2 Working with JdbcTemplate
DEFINING JDBC REPOSITORIES
1. let define operations in repository
  *  Query for all object into a collection of objects
  *  Query for a single object by its id
  *  Save an object object
```
public interface IngredientRepository {
	
	Iterable<Ingredient> findAll();
	Ingredient findOne(String id);
	Ingredient save(Ingredient ingredient);

}
```
2. Write an implementation of IngredientRepository that uses JdbcTemplate to query the database
```
@Repository
public class JdbcIngredientRepository implements IngredientRepository{
	
	private JdbcTemplate jdbc;
	
	@Autowired
	public JdbcIngredientRepository(JdbcTemplate jdbc) {
		this.jdbc = jdbc; 
   }

}
```
* it injects it with JdbcTemplate via the `@Autowired` annotated construction.
```
@Override
	public Iterable<Ingredient> findAll() {
		return jdbc.query("select id, name, type from Ingredient", 
				this::mapRowToIngredient);
	}
	
	@Override
	public Ingredient findOne(String id) {
	return jdbc.queryForObject(
	"select id, name, type from Ingredient where id=?", this::mapRowToIngredient, id);
	}
	
	private Ingredient mapRowToIngredient(ResultSet rs, int rowNum)
		    throws SQLException {
		return new Ingredient(
		rs.getString("id"),
		rs.getString("name"), Ingredient.Type.valueOf(rs.getString("type")));
		}

```
* The `query()` method accepts the SQL for the query as well as an implemetation of Spring’s RowMapper for mapping each row in the result set to an object.
* JdbcTemplate’s `update()` : used for any query that writes or updates data in the database
* * method only expects to return a single Ingredient object, so it uses the `queryForObject()`

3. Inject it into Controller and use it to provide a list of objects 
```
@Autowired
public DesignTacoController(IngredientRepository ingredientRepo) {
  this.ingredientRepo = ingredientRepo; }

```
### 3.1.4 Inserting data
Two ways to save data with JdbcTemplate include the following:
* Directly, using the update() method
* Using the SimpleJdbcInsert wrapper class

SAVING DATA WITH JDBCTEMPLATE

---
## 3.2 Persisting data with Spring Data JPA
### 3.2.2 Annotating the domain as entities

* In order to declare this as a JPA entity, Ingredient must be annotated with `@Entity`. 
* its id property must be annotated with `@Id` to designate it as the property that will uniquely identify the entity in the database.
* `@NoArgsConstructor` annotation at the class level :  JPA requires that entities have a no- arguments constructor
```
@Data
@Entity
public class Taco {
@Id @GeneratedValue(strategy=GenerationType.AUTO) private Long id;
  @NotNull
  @Size(min=5, message="Name must be at least 5 characters long")
  private String name;
  private Date createdAt;
  @ManyToMany(targetEntity=Ingredient.class)
  @Size(min=1, message="You must choose at least 1 ingredient") 
  private List<Ingredient> ingredients;
  @PrePersist
  void createdAt() {
this.createdAt = new Date(); }
}

```
* annotate the id property with `@GeneratedValue`, specifying a strategy of AUTO.
* `@ManyToMany`: A Taco can have many Ingredient objects, and an Ingredient can be a part of many Tacos.
* `@PrePersist`: is used to configure a callback for pre-persist(pre-insert) events of the entity
* `@Table`.: This specifies that Order entities should be persisted to a table in the database.

### 3.2.3 Declaring JPA repositories
Spring Data, Extend the Crud- Repository interface  which  declares about a dozen methods for CRUD operations
```
public interface TacoRepository
         extends CrudRepository<Taco, Long> {
}
```
* Spring Data JPA no need to write an implementation. When the application starts, Spring Data JPA automatically generates an implementation on the fly.
### 3.2.4 Customizing JPA repositories
Spring Data defines a sort of miniature domain-specific language (DSL) where persistence details are expressed in repository method signatures.
* Spring Data examines any methods in interface, and attempts to understand the method’s purpose in the context of the persisted object.
```
List<Order> readOrdersByDeliveryZipAndPlacedAtBetween(String deliveryZip, Date startDate, Date endDate);
```
<img width="399" alt="Screen Shot 2021-10-13 at 12 14 52 PM" src="https://user-images.githubusercontent.com/27160394/137066181-ec8c9b67-c6e1-4833-9ac7-1025053483f7.png">

JPA allows name the method anything you want and annotate it with `@Query` to explicitly specify the query to be performed when the method is called.
```
@Query("Order o where o.deliveryCity='Seattle'") List<Order> readOrdersDeliveredInSeattle();
```
