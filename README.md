# ubuntu-core-builds

## build information

1. Create a key
To generate a key that can then be linked to your Ubuntu Store account, run the following:
```
snapcraft create-key
```
You can pass an optional name for the key:
```
snapcraft create-key foo
```
The create-key command will ask you for a password to protect the key.

Generating the key will take some time because it's creating a 4096 bit long key that needs some entropy to complete. Moving the mouse or typing can help to speed up the process, as will installing the rng-tools package beforehand.

Now, you can list your keys with:
```
snapcraft list-keys
```
Your keys are located within ~/.snap/gnupg, and it's a good idea to keep a backup of this directory.

2. Register the key with the store
With the key created, you now need to register it with the store, effectively linking it to your account. During this step, you will be asked to select an existing key (if you have more than one) and log in with your store account credentials:
```
snapcraft register-key
```
The key is now registered with the store and you can start the actual image building.

## build

1. Create a model assertion
To build an image, it is required you have a signed model assertion.

Example
The following is an example model assertion that builds a Core 18 image for a Raspberry Pi 3 board. The JSON file is named pi3-model.json:
```json
{
  "type": "model",
  "authority-id": "<your account id>",
  "brand-id": "<your account id>",
  "series": "16",
  "model": "my-pi3",
  "architecture": "armhf",
  "base": "core18",
  "gadget": "pi=18-pi3",
  "kernel": "pi-kernel=18-pi3",
  "timestamp": "<timestamp>"
}
```
- authority-id and brand-id are your Ubuntu SSO account ID. See your account page, in the Account-Id field.
- series: the version of the assertion format, and snap namespaces, being used. Later series are backwards compatible.
- model: a free form lower-case name for your target device.
- architecture: the architecture of the device you are building the image for.
- base: the runtime environment to use.
- gadget and kernel refer to snaps already existing in the store or in the current directory.
- =18-pi3, appended to gadget and kernel, defines the channel to track. For the universal pi snaps, these denote channels for specific models.
- timestamp is a valid timestamp you need to generate using the date -Iseconds --utc command. The timestamp must be later than when the key that signs the model was registered.
- required-snaps: you can optionally add a list of required snaps that will be unremovable and will be downloaded from the store or locally if they exist in the current directory
- To alternatively build an Ubuntu Core 16 image, replace the base, gadget and kernel lines with the following:
```json
  "gadget": "pi3",
  "kernel": "pi2-kernel",
```
Note: There are additional requirements when the image targets a Brand Store. Use your Brand SSO Account's account-id in the authority-id and brand-id fields. Also add a store key whose value is your Brand store-id.

2. Sign your model assertion
Now you have to sign the model assertion with a key by piping your JSON model through the snap sign -k <key name> command. The output is the assertion document you will use to build your image.

Note: If the model is for a Brand Store, the key must be registered by the Brand SSO Account.
```
cat pi3-model.json | snap sign -k default &> pi3.model
```
This command will ask you for the password used when you created the key.

You can check the signing process has worked by ensuring the signed key has been appended to pi3.model.

3. Build the image
You can now create your image with the ubuntu-image tool. To install it, run:
```
snap install --classic ubuntu-image
```
Then, create the image:
```
sudo ubuntu-image -c beta -O pi3-test pi3.model
```
Arguments
-c: the channel snaps are downloaded from
-O: a directory for generated files
You can include specific snaps pre-installed by default in the image by using the --extra-snaps argument. For example:
```
sudo ubuntu-image -c beta --extra-snaps rocketchat-server --extra-snaps nextcloud -o pi3-test.img pi3.model
```
Note: The --extra-snaps argument takes either a snap name accessible from the store or a local path to a built snap.

4. Your image is ready
You can use a tool like dd to write the image to an SD card and boot your device (replace sdXX with the node for your storage device):
```
sudo dd if=pi3-test/pi3.img of=/dev/sdXX bs=32M
sync
```
### First boot tips
On the first boot you have to sign in with a valid Store account to make use of the device.
console-conf will download the SSH key registered with your Store account and configure it so you can log into the device via ssh <account name>@<device address> without a password.
There is no default ubuntu user on these images, but you can run sudo passwd <account name> to set a password in case you need a local console login.
