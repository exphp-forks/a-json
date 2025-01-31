<div align="center">
  <!-- <img alt="AJSON" src="logo.png"> -->
  <h1>A-JSON</h1>
  <p>Read JSON values quickly - Rust JSON Parser</p>

  <a href="https://github.com/importcjj/ajson">
  <img src="https://travis-ci.org/importcjj/a-json.svg?branch=master"></a>

  <img src="https://img.shields.io/badge/crates.io-0.1.0-blue">

  <a href="https://importcjj.github.io/rust-ajson-playground/">
  <img src="https://img.shields.io/badge/goto-playground-orange">

</a>


</div>


#### change name to AJSON, see [issue](https://github.com/importcjj/a-json/issues/2)
Inspiration comes from [gjson](https://github.com/tidwall/gjson) in golang

## Installation
Add it to your `Cargo.toml` file:
```
[dependencies]
ajson = "0.1"
```
Then add it to your code:
```rust
extern crate ajson;
```

## Todo

* Add documentation
* Follow api-guidelines
* Update benchmark
* Optimize

## Enjoy

AJSON get json value with specified path, such as `project.name` or `project.version`. When the path matches, it returns immediately!

```rust
let data = r#"
{
  "project": {
    "name": "ajson",
    "maintainer": "importcjj",
    "version": 0.1,
    "rusts": ["stable", "nightly"]
  }
}
"#;

let name = ajson::get(data, "project.name");
println!("{}", name.as_str()); // ajson
```

## Path Syntax

JSON example

```json
{
    "name": {"first": "Tom", "last": "Anderson"},
    "age":37,
    "children": ["Sara","Alex","Jack"],
    "fav.movie": "Deer Hunter",
    "friends": [
        {"first": "Dale", "last": "Murphy", "age": 44, "nets": ["ig", "fb", "tw"]},
        {"first": "Roger", "last": "Craig", "age": 68, "nets": ["fb", "tw"]},
        {"first": "Jane", "last": "Murphy", "age": 47, "nets": ["ig", "tw"]}
    ]
}
```

#### basic
Below is a quick overview of the path syntax, for more complete information please check out GJSON Syntax.

A path is a series of keys separated by a dot. A key may contain special wildcard characters '*' and '?'. To access an array value use the index as the key. To get the number of elements in an array or to access a child path, use the '#' character. The dot and wildcard characters can be escaped with '\'.

```
name.last        >> "Anderson"
age              >> 37
children         >> ["Sara","Alex","Jack"]
children.#       >> 3
children.1       >> "Alex"
child*.2         >> "Jack"
c?ildren.0       >> "Sara"
fav\.movie       >> "Deer Hunter"
friends.#.first  >> ["Dale","Roger","Jane"]
friends.1.last   >> "Craig"
```

#### Escape character
Special purpose characters, such as ., *, and ? can be escaped with \.

```
fav\.movie             "Deer Hunter"
```

#### Arrays
The # character allows for digging into JSON Arrays.To get the length of an array you'll just use the # all by itself.

```
friends.#              3
friends.#.age         [44,68,47]
```

#### queries
You can also query an array for the first match by using #(...), or find all matches with #(...)#. Queries support the ==, !=, <, <=, >, >= comparison operators and the simple pattern matching % (like) and !% (not like) operators.

```
friends.#(last=="Murphy").first   >> "Dale"
friends.#(last=="Murphy")#.first  >> ["Dale","Jane"]
friends.#(age>45)#.last           >> ["Craig","Murphy"]
friends.#(first%"D*").last        >> "Murphy"
friends.#(nets.#(=="fb"))#.first  >> ["Dale","Roger"]
```

#### construct
Basically, you can use selectors to assemble whatever you want, and of course, the result is still a json ;)


```
{name.first,age,"murphys":friends.#(last="Murphy")#.first}
[name.first,age,children.0]
```

```rust
ajson::get(json, "name.[first,last]").as_array();
ajson::get(json, "name.first"); 
ajson::get(json, "name.last");
```

## Value

Value types.
```rust
enum Value {
    String(String),
    Number(Number),
    Object(String),
    Array(String),
    Boolean(bool),
    Null,
    NotExist,
}
```

Value has a number of methods that meet your different needs.

```rust
value.as_str() -> &str
value.as_u64() -> u64
value.as_i64() -> i64
value.as_f64() -> f64
value.as_bool() -> bool
value.as_array() -> Vec<Value>
value.as_map() -> HashMap<String, Value>
value.get(&str) -> Value
```


```rust
value.exists() -> bool
value.is_number() -> bool
value.is_string() -> bool
value.is_bool() -> bool
value.is_object() -> bool
value.is_array() -> bool
value.is_null() -> bool
```

Sometimes you need to check if value exists, you can use `exists()`. Notice that when used for `null` values, exists returns `true`.

```rust

let v = ajson::get(json, "path");
if v.exist() {
  println!("got it {}", value);
}
```

## get or parse?

`Parse` needs to read a complete json element, but `get` returns the result immediately, so `get` is recommended if you want to simply get a value

## io::Read

Not only string, AJSON also can parse JSON from io::Read.

```rust
use std::fs::File;

let f = file::Open("path/to/json").unwrap();
let json = ajson::parse_from_read(f);
let value = json.get("a.b");
println!("{}", value.as_str());
```

## Validate

AJSON can help you get the desired value from flawed JSON, but it's worth being more careful because of its looseness.

`be careful!!!`

Maybe need a validate function 🤔

## Performance

`$ cargo bench`

* [ajson](https://github.com/importcjj/ajson)
* [serde_json](https://github.com/serde-rs/json)
* [rust-json](https://github.com/maciejhirsz/json-rust)

```
ajson benchmark         time:   [6.7000 us 6.8023 us 6.9081 us]                             
                        change: [-1.8368% -0.4152% +1.0466%] (p = 0.58 > 0.05)
                        No change in performance detected.
Found 3 outliers among 100 measurements (3.00%)
  3 (3.00%) high mild

serde_json benchmark    time:   [48.196 us 48.543 us 48.947 us]                                  
                        change: [+2.9073% +4.4909% +6.3532%] (p = 0.00 < 0.05)
                        Performance has regressed.
Found 3 outliers among 100 measurements (3.00%)
  1 (1.00%) high mild
  2 (2.00%) high severe

json-rust benchmark     time:   [24.540 us 24.773 us 25.061 us]                                 
                        change: [+4.8288% +6.0452% +7.4633%] (p = 0.00 < 0.05)
                        Performance has regressed.
Found 5 outliers among 100 measurements (5.00%)
  4 (4.00%) high mild
  1 (1.00%) high severe
```

* MacBook Pro (13-inch, 2018, Four Thunderbolt 3 Ports)
* 2.7 GHz Intel Core i7
* 16 GB 2133 MHz LPDDR3

## problems

AJSON has just been finished, there may be some bugs and shortcomings, please feel free to issue. Also, Rust is a new language for me, and maybe ajson isn't rust enough, so I hope you have some suggestions.

## License
 MIT License.
