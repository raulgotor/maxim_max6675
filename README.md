<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Thanks again! Now go create something AMAZING! :D
***
***
***
*** To avoid retyping too much info. Do a search and replace for the following:
*** raulgotor, maxim_max6675, twitter_handle, MAXIM AS6675 Driver, Platform agnostic C driver for the MAXIM AS6675 Cold-Junction-Compensated K-Thermocouple-to-Digital Converter
-->



<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]

<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/raulgotor/maxim_max6675">
    <!img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

<h3 align="center">MAXIM AS6675 Driver</h3>

  <p align="center">
    Platform agnostic C driver for the MAXIM AS6675 Cold-Junction-Compensated K-Thermocouple-to-Digital Converter
    <br />
    <a href="https://github.com/raulgotor/maxim_max6675"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/raulgotor/maxim_max6675">View Demo</a>
    ·
    <a href="https://github.com/raulgotor/maxim_max6675/issues">Report Bug</a>
    ·
    <a href="https://github.com/raulgotor/maxim_max6675/issues">Request Feature</a>
  </p>

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
  <ol>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT
## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

Here's a blank template to get started:
**To avoid retyping too much info. Do a search and replace with your text editor for the following:**
`raulgotor`, `maxim_max6675`, `twitter_handle`, `MAXIM AS6675 Driver`, `Platform agnostic C driver for the MAXIM AS6675 Cold-Junction-Compensated K-Thermocouple-to-Digital Converter`


### Built With

* []()
* []()
* []()

-->

<!-- GETTING STARTED -->
## Getting Started

To get a local copy up and running follow these simple steps.

### Installation

1. Navigate to your project's source directory

2. Clone the repo
   ```sh
   git clone https://github.com/raulgotor/maxim_max6675.git
   ```
3. Write a transfer function (see next section)


<!-- USAGE EXAMPLES -->
## Usage

### Transfer function

A transfer function glues the driver logic with the current device specific API.
The transfer function needs to be injected to the driver, so it can access your 
device specific SPI peripheral while remaining agnostic to the platform you are
using it at.

The transfer function should match with the following prototype:

  ```c 
  /*!
   * @brief SPI Transfer function
   * 
   * Function should write the specified number of bytes in the provided buffer
   * 
   * @param[out]          p_rx_buffer         Pointer to a buffer where to read
   *                                          the data to
   * @param[in]           size                Number of bytes to read
   * 
   * @return              bool                Operation result
   */
  typedef bool (*pf_read_func_t)(uint8_t * const p_rx_buffer, size_t const size);
   
   ```

#### Transfer function and module initialization example
An example of a transfer function for Espressif's esp-idf would be:

  ```c

  static spi_device_handle_t m_max6675_spi_handle

  bool max6675_spi_xchg(uint8_t * const rx_buffer, size_t const size)
  {
          // Configure the transaction type
          spi_transaction_t const transaction = {
                          .tx_buffer = NULL,
                          .rx_buffer = rx_buffer,
                          .length = 8 * size,
                          .rxlength = 8 * size,
          };

          bool success = (NULL != rx_buffer) && (0 != size);
          esp_err_t esp_result;

          // Lock the SPI bus for our use
          if (success) {
                  esp_result = spi_device_acquire_bus(m_max6675_spi_handle,
                                                      portMAX_DELAY);

                  success = (ESP_OK == esp_result);
          }

          // Init transaction (request in this case) and unlock the bus at the end
          if (success) {
                  esp_result = spi_device_transmit(m_max6675_spi_handle,
                                                   &transaction);

                  spi_device_release_bus(m_max6675_spi_handle);
  
                  success = (ESP_OK == esp_result);
          }

          return success;
  }
  
  ```

Below an implementation example for an ESP32 MCU using esp-idf is suggested:

- Device specific SPI bus and peripheral initialization
- Transfer function definition
- Driver initialization with injection of the transfer function

