Immutability means that once an object is created, its state cannot be changed.

How to create an immutable class

- Make the class `final` so it can't be subclassed.
- Make fields `private`.
- Make fields `final`.
- Initialize fields through the constructor.
- Don't provide setters.
- If a field contains a **mutable object**, don't expose the original reference. Use defensive copies.
    For Objects we can do this

```
final class Employee {
    private final String name;
    private final Address address;

    public Employee(String name, Address address) {
        this.name = name;
        this.address = new Address(address.getCity());
    }

    public Address getAddress() {
        return new Address(address.getCity()); //Even if user changes the state, Employee Object wont change
    }
}
```
