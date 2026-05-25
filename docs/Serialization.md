# Serialization

## What is "Serialization"?

Serialization is the process of converting data structures into a linear sequence of bytes for transmission or storage. The resulting bytes can later be restored via **deserialization**.

In Java, there are two standard serialization mechanisms:
- `java.io.Serializable` - standard serialization.
- `java.io.Externalizable` - extended serialization with custom logic.

**Changes the Java Object Serialization specification handles automatically:**
- Adding new fields to the class.
- Changing fields from `static` to non-`static`.
- Changing fields from `transient` to non-`transient`.

Reversing these changes or removing fields requires additional handling depending on the required degree of backward compatibility.

---

## Serialization/Deserialization Using `Serializable`

When using `Serializable`, the serialization algorithm uses the Reflection API to:

1. Write class metadata to the stream (class name, `serialVersionUID`, field identifiers).
2. Recursively write superclass descriptions up to (but not including) `java.lang.Object`.
3. Write primitive values of serializable instance fields, starting from the highest superclass.
4. Recursively write objects referenced as fields.

Previously serialized objects are not serialized again, allowing the algorithm to handle circular references correctly.

**Deserialization:** memory is allocated for the object and fields are populated from the stream. The object's own constructor is **not called**. However, the no-argument constructor of the first non-serializable parent class **will** be called - if it doesn't exist, deserialization fails with an error.

---

## How to Modify Default Serialization Behavior?

**Option 1: Implement `Externalizable`**

Provides full control via two methods:
```java
public void writeExternal(ObjectOutput out) throws IOException;
public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
```
During deserialization, the no-argument constructor is called first, then `readExternal()` is invoked on the created object.

**Option 2: Define special private methods on a `Serializable` class**

If any of the following methods are defined, the serialization mechanism uses them instead of the default behavior:

| Method           | Purpose                                                   |
|------------------|-----------------------------------------------------------|
| `writeObject()`  | Writes the object to the stream.                          |
| `readObject()`   | Reads the object from the stream.                         |
| `writeReplace()` | Replaces the object with another instance before writing. |
| `readResolve()`  | Replaces the object with another instance after reading.  |

---

## How to Exclude Fields from Serialization?

Mark the field with the `transient` keyword - it will be skipped during serialization.

Typically used for:
- Fields holding intermediate state that can be recomputed.
- Fields referencing objects that don't need or can't support serialization.

---

## Effect of `static` and `final` on Serialization

- **`static`** - not serialized; values remain unchanged after deserialization. Technically serializable via `Externalizable`, but not recommended due to hard-to-diagnose side effects.
- **`final`** - serialized like ordinary fields with `Serializable`. Cannot be deserialized via `Externalizable` because `final` fields must be initialized in the constructor and cannot be set later in `readExternal()`. Use only standard serialization for classes with `final` fields.

---

## How to Prevent Serialization?

Override the private serialization methods to throw `NotSerializableException`:

```java
private void writeObject(ObjectOutputStream out) throws IOException {
    throw new NotSerializableException();
}

private void readObject(ObjectInputStream in) throws IOException {
    throw new NotSerializableException();
}
```

Any attempt to serialize or deserialize the object will then throw an exception.

---

## What is `serialVersionUID`?

`serialVersionUID` indicates the **version** of a serialized class. It is used during deserialization to verify that the serialized data is compatible with the current class definition.

If not explicitly declared, the JVM generates it automatically based on class metadata (fields, types, access modifiers, implemented interfaces, etc.). This is fragile - adding or removing attributes can change the generated value, causing an `InvalidClassException` at runtime.

**Always declare it explicitly:**
```java
private static final long serialVersionUID = 20161013L;
```

**When to change it:** when making **incompatible** changes to the class (e.g., removing a field). Backward-compatible changes (e.g., adding a field) do not require a version bump.

---

## Serialization Problem with Singleton

Deserializing a Singleton produces a **new, separate instance**, breaking the Singleton guarantee. Two solutions:

1. **Prevent serialization entirely** - override `writeObject()`/`readObject()` to throw `NotSerializableException` (see above).
2. **Implement `readResolve()`** - return the existing Singleton instance instead of the deserialized one:

```java
private Object readResolve() throws ObjectStreamException {
    return INSTANCE; // return the existing singleton
}
```

---

## How to Validate a Deserialized Object?

Implement the `ObjectInputValidation` interface and override `validateObject()`:

```java
public class Person implements Serializable, ObjectInputValidation {

    private int age;

    @Override
    public void validateObject() throws InvalidObjectException {
        if (age < 39 || age > 60)
            throw new InvalidObjectException("Invalid age");
    }
}
```

---

## How to Sign and Encrypt Serialized Data?

To ensure data integrity and confidentiality, add signing/encryption logic in `writeObject()`/`readObject()`, or wrap the object using the standard library classes:

- **`javax.crypto.SealedObject`** - encrypts the serialized object using a symmetric key.
- **`java.security.SignedObject`** - signs the serialized object for integrity verification.

Both classes are themselves `Serializable`, so they act as wrappers around the original object.