1) Initialize the SPI bus and add a MAX6675 instance
      ```c
      bool spi_init(void) {
              bool success;
              spi_bus_config_t const bus_cfg = {
                              .miso_io_num = MAX6675_MISO_PIN,
                              .mosi_io_num = MAX6675_MOSI_PIN,
                              .sclk_io_num = MAX6675_CLK_PIN,
                              .max_transfer_sz = (4 * 8)
              };
      
              spi_device_interface_config_t const max6675_cfg = {
                              .mode = 0,
                              .clock_speed_hz = 2 * 1000 * 1000,
                              .spics_io_num = 4,
                              .queue_size = 3
              };
    
              esp_err_t esp_result;
    
              // Initialize ESP32's SPI bus
              esp_result = spi_bus_initialize(HSPI_HOST, &bus_cfg, 2);
    
              success = (ESP_OK == esp_result);
    
              if (success) {
                      // Make esp-idf aware of our MAX6675
                      esp_result = spi_bus_add_device(HSPI_HOST,
                                                      &max6675_cfg,
                                                      &m_max6675_spi_handle);
      
                      success = (ESP_OK == esp_result);
              }
    
              return success;
      }
      ```
2) Define the transfer function as indicated above
3) Initialize the module and inject the transfer function
   1) Declare a handle for this specific instance of MAX6675
      ```c
      max6675_handle_t handle;
      ```
   2) Pass both the handle and the transfer function to the driver's initializing function
      ```c
      max6675_error_t max6675_result = max6675_init(&handle, max6675_spi_xchg);
      ```
  
### Reading the temperature

Bellow there is an example for performing temperature readings.
1) Check whether the thermocouple cable is properly connected
2) If everything is fine, read and use the temperature value. The temperature is
   provided in centi-degrees Celsius.

```c
int16_t temperature;
bool sensor_is_connected;
bool success;
max6675_error_t result;

result = max6675_is_sensor_connected(&handle, &sensor_is_connected);

success = ((MAX6675_ERROR_SUCCESS == result) && (sensor_is_connected));

if (success) {
        result = max6675_read_temperature(&handle, &temperature);

        success = (MAX6675_ERROR_SUCCESS == result);
}

if (success) {
        printf("The temperature is %f degrees celsius\n", ((float)temperature)/100);
}
```

### Further documentation

_Please refer to the in code documentation and to the [MAXIM MAX6675 Datasheet](https://datasheets.maximintegrated.com/en/ds/MAX6675.pdf)_



<!-- ROADMAP -->
## Roadmap

See the [open issues](https://github.com/raulgotor/maxim_max6675/issues) for a list of proposed features (and known issues).



<!-- CONTRIBUTING -->
## Contributing

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.



<!-- CONTACT -->
## Contact

Raúl Gotor

Project Link: [https://github.com/raulgotor/maxim_max6675](https://github.com/raulgotor/maxim_max6675)


<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

* [Embedded C Coding Standard, 2018 Michael Barr](https://barrgroup.com/sites/default/files/barr_c_coding_standard_2018.pdf)
* [Best README template](https://github.com/othneildrew/Best-README-Template)


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/raulgotor/maxim_max6675.svg?style=for-the-badge
[contributors-url]: https://github.com/raulgotor/maxim_max6675/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/raulgotor/maxim_max6675.svg?style=for-the-badge
[forks-url]: https://github.com/raulgotor/maxim_max6675/network/members
[stars-shield]: https://img.shields.io/github/stars/raulgotor/maxim_max6675.svg?style=for-the-badge
[stars-url]: https://github.com/raulgotor/maxim_max6675/stargazers
[issues-shield]: https://img.shields.io/github/issues/raulgotor/maxim_max6675.svg?style=for-the-badge
[issues-url]: https://github.com/raulgotor/maxim_max6675/issues
[license-shield]: https://img.shields.io/github/license/raulgotor/maxim_max6675.svg?style=for-the-badge
[license-url]: https://github.com/raulgotor/maxim_max6675/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/raulgotor