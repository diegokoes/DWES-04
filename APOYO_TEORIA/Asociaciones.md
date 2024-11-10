# 1. @OneToMany + @ManyToOne

![image](https://github.com/user-attachments/assets/b2ecf836-0ae3-43ad-9bfd-3748102c72c8) ![image](https://github.com/user-attachments/assets/89481a6a-4d66-4c38-841d-8b95c3fee2f9)


## Entidad propietaria Author: @OneToMany

```
@Entity
public class Author {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Book> books;

  ....
```

- Una instancia de Author puede estar asociada con múltiples instancias de Book.
- Un autor tiene una lista de libros (en este ejemplo un conjunto).
  
- En la clase Book, hay un campo que se llama author, que representa la relación con el autor (private Author author;).
- Al definir **mappedBy**:
    - Le estamos diciendo a JPA que Book tiene la clave foránea.
    - Tiene el valor de author, que es el nombre del campo en Book.
  
- El **Cascade** se maneja en el entity principal o padre:
    - **CascadeType.ALL:** permite que cualquier operación (persistir, eliminar, actualizar) realizada en un autor se aplique también en sus libros.
    - **orphanRemoval = true:** elimina de la base de datos cualquier libro que se elimine de la lista books del autor.


## Entidad poseída Book: @ManyTOne

```
@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    // Muchos libros pueden tener el mismo autor....
    @ManyToOne
    @JoinColumn(name = "author_id")
    private Author author;

    @Temporal(TemporalType.DATE)
    @Column(name="publication_date")
    private LocalDate publicationDate;
```
- Muchos libros (Book) pueden estar asociados con un único autor (Author).
- La anotación **@JoinColumn** se utiliza para especificar la columna en la tabla Book que se usará como clave foránea para referenciar al Author.
    - El atributo **name = "author_id"** indica que esta columna en la tabla Book se llamará author_id.

# 2. @ManyToMany

![image](https://github.com/user-attachments/assets/d24c9d35-7c21-47aa-9f80-31fee33a07dd)

## Entidad User: @ManyToMany + @JoinTable

```
@Entity
@Table(name = "users") // Nombre de la tabla en la base de datos
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // ID autoincremental en H2
    @Column(name = "user_id") // Nombre de la columna en la tabla
    private Integer id;

    @Column(name = "username", length = 50, unique = true, nullable = false) 
    private String username; // Nombre de usuario único, no nulo

    @Column(name = "password", nullable = false)
    private String password; // Contraseña no nula

    /*
     * El uso de CascadeType.ALL podría ser muy arriesgado aquí porque permite también las operaciones de eliminación en cascada (REMOVE), 
     * lo cual puede provocar que al eliminar un User, también se eliminen roles que pueden estar asociados con otros usuarios. 
     * Esto normalmente no es deseable en una relación muchos-a-muchos, ya que los roles suelen ser entidades compartidas.
     * Usando únicamente CascadeType.PERSIST y CascadeType.MERGE garantizas que:
     * PERSIST: Cuando un User se guarda por primera vez, todos los roles asignados también se guardarán automáticamente si aún no existen en la base de datos.
     * MERGE: Al actualizar un User, las entidades Role relacionadas también se actualizarán si corresponde.
     */
    

    /*
     * Estrategia de carga (fetch = FetchType.LAZY)
     * Utilizada para optimizar el rendimiento cargando las relaciones muchos-a-muchos solo cuando se necesiten. 
     * Esto es útil en relaciones complejas para evitar que se carguen automáticamente grandes conjuntos de datos.
     */

    // Relación muchos-a-muchos con Role, con tabla intermedia user_roles
    @ManyToMany(fetch = FetchType.LAZY, cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "user_roles", // Nombre de la tabla de unión
        joinColumns = @JoinColumn(name = "user_id"), // Columna en user_roles que hace referencia a User
        inverseJoinColumns = @JoinColumn(name = "role_id") // Columna en user_roles que hace referencia a Role
    )
    private Set<Rol> roles = new HashSet<>();

    ...
```

**@JoinTable:** Define la tabla de unión que se usará para establecer la relación entre User y Role.
- **name =** "user_roles": Especifica el nombre de la tabla de unión, en este caso user_roles.
- **joinColumns =** @JoinColumn(name = "user_id"): Define la columna en user_roles que hace referencia a la entidad User.
- **inverseJoinColumns =** @JoinColumn(name = "role_id"): Define la columna en user_roles que hace referencia a la entidad Role.
