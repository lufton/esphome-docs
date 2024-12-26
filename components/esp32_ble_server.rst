BLE Server
==========

.. seo::
    :description: Instructions for setting up Bluetooth LE GATT Server in ESPHome.
    :image: bluetooth.svg

The ``esp32_ble_server`` component in ESPHome sets up a simple BLE GATT server that exposes the device name,
manufacturer and board. This component allows other components to create their own services to expose
data and control.

.. warning::

    The BLE software stack on the ESP32 consumes a significant amount of RAM on the device.
    
    **Crashes are likely to occur** if you include too many additional components in your device's
    configuration. Memory-intensive components such as :doc:`/components/voice_assistant` and other
    audio components are most likely to cause issues.

.. code-block:: yaml

    # Example configuration

    esp32_ble_server:
      manufacturer: "Orange"
      manufacturer_data: [ 0x4C, 0, 0x23, 77, 0xF0 ]
      services:
        - service_uuid: 37000000-82c9-4adb-90cd-792b53207775
          characteristics:
            - characteristic_uuid: 37000001-82c9-4adb-90cd-792b53207775
              properties:
                - read

Configuration variables:
------------------------

- **manufacturer** (*Optional*, string): The name of the manufacturer/firmware creator. Defaults to ``ESPHome``.
- **model** (*Optional*, string): The model name of the device. Defaults to the friendly name of the ``board`` chosen
  in the :ref:`core configuration <esphome-configuration_variables>`.
- **manufacturer_data** (*Optional*, list of bytes): The manufacturer-specific data to include in the advertising
  packet. Should be a list of bytes, where the first two are the little-endian representation of the 16-bit
  manufacturer ID as assigned by the Bluetooth SIG.
- **services** (*Optional*, list of services): The list of custom services to be created. See :ref:`service-schema`.

.. _service-schema:

Service Schema
--------------

- **service_uuid** (*Required*, UUID): The UUID of the service.
- **advertise** (*Optional*, boolean): Whether service should be advertised. Defaults to ``false``.
- **handles** (*Optional*, int): The number of handles to be allocated for the service.
  Defaults to ``1 + characteristic_count x 2``.
- **instance_id** (*Optional*, int): The instance ID of the service. Defaults to ``0``.
- **characteristics** (*Required*, list): The list of characteristics this service should expose. See
  :ref:`characteristic-schema`.

.. _characteristic-schema:

Characteristic Schema
---------------------

- **id** (*Optional*, :ref:`config-id`): The ID to use for code generation and for reference by actions.
- **characteristic_uuid** (*Required*, UUID): The UUID of the characteristic.
- **properties** (*Required*, list): The list of enabled properties. At least one should be provided. Available options:
    - **read**
    - **write**
    - **notify**
    - **broadcast**
    - **indicate**
    - **write_without_response**
- **on_write** (*Optional*, :ref:`Automation <automation>`): An automation to perform when characteristic
  receives new value via **write** or **write_without_response** BLE event. See :ref:`ble_characteristic-on_write`.

BLE Characteristic Automation
-----------------------------

.. _ble_characteristic-on_write:

``on_write``
------------

This automation is triggered when the client writes to BLE characteristic.
A variable ``data`` of type ``std::vector<uint8_t>`` with written data is passed to the automation for use in lambdas.

.. code-block:: yaml

    esp32_ble_server:
      services:
        - service_uuid: 37000000-82c9-4adb-90cd-792b53207775
          characteristics:
            - characteristic_uuid: 37000001-82c9-4adb-90cd-792b53207775
              properties:
                - write
              on_write:
                - lambda: |-
                    ESP_LOGE("HUD", "Break signal: %d", data[0] >> 0 & 1);

``ble_characteristic.set_value`` Action
---------------------------------------

This action sets value in specific BLE characteristic. If characteristic supports **notify** or **indicate**
property then client can be also notified of characteristic's value change.

Example usage:

.. code-block:: yaml

    esp32_ble_server:
      services:
        - service_uuid: 37000000-82c9-4adb-90cd-792b53207775
          characteristics:
            - id: signals_char
              characteristic_uuid: 37000001-82c9-4adb-90cd-792b53207775
              properties:
                - read
                - notify

    button:
      - platform: template
        name: Break
        on_press:
          - ble_characteristic.set_value:
              id: signals_char
              value: [ 0x01 ]
              notify: true

Configuration variables:

- **id** (**Required**, :ref:`config-id`): The ID of the associated BLE characteristic.
- **value** (**Required**, Array of bytes or :ref:`lambda <config-lambda>`): The value to be written.
- **notify** (**Required**, boolean): Whether characteristic should notify clients of a value change.
  Only available if characteristic has **notify** or **indicate** property set.
  Defaults to ``false``.

See Also
--------

- :doc:`esp32_ble`
- :doc:`esp32_improv`
- :apiref:`esp32_ble/ble.h`
- :ghedit:`Edit`
