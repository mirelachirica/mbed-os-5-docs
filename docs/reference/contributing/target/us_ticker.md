### Microsecond ticker

Implementing the microsecond ticker enables Mbed OS to perform operations that require precise timing. You can use this API to shedule events, record elapsed time and perform submillisecond delays.

<span class="warnings">**Warning:** We are changing the microsecond ticker HAL API in an upcoming release of Mbed OS. You can find details on how it may affect you in the [implementing the microsecond ticker API](#implementing-the-microsecond-ticker-api) section.

#### Assumptions

##### Defined behavior

- Has a reported frequency between 250KHz and 8MHz.
- Has a counter that is at least 16 bits wide.
- The function `ticker_init` is safe to call repeatedly.
- The function `ticker_init` allows the ticker to keep counting and disables the ticker interrupt.
- Ticker frequency is nonzero, and the counter is at least 8 bits.
- The ticker rolls over at (1 << bits) and continues counting starting from 0.
- The ticker counts at the specified frequency plus or minus 10%.
- The ticker increments by 1 each tick.
- The ticker interrupt fires only when the ticker times increments to or past the value set by `ticker_set_interrupt`.
- It is safe to call `ticker_set_interrupt` repeatedly before the handler is called.
- The function `ticker_fire_interrupt` causes `ticker_irq_handler` to be called immediately from interrupt context.
- The ticker operations `ticker_read`, `ticker_clear_interrupt`, `ticker_set_interrupt` and `ticker_fire_interrupt` take less than 20us to complete.

##### Undefined behavior

- Calling any function other than `ticker_init` before the initialization of the ticker.
- Whether `ticker_irq_handler` is called a second time if the time wraps and matches the value set by `ticker_set_interrupt` again.
- Calling `ticker_set_interrupt` with a value that has more than the supported number of bits.
- Calling any function other than `us_ticker_init` after calling `us_ticker_free`.

##### Notes

Be careful around these common trouble areas when implementing this API:

- The ticker cannot drift when rescheduled repeatedly
- The ticker keeps counting when it rolls over
- The ticker interrupts fires when the compare value is set to 0 and and overflow occurs

#### Dependencies

To implement this API, the device must have a hardware counter that has a count value at least 16 bits wide and can operate between 250KHz and 8MHz.

#### Implementing the microsecond ticker API

We are working on the new HAL microsecond ticker API, which will replace current version in an upcoming release of Mbed OS. You need to implement the microsecond ticker API in both variants. First, you need to implement the current API. You can find it on master branch:

[![View code](https://www.mbed.com/embed/?type=library)](http://os.mbed.com/docs/development/mbed-os-api-doxy/us__ticker__api_8h_source.html)

To make sure your platform is ready for the upcoming changes, you need to implement the future API and submit it in a separate pull request against `feature-hal-spec-ticker` branch. You can find the API and specification for the new microsecond ticker API in the following header file:

[![View code](https://www.mbed.com/embed/?type=library)](http://os.mbed.com/docs/development/feature-hal-spec-ticker-doxy/group__hal__us__ticker.html)

To enable microsecond ticker support in Mbed OS, add the `USTICKER` label in the `device_has` option of the target's section in the `targets.json` file.

#### Testing

The Mbed OS HAL provides a set of conformance tests for the microsecond ticker. You can use these tests to validate the correctness of your implementation. To run the microsecond ticker HAL tests, use the following command:

```
mbed test -t <toolchain> -m <target> -n tests-mbed_hal-lp_us_ticker*,tests-mbed_hal-us_ticker*
```

You can read more about the test cases:

 [![View code](https://www.mbed.com/embed/?type=library)](http://os.mbed.com/docs/development/feature-hal-spec-ticker-doxy/group__hal__us__ticker.html)

 [![View code](https://www.mbed.com/embed/?type=library)](http://os.mbed.com/docs/development/feature-hal-spec-ticker-doxy/group__hal__ticker__tests.html)

 [![View code](https://www.mbed.com/embed/?type=library)](http://os.mbed.com/docs/development/feature-hal-spec-ticker-doxy/group__hal__us__ticker__tests.html)