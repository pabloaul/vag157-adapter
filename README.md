# VAG157 OBD2 Adapter Reverse Engineering
<img src='https://github.com/pabloaul/vag157-adapter/assets/35423980/87b95a17-67a8-46da-8298-4344bebe2513' width='512'>

## Motivation
I was given this chinese OBD2 adapter which came with a disc that contained a suspicious cracked copy of VCDS. Those adapters usually only work with the exact copy that is on the disc. Since the adapters are usually nothing special and are just back to back adapters to get from USB to OBD I was interested in seeing how exactly it works with the intention to get it running on a more recent version of VCDS.

## Hardware
- Chips on adapter:\
    ATMEGA162 - Microcontroller\
    STC12C2052AD - secondary Microcontroller (security chip)\
    ATF16V8B - Atmel PLC\
    FT232RL - USB to UART adapter\
    TJA1050 - CAN Transceiver IC\
    MCP2515 - CAN Protocol IC

- Connection chain: 
    > **computer to ATMEGA**
    > *-USB->* **FTDI FT232RL** *-UART->* **ATMEGA162**
    > 
    > **ATMEGA to car**
    > *-SPI->* MCP2515 -> TJA1050 *-CAN->* Car
    > 
    > //todo: missing is the PLC and what exactly it does here

The good thing about this adapter is that it uses genuine hardware. What is interesting is the STC chip that seems to be another chip not directly related with communicating with the car. 

Digging through some forums and a few failed attempts trying to flash the ATMEGA, the purpose of this chip became clear: 
Running unpatched VCDS software will wipe the firmware of your adapter if it finds out that it is non-genuine hardware. This STC chip prevents the device being flashed/reflashing the modded chinese firmware back.

Simply removing the STC chip¹ with a hot air gun made the ATMEGA flashable again and I was able to install my desired firmware to it.

<img src='https://github.com/pabloaul/vag157-adapter/assets/35423980/0c9ad538-671c-427a-96ef-340e97c2e6fe' width='512'>
<img src='https://github.com/pabloaul/vag157-adapter/assets/35423980/1ee34ac6-9be2-4b8f-a42b-862f79b88b1a' width='512'>

<details>
  <summary><b>Where to solder for flashing</b></summary>
  The MCP2515 chip is the easiest to solder to and some of the programming related pins go there making it a good choice for getting your flashing wires down.
<table>
<tbody>
<tr style="height: 22px;">
    <td style="height: 22px;">&nbsp;<b>ATMEGA162</b></td>
<td style="height: 22px;">&nbsp;<b>MCP2515</b></td>
</tr>
<tr style="height: 22px;">
<td style="height: 22px;">&nbsp;PB5 (MOSI)</td>
<td style="height: 22px;">&nbsp;SI 14</td>
</tr>
<tr style="height: 22px;">
<td style="height: 22px;">&nbsp;PB6 (MISO)</td>
<td style="height: 22px;">&nbsp;SO 15</td>
</tr>
<tr style="height: 22px;">
<td style="height: 22px;">&nbsp;PB7 (SCK)</td>
<td style="height: 22px;">&nbsp;SCK 13</td>
</tr>
<tr style="height: 22px;">
<td style="height: 22px;">&nbsp;RESET</td>
<td style="height: 22px;">&nbsp;none (soldered directly to ATMEGA)</td>
</tr>
<tr style="height: 22px;">
<td style="height: 22px;">&nbsp;GND</td>
<td style="height: 22px;">&nbsp;found everywhere </td>
</tr>
</tbody>
</table>   
</details>


## Software
I setup a clean Windows VM, passed the adapter through, downloaded an original copy of VCDS 22.10 (x86) and [Kolimer patches](https://sidicer.lt/files/VCDSLoader_v9.2.zip)² for it.

STC Chip¹: https://mhhauto.com/Thread-VCDS-VAG-COM-REPAIR?page=137 \
VCDS/Kolimer²: https://sidicer.lt/vcds/
