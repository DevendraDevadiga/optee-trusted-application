# Client/Host application, Trusted Application and Pseudo Trusted Applications

Here explaining the complete flow of Client/Host application, Trusted Application and Pseudo Trusted Applications using sample application.

![optee-ca-ta-pta](https://user-images.githubusercontent.com/36186082/148690120-2bc818a6-3777-4c2f-9cf0-07930d041a69.png)


## Client/Host application
The example Client and Trusted application is provided below:

```ruby
https://github.com/linaro-swg/optee_examples
```
We recommend to get the complete source code setup by following QEMU build for ARMv7 by following below link:
```ruby
https://optee.readthedocs.io/en/latest/building/devices/qemu.html
```
Enter the optee_example directory and take copy of existing code
```ruby
$ cd optee_examples
$ mkdir demo
$ cp -rf hello_world/* demo/
$ cd demo
```
The source codes under **"host"** folder is for Client/Host application.
```ruby
$ tree
.
├── Android.mk
├── CMakeLists.txt
├── host
│   ├── main.c
│   └── Makefile
├── Makefile
└── ta
    ├── Android.mk
    ├── hello_world_ta.c
    ├── include
    │   └── hello_world_ta.h
    ├── Makefile
    ├── sub.mk
    └── user_ta_header_defines.h

3 directories, 11 files
$
```
Change the name of your host application. In make files binary name used as "optee_example_hello_world".
This can be named as anything as your wish, its the name of the host binary which will be generated.

E.g SecureApp or optee_example_demo etc
For demo purpose we can use as "optee_example_demo"

Android.mk and CMakeLists.txt- When OP-TEE used along with Android OS build setup, this file will be used.
```ruby
$ vi Android.mk
```
```ruby
-LOCAL_MODULE := optee_example_hello_world
+LOCAL_MODULE := optee_example_demo
````
```ruby
$ vi CMakeLists.txt
```
```ruby
-project (optee_example_hello_world C)
+project (optee_example_demo C)
````
host/Makefile - When OP-TEE used with Linux build setup this file will be used.
```ruby

$ vi host/Makefile
```
```ruby
-BINARY = optee_example_hello_world
+BINARY = optee_example_demo
```


## Document update still in progress....



