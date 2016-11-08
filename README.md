# Knurling

Knurling is an attempt to provide an ergonomic interface to write application level embedded drivers and libraries, and to provide a more consistent API for board maintainers to develop against. Knurling breaks development down into three main components:

## Library Development

Developers can write drivers and libraries against a set of interfaces defined as traits, abstracting the "How", and instead focusing on the "What".

```Rust
// This library is operating on an I2C Master
use knurling_traits::i2c_master::I2cMaster;

pub struct Hih613x<I: I2cMaster> {
    driver: I,
    address: u8,
}

impl<I: I2cMaster> Hih613x<I> {

    /// Trigger a conversion of temperature and humidity data
    pub fn start_conversion(&mut self) -> Result<(), ()> {
        // Trigger conversion
        match self.driver.write_bytes(self.address, &[0]) {
            Ok(2) => Ok(()),
            _ => Err(()),
        }
    }

}
```

## Board Support Package Development

Developers focusing on "bringing up" an embedded board can implement these traits in one or many ways. For example, the developer could implement I2C using core peripherals, or even as a "bit bang" implementation. This is abstracted away from the drivers and library whenever possible.

## End users

End users will combine the two above items, picking the libraries that are useful for them, along with the implementations of these APIs that match their use cases.

```Rust
/// Developed by the library - e.g. in crate "ti-modem"
struct TiModem<S: knurling_traits::Serial> {
  state: SomeMetaData,
  port: S
}

/// Developed by the board maintainer - e.g. in crate "teensy3"
struct HwBufferBackedSerial {
  // ...
}

impl knurling_traits::Serial for HwBufferBackedSerial {
  // In here, we use the hardware native buffers available to implement the serial trait
}

/// In the application code developed by the user
fn main() {
  // setup the modem
  let modem = TiModem::new(HwBufferBackedSerial::new(some, serial, config), some, modem, configs);
  // ...
}
```

# In the future

In the future, we would like to decide on a good way to "specialize" these traits, and to better integrate concepts like `futures` that could lend better "Zero Cost Abstractions".

Right now, `knurling` isn't production ready, but we are looking for bigger and more complicated examples to work on, to help refine what this effort could look like. Issues and Pull Requests are welcome!
