# TimekeepingReactNative


Peripheral Communication with React Native BLE (Bluetooth Low Energy)

Image by 
Don GIannatti
In this IoT era, crafting software that able to communicate with hardware is not as hard as it used to be. Especially with many development updates on wireless technology.

One of the most famous wireless technology that used a lot on daily basis is Bluetooth.

By using Bluetooth we will be able to share data across devices, connect to headphones, read data from our smartwatch, or even remotely control air conditioner, tv, smart lamp, etc.

It’s a witchcraft!

Around 2011, the newest version of Bluetooth technology is introduced, it’s called Bluetooth 4.0 or Bluetooth Low Energy.

BLE or Bluetooth Low Energy is a wireless personal area network technology designed and marketed by the Bluetooth Special Interest Group (Bluetooth SIG) aimed at novel applications in the healthcare, fitness, beacons, security, and home entertainment industries ~ Wikipedia

I’m not going to give a college lecture about BLE here, but let me explain it quickly. Basically, BLE is the newest version of Bluetooth technology that provides the same feature as its predecessor, but with considerably reduced power consumption and cost, and has better native support to a lot of operating systems.

In this post, we are going to create a simple react-native mobile app that consumes and writes data to Bluetooth peripherals.

On the react-native end, we will utilize react-native-ble-manager library.
A simulated peripheral will be used here for testing purposes. Thanks to the Bleno team for providing a powerful tool to make BLE peripheral implementation become so easy.
BLE Roles and GATT Transactions
Before we start doing any engineering stuff, let’s talk about BLE roles first. I promise I’ll make this quick.

There are two key roles in BLE communication:

The first role is Peripheral. Blood pressure meters, smartwatches, headphones are examples of peripheral. Peripheral advertises itself and waits for a central to connect to it.
The second role is Central. A smartphone is one example of central. The central device will connect to the peripheral and then communicate with it.
Now for GATT. In the BLE world …

GATT is an acronym for the Generic Attribute Profile, and it defines the way that two Bluetooth Low Energy devices transfer data back and forth using concepts called Services and Characteristics.

https://learn.adafruit.com/introduction-to-bluetooth-low-energy/gatt

GATT transactions in BLE are based on high-level, nested objects called Profiles, Services and Characteristics, which can be seen in the illustration below:


Image from https://learn.adafruit.com/introduction-to-bluetooth-low-energy/gatt
Profile doesn’t actually exist on the BLE peripheral itself, it’s simple a pre-defined collection of Services that has been compiled by either the Bluetooth SIG or by the peripheral designers.
Services are used to break data up into logic entities, and contain specific chunks of data called characteristics. A service can have one or more characteristics, and each service distinguishes itself from other services by means of a unique numeric ID called a UUID, which can be either 16-bit (for officially adopted BLE Services) or 128-bit (for custom services).
The lowest level concept in GATT transactions is the Characteristic, which encapsulates a single data point (though it may contain an array of related data, such as X/Y/Z values from a 3-axis accelerometer, etc.).
Learning Expectation
On the high level, what we are going to do is creating a simulated virtual peripheral named Foodbank, then host it on our computer. This peripheral store food information sent by the central device. We can also read the stored food there. It’s pretty much what Foodbank usually does, store food, get food.

The mobile app will show the list of available BLE peripheral devices. Later we will connect to our Foodbank peripheral from the list, to store a pizza and get one 😁

The Bluetooth capability can only be tested on an actual device. It won’t work on simulator or emulator.

Ok, let’s start!

Simulated Virtual Foodbank Peripheral in Node.js
Initialize a new Node.js project. Install 3rd party lib called Bleno on it.

FYI, we won’t be using the official Bleno package due to some issues. Instead, we will use forked Bleno maintained by @abandonware.

mkdir food-bank-peripheral
cd food-bank-peripheral
npm init
npm install @abandonware/bleno --save
touch food-bank-characteristic.js # for storing characteristic code
touch food-bank-service.js # for storing service code
touch food-bank.js # for storing the main code
Spoiler alert, even though the project setup is complete, but getting it running won’t be so easy.

We need to set up several native dependencies based on our operating system. Please take a look at https://github.com/abandonware/bleno for more details.

Foodbank Characteristic Definition
Next, let’s navigate to food-bank-characteristic.js file. Prepare theFoodBankCharacteristic function.


FoodBankCharacteristic definition
We pick valid UUID string prefixed with number 2000… as characteristic UUID

From the properties: ['read', 'write'] statement we can assume that there will be two event handlers created, read and write.

Now let’s create it. Set up the onWriteRequest event listener on the newly created characteristic. This particular listener is used for handling write requests from the central device.


Characteristic onWriteRequest definition
The central will send a write-request to the peripheral with payload is the food name. Then we will store the payload to this._storedFood.

