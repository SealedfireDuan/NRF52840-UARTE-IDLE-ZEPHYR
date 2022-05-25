# NRF52840-UARTE-IDLE-ZEPHYR
Why Nrf52840 with Zephyr can't set UARTE to idle?

Backgrounds:
1/ The core of the system are NRF52840 and RF-module, with 2-wire UARTE and serveral control lines (PWR, RESET etc.) connecting them, NRF is the master.
2/ The FW is composed upon Zephyr 2.7.0, with NRF BLE and LTE.
3/ In idle state, NRF power-offs RF-module but keeps the VCC input, then sets UARTE to idle state using pm_state_set(dev, state) from Zephyr API.
4/ Current after that is still several hundreds of uA, much higher than the supposed value.

Check-list for issue:
1/ Both UARTEs in NRF52 should be set "okay" and rx-pull-up should be there.
2/ CONFIG_PM and CONFIG_PM_DEVICE must be both y, to enable pm_state_set(dev, state).
3/ CONFIG_UART_INTERRUPT_DRIVEN must be y, so that UARTE can wake up the system.
4/ On the RF-module side, the TX pin is always low after power-off, triggering the RX in NRF52 UARTE and keeps it active.
5/ Also look through this commit in Zephyr: https://github.com/zephyrproject-rtos/zephyr/commit/a2e7b0ddf4b76875f503b7e29c02f56ba8248885
6/ Ensure RX INT is disabled before setting UARTE low power, and enabled after setting UARTE normal.