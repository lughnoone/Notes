# TOMB

Few commands about Tomb. Full detail at https://www.dyne.org/software/tomb/

## Create a 10M Tomb
```
tomb dig -s 10 mytomb
```
## Create a Tomb key
```
tomb forge mytomb.key
```
## Lock a Tomb with a key
```
tomb lock mytomb -k mytomb.key
```
## Open a Tomb
```
tomb open mytomb -k mytomb.key
```
This mount the tomb on the machine. You can use it as any directory.
## Close a Tomb
```
tomb close
```
You must not be in the Tomb to close it.
## Kill all process using a Tomb and close it
```
tomb slam all
```
## Other information
The size of a Tomb can be increase on demand, keys can be stored in images (steganography), or even exported to a physical form like a QR code (see bury, exhume, engrave). Finally, keys or Tombs can be used on/from remote locations.
