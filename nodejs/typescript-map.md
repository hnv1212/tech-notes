# TypeScript Map

The **_Map_** is a new data structure introduced in ES6 so it is available to JavaScript as well as TypeScript. A Map allows storing key-value pairs (i.e. entries), similar to the maps in other programming languages e.g. Java HashMap.

As Map is a collection, meaning it has a size, and an order and we can iterate over its keys and values.

## 1. Creating a Map
Use Map type and new keyword to create a map in TypeScript.
```ts
// Create Empty Map
let myMap = new Map<string, number>();
```

To create a Map with initial key-value pairs, pass the key-value pairs as an array to the Map constructor.
```ts
// Creating map with initial key-value pairs
let myMap = new Map<string, string>([
        ["key1", "value1"],
        ["key2", "value2"]
    ]);
```

## 2. Add, Retrieve, Delete Entries from Map
The common operations available in a Map are:
1. **map.set(key, value)** – adds a new entry in the Map.
2. **map.get(key)** – retrieves the value for a given key from the Map.
3. **map.has(key)** – checks if a key is present in the Map. Returns true or false.
4. **map.size** – returns the count of entries in the Map.
5. **map.delete(key)** – deletes a key-value pair using its key. If key is found and deleted, it returns true, else returns false.
6. **map.clear()** – deletes all entries from the Map.

```ts
// Map Operations
let nameAgeMapping = new Map<string, number>();

// 1. Add entries
nameAgeMapping.set("Lokesh", 37);
nameAgeMapping.set("Raj", 35);
nameAgeMapping.set("John", 40);

// 2. Get entries
let age = nameAgeMapping.get("John");

// 3. Check entry by key
nameAgeMapping.has("Lokesh");
nameAgeMapping.has("Brian");

// 4. Size of the Map
let count = nameAgeMapping.size;

// 5. Delete an entry
let isDelete = nameAgeMapping.delete("Lokesh");

// 6. Clear whole Map
nameAgeMapping.clear();
```

## 3. Iterating over Map
The Map entries **iterate in the insertion order.** A for-each loop returns an array of [key, value] pairs for each iteration.

Use `for...of` **loop to iterate over map keys, values, or entries.**
1. `map.keys()` – to iterate over map keys
2. `map.values()` – to iterate over map values
3. `map.entries()` – to iterate over map entries
4. `map` – use object destructuring to iterate over map entries

```ts
// Iterating over keys and values in a Map
let nameAgeMapping = new Map<string, number>();
 
nameAgeMapping.set("Lokesh", 37);
nameAgeMapping.set("Raj", 35);
nameAgeMapping.set("John", 40);

//1. Iterate over map keys

for (let key of nameAgeMapping.keys()) {
    console.log(key);                   //Lokesh Raj John
}

//2. Iterate over map values
for (let value of nameAgeMapping.values()) {
    console.log(value);                 //37 35 40
}

//3. Iterate over map entries
for (let entry of nameAgeMapping.entries()) {
    console.log(entry[0], entry[1]);    //"Lokesh" 37 "Raj" 35 "John" 40
}

//4. Using object destructuring
for (let [key, value] of nameAgeMapping) {
    console.log(key, value);            //"Lokesh" 37 "Raj" 35 "John" 40
}
```  
