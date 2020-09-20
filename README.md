# Ouroboros

[![Ouroboros on Crates.IO](https://img.shields.io/crates/v/ouroboros)](https://crates.io/crate/ouroboros)
[![Documentation](https://img.shields.io/badge/documentation-link-success)](https://docs.rs/ouroboros)


Easy self-referential struct generation for Rust. 
Dual licensed under MIT / Apache 2.0.

```rust
use ouroboros::self_referencing;

#[self_referencing]
struct MyStruct {
    int_data: Box<i32>,
    float_data: Box<f32>,
    #[borrows(int_data)]
    int_reference: &'this i32,
    #[borrows(mut float_data)]
    float_reference: &'this mut f32,
}

fn main() {
    let mut my_value = MyStructBuilder {
        int_data: Box::new(42),
        float_data: Box::new(3.14),
        int_reference_builder: |int_data: &i32| int_data,
        float_reference_builder: |float_data: &mut f32| float_data,
    }.build();

    // Prints 42
    println!("{:?}", my_value.use_int_data_contents(|int_data| *int_data));
    // Prints 3.14
    println!("{:?}", my_value.use_float_reference(|float_reference| **float_reference));
    // Sets the value of float_data to 84.0
    my_value.use_all_fields_mut(|fields| {
        **fields.float_reference = (**fields.int_reference as f32) * 2.0;
    });

    // We can hold on to this reference...
    let int_ref = my_value.use_int_reference(|int_ref| *int_ref);
    println!("{:?}", *int_ref);
    // As long as the struct is still alive.
    drop(my_value);
    // This will cause an error!
    // println!("{:?}", *int_ref);
}
```