<!--
.. title: A Generic Approach to LLD Problems
.. slug: a-generic-approach-to-lld-problems
.. date: 2024-10-31 17:53:05 UTC+05:30
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

# Design and Requirements

Every LLD problem has a set of requirements, but the framework that I'm going to discuss is based on real-life application development.
There are 3 components in the design:

1. **Service Layer**: This layer is responsible for handling the business logic. It is the core of the application.
2. **Persistence Layer**: This layer is responsible for handling the database operations.
3. **Controller Layer**: This layer is responsible for handling the incoming requests and sending the response back to the client.

<!-- TEASER_END -->
# Intuition

We first identify the entities in the problem statements, how the entities react with each other.

1. **Entities**: Define the data model of our system.
2. **Relationships**: Defines the business logic of our system.

The business logic can vary from problem to problem, but the management data model remains the same. With this apporoach we can easily manipulate data to support the business logic.

The main properties of the entities are:

1. They have an unique ID
2. They can retrieved by their ID
3. Other classes should only store the ID of their relationships.

# Approach

In LLD problems, we don't have to worry about the Controller Layer, as it is not a part of the problem statement. We only have to focus on the Service Layer. The persistence layer can be abstracted to use in-memory hashmap.

Here is a generic implementation of the Persistence Layer:

```java
class DataStore {
    private final Map<String, BaseModel> store = new HashMap<>();

    public void add(BaseModel model) {
        this.store.put(model.getId(), model);
    }

    public <T extends BaseModel> T getById(String id, Class<T> cls) {
        if (store.containsKey(id) && cls.isInstance(store.get(id))) {
            return cls.cast(store.get(id));
        }
        return null;
    }

    public void remove(String id) {
        this.store.remove(id);
    }

    public <T extends BaseModel> Stream<T> getDataStream(Class<T> cls) {
        return this.store.values()
                .stream()
                .filter(cls::isInstance)
                .map(cls::cast);
    }
}
```
For this to work, every entity should extend the BaseModel class. Here is the BaseModel class:

```java
class BaseModel {
    private final String id;

    BaseModel() {
        this.id = UUID.randomUUID().toString();
    }

    public String getId() {
        return this.id;
    }
}
```

# Conclusion

This is a generic approach to solve LLD problems. It is not necessary to follow this approach, but it can be helpful in solving LLD problems quickly. The main idea is to separate the business logic from the data model. This approach can be extended to support more complex business logic.
