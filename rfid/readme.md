# RFID

This part has two files in Python, one for writing informations on a RFID card and one for reading these.

## Getting Started

If you want to test the codes, download this project and go in this directory then follow this instructions.

### Prerequisites

You need to install these libraries :

* RC522 library (pi-rc522)

    [Link of the Github](https://github.com/ondryaso/pi-rc522 "Named link title")

    Install using pip:

    ```
    pip install pi-rc522
    ```
* Spidev

    ```
    pip install spidev
    ```

* RPi.GPIO

    ```
    pip install RPi.GPIO
    ```

### Connecting
Connecting RC522 module to SPI is pretty easy. The following table come from [the library's Github](https://github.com/ondryaso/pi-rc522).
If you want more informations, you can go check this Github.

| Board pin name | Board pin | Physical RPi pin | RPi pin name |
|----------------|-----------|------------------|--------------|
| SDA            | 1         | 24               | GPIO8, CE0   |
| SCK            | 2         | 23               | GPIO11, SCKL |
| MOSI           | 3         | 19               | GPIO10, MOSI |
| MISO           | 4         | 21               | GPIO9, MISO  |
| IRQ            | 5         | 18               | GPIO24       |
| GND            | 6         | 6, 9, 20, 25     | Ground       |
| RST            | 7         | 22               | GPIO25       |
| 3.3V           | 8         | 1,17             | 3V3          |

### Running the codes

For the writing part, you can already launch the code by writing this following bash instruction on your Raspberry.

In this project, we're using python3.

```
python3 RFID_writeCard.py
```
See later for the explanation of the code.

For the reading part, you have to create the ".exe" which is the code from the other parts that you have to lauch when the door is closed.
You have to modify the line 69 with your relative path to the ".exe".

```
b = subprocess.call(["relative_path", str(latitude), str(longitude)], shell = False)
```

After that you can run the code without error by writing :

```
python3 RFID_readCard.py
```

## Informations about RFID

The RFID module detect magnetic badges and cards from a distance of 10cm. By passing your card before, this module can read the id of the card (UID) and all the informations on it. Each card has `its own UID`.

In this project, we used the most common card with a frequency of 13,56MHz.

### Structure

A RFID card is divided in many sectors subdivided in blocks which is also subdivided in datas.
There are : * 16 sectors
            * 4 blocks per sector (64 in total)
            * 16 datas per block (each data is a number of 8 bits)

This picture comes from this [tutorial](https://plaisirarduino.fr/rfid-avec-arduino/). Don't hesitate to check this tutorial for more informations.

To write on a card informations, you have to define the sector (in red) and the address or the block (in blue). We will see later how to do this in python. 

![Structure of the card](https://plaisirarduino.fr/arduino/wp-content/uploads/2018/07/contenue-RFID.png "RFID card structure")

`Don't modify the block 0` because sometime it can't be modified because it contains the UID. Some cards allow to modify the UID and others don't.

`Don't write the last block of each sector` because it can make the card unuseable (example : sector 1 block 7, sector 2 block 11, ...).

This structure is very important because it determines the structure of the code.

### Encryption

We can also encrypt the data that we put in the differents sectors. For this, we use two keys, a `key A` and a `key B`. Then, the person, who wants to read informations in a certain sector, has to have the correct key to decrypt them. Otherwise he can't read them.

By default these key has the value OxFF OxFF OxFF OxFF OxFF OxFF. We don't change it at this moment but it can be interesting to change them. In fact, it can prevent that anybody can read and write on the card.

### Important instructions

After the instanciation of an object RFID, we can call methods on it to make some actions.

* Wait the presence of a RFID tag

    The first instruction initialize the detection, it enables IRQ on detect and waits a tag. If there is one, we "ask" the state of the tag. The request method returns `True` if the tag is **not** present. If request is successful, we can call the next instruction.

    ```
    object.wait_for_tag()
    object.request()
    ```
* Recover the id of a RFID tag

    This method returns if there is an error or not and the identifier of the tag.

    ```
    object.anticoll()
    ```

* Select the RFID tag by its identifier

    Thanks to this instruction, we can now do whatever we want.

    ```
    object.select_tag(uid)
    ```

* Authentification

    With this instruction, we give the authentification that we use A or B (see Encryption part) and which block we want to acces with the key which is the third parameter in this instruction. The fourth is the identifier of the tag. 

    ```
    object.card_auth(object.auth_a, block_number, key_array, uid)
    ```

    write/read instruction
