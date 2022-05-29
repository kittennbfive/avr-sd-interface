# avr-sd-interface

This repo contains the code needed for low-level access to an SD-card using an AVR. It can be used standalone or with [kittenFS32](https://github.com/kittennbfive/kittenFS32).

## Licence and disclaimer
(c) 2022 by kittennbfive
  
AGPLv3+ and NO WARRANTY!

## Important limitation
This code sets the sector size to 512 bytes as expected by kittenFS32.

## Electrical remainder
While an AVR can be used with 5V (and *must* be used with 5V for maximum speed) a SD-card will blow up if connected to this voltage. It needs 3,3V. If you don't need the extra speed for the AVR the easiest way is to supply 3,3V to the AVR and the card. Be careful!

## Files included
### spi.{c,h}
This code is for talking to the SPI interface of the AVR. It is written for ATmega328P but might be used for other AVR too. Check the datasheet!
### sd.{c,h}
This code is hardware-independant and provides the actual functionality for talking to the card.

## Error-handling
In case of an IO-error while communicating with the card, the code calls `void sd_handle_io_error(const sd_error_t err);` **that you need to provide**. `sd_error_t` is defined inside sd.h.

## API
```
sd_init_result_t sd_init(void);
void sd_read_sector(const uint32_t sector, uint8_t * const data);
void sd_write_sector(const uint32_t sector, uint8_t const * const data);
```
### sd_init
Call this before any interaction with the card but *after* `spi_init()`. If everything is fine and the card initialized correctly the code returns `SD_INIT_NO_ERROR` which will always be 0. Otherwise the code returns an error code which numerical value might change in the future, always use the constants defined in the header.  
Caution: According to the standard the SPI-frequency should be <400kHz for card initialization, but all cards i tested accepted much more. However keep this in mind if you have a card that does not initialize! My code sets the SPI-speed to the maximum=F_CPU/2.

### sd_read_sector
Read a sector of 512 bytes from the card. The 16 bit CRC sent by the card is ignored.

### sd_write_sector
Write a sector of 512 bytes to the card and wait until done (this can take up to 250ms or even 500ms while the card is busy doing its internal stuff).
