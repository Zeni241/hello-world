**22nd Nov. 2020**

1.      Incorporated BLE instead of WiFi Access Point and http Server.

2.      Due to bellow warning, now using "esp_netif.h" instead of "tcpip_adapter.h" and "esp_event.h" instead of "esp_event_loop.h" . Also a few minor changes in wifi.c code due to this change. Its working fine.

3.    **SENDING NOTIFICATION (SORT OF PUSH NOTIFICATION) TO CLIENT**

```
        esp_err_tesp_ble_gatts_send_indicate(esp_gatt_if_t  gatts_if, uint16_t conn_id, uint16_t attr_handle, uint16_t value_len, uint8_t *value, bool need_confirm)

```     
**Sample:** 

```  
        esp_ble_gatts_send_indicate(roboble_profile_tab[PROFILE_APP_IDX].gatts_if , 0, roboble_handle_table[IDX_CHAR_VAL_A], sizeof(ind-recv), ind-recv,false )

```        
        
        Send indicate or notify to GATT client. Set param need_confirm as false will send notification, otherwise indication.

        Return
        ESP_OK : success

        other : failed

        Parameters
        [in] gatts_if: GATT server access interface

        [in] conn_id: - connection id to indicate.

        [in] attr_handle: - attribute handle to indicate.

        [in] value_len: - indicate value length.

        [in] value: value to indicate.

        [in] need_confirm: - Whether a confirmation is required. false sends a GATT notification, true sends a GATT indication.
**EVENT BIT:** 

        If a bit is set to 1 in the EventBits_t variable, then the event represented by that bit has occurred. If a bit is set to 0 in the EventBits_t variable, then the event represented by that bit has not occurred.

**WIFI/BLE COEXISTANCE:** 

        I can describe something about the WIFI and BLE/BR/EDR coexistence.
        As everyone know, ESP32 only have one RF, so WIFI and Bluetooth share the RF. Thus, low layer have two method to control the RF sharing: hardware control automatically and software control.

        1. Hardware control:
        If you disable CONFIG_SW_COEXIST_ENABLE, it will use hardware control. In this mode, Bluetooth has higher priority then WiFi. It will keep the bluetooth performance, but WiFi may be effected depend on bluetooth traffic. It means while WIFI is RX a packet, if bluetooth want to TX or RX a packet, it will stop WiFi. For example, BLE is scanning and WiFi is just in connection to hear beacon. If you set BLE scan interval equal to scan window, it means BLE scan will occupy RF all the time, so WiFi connection may be lost. So to keep WiFi connection, you should set BLE scan window little than scan interval and should not set scan window too large (such scan window=0x10, scan_interval=0x80 seems good), It will give WiFI a lot chances to use RF. Although, there 's still some conflicts may happen that WiFI is TX/RX while bluetooth is TX/RX, but the ratio is decreased. It may cause WiFi TX/RX performance decrease but don't cause WiFi connection lost. In fact, the performance is not too bad. For most IoT applications, it can be adopted.
        Most of BLE have enough time gap(except BLE scan parameter or connection interval is too bad), there's a lot chance for WiFI. So BLE and WiFi can be work in hardware control mode.
        But for BR/EDR(classic BT), due to the restriction of BR/EDR, BR/EDR has no enough time gap. Each Master-To-Slave frame, there's only a little time (much little than 625us) time gap. So BR/EDR cannot work better with WiFi simultaneously in hardware control mode.

        2. Software control:
        For resolve the restrictions of hardware control mode, we develop software control mode. This control mode works in low layer to arbit the RF occupy right for WiFI or bluetooth.
        It depends on WiFi power-save and Bluetooth retransmission mechanism. So this mode is only for WiFi station mode (AP mode could not support power-save). In this mode, most applications (A2DP/SPP/GATT-based Profiles) need not to care about the coexistence implementation, not afraid WiFi connection lost and other things, just use WiFi and Bluetooth as your expect. Except some special applications, such like BLE mesh(use scan/adv) and WiFi TCP connection, to keep BLE scan performance, it may need to modify the software control parameters to make the coexistence more fit to this application scenario.
        In software control mode, there's 3 option to modify the coexist preference（Balance, prefer to WiFi and prefer to Bluetooth). Normally, Balance is a good choice. For example, in balance preference, Bluetooth A2DP can play music fluently while WiFi is running throughput test (In sheildbox, the TCP RX can reach 8Mbps).

        Above all:
        1. BLE can works under both hardware control mode and software control mode. But in hardware control mode, you should care about the BLE scan parameter is bad or BLE connection interval is too small.
        2. BR/EDR can only works in software control mode.

**REMOVE A CHARACTER FORM A STRING AT SPECIFIC POSITION:** 


        strcpy(&str[idx_to_delete], &str[idx_to_delete + 1]);
        Pretty efficient and simple. strcpy uses memmove on most implementations.

**MEMORY**

        "Component config-> Bluetooth-> Bluedorid Bluetooth stack enabled-> Bluedriod memory debug" Set to disabled
        "Component config-> Bluetooth-> Bluedorid Bluetooth stack enabled->Close the bluedroid bt stack log print" Set to enabled
        "Component config-> Bluetooth-> Bluedorid Bluetooth stack enabled-> Classic Bluetooth" Set to disabled (if you are not suing classic bluetooth)
        "Component config-> Bluetooth-> Bluedorid Bluetooth stack enabled-> Include BLE Security module (SMP)" Set to disabled (only if you do not intend to use SMP)


        If you never intend to use bluetooth in a current boot-up cycle, you can call esp_bt_mem_release(ESP_BT_MODE_BTDM) before esp_bt_controller_init or after esp_bt_controller_deinit.
        For example, if a user only uses bluetooth for setting the WiFi configuration, and does not use bluetooth in the rest of the product operation”. In such cases, after receiving the WiFi configuration, you can disable/deinit bluetooth and release its memory. Below is the sequence of APIs to be called for such scenarios:

        esp_bluedroid_disable();
        esp_bluedroid_deinit();
        esp_bt_controller_disable();
        esp_bt_controller_deinit();
        esp_bt_mem_release(ESP_BT_MODE_BTDM);

**While vs for**

        we usually use for when there is a known number of iterations, and use while constructs when the number of iterations in not known in advance. 

**WiFi default initialization**

        The initialization code as well as registering event handlers for default interfaces, such as softAP and station, are provided in two separate APIs to facilitate simple startup code for most applications:

        esp_netif_create_default_wifi_ap()

        esp_netif_create_default_wifi_sta()

        Please note that these functions return the esp_netif handle, i.e. a pointer to a network interface object allocated and configured with default settings, which as a consequence, means that:

        The created object has to be destroyed if a network de-initialization is provided by an application.

 **These default interfaces must not be created multiple times, unless the created handle is deleted using esp_netif_destroy().**

        When using Wifi in AP+STA mode, both these interfaces has to be created.

**TOUCH**

        CONFIG_ESP32_RTC_EXT_CRYST_ADDIT_CURRENT
        Additional current for external 32kHz crystal

        Found in: Component config > ESP32-specific

        Choose which additional current is used for rtc external crystal.

        With some 32kHz crystal configurations, the X32N and X32P pins may not have enough drive strength to keep the crystal oscillating during deep sleep. If this option is enabled, additional current from touchpad 9 is provided internally to drive the 32kHz crystal. 
_**if this option is enabled, deep sleep current is slightly higher (4-5uA) and the touchpad and ULP wakeup sources are not available.**_