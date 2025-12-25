






## Structural Design Patterns – When to Use

| Pattern    | When to Use |
|-----------|-------------|
| **Decorator** | - To modify or extend the functionality of an object without changing its base code.<br>- To implement additional functionalities of similar objects instead of reusing the same code. |
| **Facade** | - To simplify a client’s interaction with a system by hiding the underlying complex code.<br>- To interact with the methods present in a library without knowing the processing that happens in the background. |
| **Adapter** | - To enable old APIs to work with new refactored ones.<br>- To allow an object to cooperate with a class that has an incompatible interface.<br>- To reuse the existing functionality of classes. |
| **Bridge** | - To extend a class in several independent dimensions.<br>- To change the implementation at runtime.<br>- To share the implementation between objects. |
| **Composite** | - To allow the reuse of objects without worrying about their compatibility.<br>- To develop a scalable application that uses plenty of objects.<br>- To create a tree-like hierarchy of objects. |
| **Flyweight** | - To share a list of immutable strings across the application.<br>- To prevent load time as it allows caching. |
| **Proxy** | - To reduce the workload on the target object. |


## Behavioral Design Patterns – Where It Can Be Used

| Pattern | Where It Can Be Used |
|-------|----------------------|
| **Command** | - It can be used to queue and execute requests at different times.<br>- It can perform **reset** or **undo** operations.<br>- It can be used to keep a history of requests made. |
| **Iterator** | - It can be used when dealing with problems explicitly related to iteration, for designing flexible looping constructs and accessing elements from a complex collection without knowing the underlying representation.<br>- It can be used to implement a generic iterator that traverses any collection, independent of its type, efficiently. |
| **Mediator** | - It can be used to avoid tight coupling of objects in a system with many objects.<br>- It can be used to improve code readability.<br>- It can be used to make code easier to maintain. |
| **Observer** | - It can improve code management by breaking down large applications into a system of loosely-coupled objects.<br>- It can improve communication between different parts of the application.<br>- It can be used to create a **one-to-many dependency** between objects that are loosely coupled. |
| **Visitor** | - It can perform operations across a group of related but different objects without changing their classes.<br>- It can add new operations to complex object structures without modifying the objects themselves.<br>- It can add extensibility to libraries or frameworks by allowing users to plug in new behaviors easily. |
| **Interpreter** | - It can be used to build simple scripting languages or small domain-specific languages (DSLs).<br>- It can be used to define rules or grammar for processing expressions.<br>- It can interpret math expressions, search filters, or configuration scripts. |
| **Memento** | - It can implement **undo** and **redo** features in editors or applications.<br>- It can save snapshots of an object’s state without exposing its internal details.<br>- It can restore a system to a previous state when needed. |
| **State** | - It can be used when an object’s behavior needs to change based on its internal state.<br>- It can be used in scenarios like media players, game characters, or workflow steps.<br>- It can remove large **if-else** or **switch** statements related to state transitions. |
| **Strategy** | - It can select an algorithm at runtime based on the situation.<br>- It can swap out business rules or operations without changing the main logic.<br>- It can be used for payment, sorting, or route-finding strategies. |
| **Template Method** | - It can be used when multiple classes follow the same steps but need to change some of those steps.<br>- It can be used to enforce a common structure for related processes.<br>- It can be used in frameworks to let subclasses override parts of an algorithm. |
