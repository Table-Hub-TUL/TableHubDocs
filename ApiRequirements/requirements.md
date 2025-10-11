## Restaurant list

```java

public class RestaurantSimple {
    public int id;
    public String name;
    public List<String> cuisine;
    public Address address;
    public Location location;
    public double rating;
    public int freeTables;
}
```

## Restaurant detail

```java
public class RestaurantDetail {
    public int id;
    public String name;
    public List<String> cuisine;
    public Address address;
    public Location location;
    public double rating;
    public List<Section> sections;
}
```

## Helper types


### Section
```java
public class Section {
    public int id;
    public String name;
    public List<Table> tables;
    public SectionLayout layout;
}
```

### Table
```java
public class Table {
    public int id;
    public TableStatus status;
    public int positionX;
    public int positionY;
    public int capacity;
}

public enum TableStatus {
    UNKNOWN,
    OCCUPIED,
    AVAILABLE
}
```

### Address
```java
public class Address {
    public int streetNumber;
    public Integer apartmentNumber; // nullable
    public String city;
    public String postalCode;
    public String country;
}
```

### Location
```java
public class Location {
    public double lat;
    public double lng;
}
```

### Section layout
```java
public class SectionLayout { // all values in cm
    public int viewportWidth;
    public int viewportHeight;
    public String shape; // SVG path string, ref: https://www.w3schools.com/graphics/svg_path.asp
}
```