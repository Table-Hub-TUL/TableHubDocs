## Restaurant list

```java
public class RestaurantSimpleDTO {
    public int id;
    public String name;
    public List<String> cuisines;
    public AddressDTO address;
    public LocationDTO location;
    public double rating;
    public List<TableAvailablity> availabilityHistogram;
}
```

## Restaurant detail

```java
public class RestaurantDetailDTO {
    public int id;
    public List<SectionDTO> sections;
}
```

## Helper types


### Section
```java
public class SectionDTO {
    public int id;
    public String name;
    public List<TableDTO> tables;
    public List<PointOfInterestDTO> pois;
    public SectionLayout layout;
}
```

### Table
```java
public class TableDTO {
    public int id;
    public TableStatus status;
    public PointDTO position;
    public int capacity;
}

public enum TableStatus {
    UNKNOWN,
    OCCUPIED,
    AVAILABLE
}
```
### Point of interest (POI)

```java
public class PointOfInterestDTO {
    public int id;
    public String description; // free text
    public PointDTO topLeft;
    public PointDTO bottomRight;
    public List<Integer> otherSectionPoiLink;
}
```


### Address
```java
public class AddressDTO {
    public int streetNumber;
    public String streetName;
    public Integer apartmentNumber; // nullable
    public String city;
    public String postalCode;
    public String country;
}
```

### Location
```java
public class LocationDTO {
    public double lat;
    public double lng;
}
```

### Layout point
```java
public class PointDTO {
    public int x;
    public int y;
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

### Table histogram entry
```java
public class TableAvailablity {
    public int tableCapacity;
    public int availableSeats;
}
```