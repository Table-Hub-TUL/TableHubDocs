## Restaurant list

```ts
type RestaurantSimple = {
    id: number;
    name: string;
    cuisine: string[];
    address: Address;
    location: Location;
    rating: number;
    freeTables: number;
};
```

## Restaurant detail

```ts
type RestaurantDetail = {
    id: number;
    name: string;
    cuisine: string[];
    address: Address;
    location: Location;
    rating: number;
    sections: Section[];
};
```

## Helper types


### Section
```ts
type Section = {
    id: number;
    name: string;
    tables: Table[];
};
```

### Table
```ts
type Table = {
    id: number;
    status: 'UNKNOWN' | 'OCCUPIED' | 'AVAIALABLE';
    positionX: number;
    positionY: number;
    capacity: number;
};
```

### Address
```ts
type Address = {
    streetNumber: number;
    aparmentNumber: number | null;
    city: string;
    postalCode: string;
    country: string;
};
```

### Location
```ts
type Location = {
    lat: number;
    lng: number;
};