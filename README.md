Setup guide to Programming Rust on the MSPM0L1306

1. Introduction

The purpose of this document is to give a basic overview and guide to preparing a programming environment in Visual Studio Code for the MSPM0L1306. The final result will be a project template that can control the peripherals on the device and flash code onto the device along with debugging data on it through OpenOCD.

2. Rust Installation 

To get started with setting up a programming environment for the MSPM0L1306 in Rust, install Rust from rust-lang.org/tools/install. After installation is complete it is recommended to restart your computer to allow for command prompt to recognize it in the path directory on your computer. 

3. Required Tools Instillation 

Now that Rust is installed the next step is to install some dependencies to create and flash a Rust project using rustup and cargo install which can be used to install addons to Rust.

```
// This will allow for the basic project template to be copied from Git Hub to be further configured.
cargo install cargo-generate
//This adds the ability to target M0 devices such as the MSPM0L1306
rustup target add thumbv6m-none-eabi
//cargo embed is a tool used to flash code onto microcontrollers using the cargo embed command 
cargo install cargo-embed
``` 

4. Configuring Basic Project 

The first step of creating the basic project template is to run the following command to create a new project file with the requirements to enable the basic functionality of programming on the MSPM0L1306 in Rust.

```
//This creates and renames the project at the specified GitHub to the current directory
cargo generate --git https://github.com/WKNick/MSPM0L1306-BASIC-RUST-CONFIG --name myproject 
```

The main file that needs to be configured is Cargo.toml specifically the [package] and [dependencies] portions. The dependencies do not need anything currently but will need to be built upon to specify new versions of Crates and also to enable access to additional Crates. The package portion does need to be changed to have the correct author name, rust version, and name. It will also require a version number and repository if the project is to be a Crate instead of a project that will be flashed onto the board.
One other file does need to be slightly modified which is the launch.json file inside of the .vscode folder.
```
“"executable":"./target/thumbv6m-none-eabi/debug/,”
```
This line needs to end in the project name that was specified in the Cargo.toml file in order to use OpenOCD to debug.

5. Flashing to Device

Flashing to the device is a simple process given the config files are all configured correctly. The default main file from GitHub will just turn on the red LED on the board. Flashing code needs to be done every time changes in the code have been made for the board to receive the new code.

```
//This is how to flash code onto the board it compiles main and anything else main 
references into binary and flashes it to the board
cargo embed 
```


6. Using the MSPM0L PAC and HAL 
There are two ways to control the peripherals on the board the first being through the HAL Crate titled MSPM0L1306-HAL and the second being the PAC Crate titled mspm0l130x. To use the PAC an in-depth knowledge of the microcontroller is necessary to know how to use this an example usage of creating a function which enables the gpio peripheral is the following.
```
pub fn enable(){
    unsafe{
        let peripherals = pac::Peripherals::steal();
        let gpioa = peripherals.GPIOA;
        gpioa.rstctl.write(|w|w.bits(0xb1000003));// reset gpio info
        gpioa.pwren.write(|w|w.bits(0x26000001));// enable gpio with reset key
        }
}
```

The alternative way is to use the HAL which contains functions for the peripherals already so that a less in-depth knowledge of the devices registers are needed. The base project that was set up in previous steps contains an examples folder that has outlined how to use the Hal for some peripherals.

7. OpenOCD Debugging 

First to use OpenOCD the application must be downloaded for your device from your preferred source.

Using OpenOCD is fairly straightforward once a program is flashed onto the board since the launch.json file is configured all that needs to be done is clicking the 4th tab in Visual Studio Code called “Run and Debug.” While in this tab clicking the green go button next to run while Debug with OpenOCD is selected will start the program. This will give access to a play/pause button and arrows to step into/ over code and also allow for register values to be read and written to while the program is paused in the bottom left window.