Ok, now let’s move on to the second listener, the onReadRequest handler. This handler will do the opposite action as onWriteRequest listener. The stored food will be used as the response to the call from the central device.


Characteristic onReadRequest definition
Foodbank Service Definition
Peripheral won’t be accessible without a service, so now let’s create the FoodBankService.

What this service does is pretty much exposing the characteristic, making it consumable from the central device.


FoodBankService definition
We pick valid UUID string prefixed with number 1000… as service UUID

Foodbank main code
In this main code, what we will do is checking the availability of BLE radio. If it’s there, then let’s advertise our Food Bank peripheral, let’s make it available to every nearest central device.


Check BLE radio then advertise peripheral if it’s available
If advertising is working then let’s set up our FoodBankServices (as well as it’s FoodBankCharacteristic). Place the codes on the advertisingStart event listener.


Handle Bleno advertisingStart event
Ok, that’s it for the peripheral device.

Mobile BLE Central App using React Native
Now let’s program our react-native mobile app.

On this part, I’m not going to share the whole coding details since it’ll too much. I’ll write down only the important steps.

However, you can always see the full working sample source code on the Gitbub link shared at the end of this post.

Prepare the base code
First of all, create a new BLE central mobile app. You can use react-native-cli or expo, doesn’t matter. FYI, for expo you must eject the bare workflow, it’s required to use external native modules.

# expo
npm install -g expo-cli
expo init BLECentralApp
cd BLECentralApp
expo eject
# react native cli
npx react-native init BLECentralApp
cd BLECentralApp
Then add the react-native-ble-manager, convert-string, and buffer dependencies.

npm install --save react-native-ble-manager
npm install --save convert-string
npm install --save buffer
For the react-native-ble-manager dependency, some additional steps are required. Please see https://github.com/innoveit/react-native-ble-manager for the details.

After that, on the scene file do import the installed libraries.


Start to fill the main scene function with some hooks definition and a function called startScan() that will trigger the device scanning process.


The BleManager.scan() will list down any available BLE peripherals next to our central device. The list of the devices will be returned sequentially via the BLE emitter listener BleManagerDiscoverPeripheral.

Below, we will prepare several listener handlers for bleEmitter object, including handler for the BleManagerDiscoverPeripheral listener.


Next, on the mount lifecycle, do initialize the BLE modules, apply the BLE listeners, and also verify certain phone permissions (for android).


On the unmount, don’t forget to remove the BLE emitter listeners.

Prepare the UI
Now, let’s create a simple UI for our BLE scanner app that contains these sections below:

A header with a single button inside for triggering the BLE device scanning.
A message when no devices are available.
A FlatList showing a list of available devices.
And a footer contains two buttons for toggling the test mode, read or write.

On on the FlatList.renderItem handler, add TouchableHighlight on the list peripheral section. When the section is clicked, it’ll trigger the connectAndTestPeripheral() function.


Now for the most interesting part, let’s prepare the connectAndTestPeripheral() function.

Within this function, we’ll try to connect to the peripheral then retrieve the services and RSSI information. You can inspect the returned data if you want.


Next, let’s prepare the read and write statements. This block of codes below doesn’t necessarily need to be placed within the BleManager.retrieveServices() callback. Just make sure it’s placed within the BleManager.connect() callback.


BleManager.write() is used for writing data to the peripheral device. It accepts several mandatory parameters: peripheral id, service UUID, characteristic UUID, and the payload in bytes format.
BleManager.read() is used to read data from the peripheral device. the accepted parameters are the same as the write function, only the payload is excluded.
When BleManager.write() operation is a success, the particular request will arrive at the peripheral, to be precise on the FoodBankCharacteristic.onWriteRequest() handler that we have created earlier. It also applies to the read operation, it will arrive at onReadRequest() handler on the peripheral.

Ok, that’s it for programming stuff. Now let’s test it.

Testing BLE Communication
Start the peripheral device using node peripheral.


Run the Virtual Peripheral Device
Then run the mobile app to your device. As I said earlier, this app needs to be tested on an actual mobile device, the Bluetooth capability won’t work on a simulator or emulator.

After the app is running, tap the scan Bluetooth devices, then a list of nearest peripherals will popping up, including our Food bank peripheral.


Tap the Store Pizza to set the testing mode to write, then tap the Food Bank peripheral, then we will get the 2nd screen coming up. Our pizza is stored in the food bank peripheral. Great!

Then try to Get the stored pizza. Do the same step but now tap the Get stored food. The peripheral will respond to our read call with the stored food information, which is Pizza. So that message is coming from the peripheral, proving that the communication between our react native mobile app and the peripheral is working as intended.

Github sample source code link https://github.com/novalagung/react-native-ble-read-peripheral
