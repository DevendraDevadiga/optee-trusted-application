# Client/Host application, Trusted Application and Pseudo Trusted Applications


Get more information on Quora Link below:

https://howtorunopteeonqemuarmv8v7.quora.com/?invite_code=ym8C6ev4mVZUnJMl2s3J

Refer this article for more information:

https://bit.ly/3Kh2dRT

Also provided video about OP-TEE:

https://youtube.com/playlist?list=PLrwOamjP8JAKZ3hIdH1z3hxvYSq7XoI8d


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
## Information provided below is incomplete..

There are 3 component of software:
1. OP-TEE Cleint - tee-supplicant
2. OP-TEE Linux driver: During boot it will create /dev/tee nodes. 
3. OP-TEE OS

Example send data flow works like this:
CA->OP-TEE client -> OP-TEE Linux driver ->OP-TEE OS -> TA -> PTA (optional)->  I2C driver in OP-TEE 

And similar li receive also opposite to above flow.

I can try to explain you with simple hello world example:

1. Invoke call from CA
https://github.com/linaro-swg/optee_examples/blob/master/hello_world/host/main.c#L86 
TEEC_InvokeCommand() =>

2 It will go to optee_client and call ioctl.
/dev/tee opened and using file descriptor (fd) call the ioctl INVOK command

https://github.com/OP-TEE/optee_client/blob/06db73b3f3fdb8d23eceaedbc46c49c0b45fd1e2/libteec/src/tee_client_api.c#L678

https://github.com/OP-TEE/optee_client/blob/06db73b3f3fdb8d23eceaedbc46c49c0b45fd1e2/libteec/src/tee_client_api.c#L730

ioctl(session->ctx->fd, TEE_IOC_INVOKE, &buf_data);


3. Now inside Linux OP-TEE driver
https://github.com/torvalds/linux/blob/9b59ec8d50a1f28747ceff9a4f39af5deba9540e/drivers/tee/tee_core.c#L832

tee_ioctl_invoke(ctx, uarg);

https://github.com/torvalds/linux/blob/9b59ec8d50a1f28747ceff9a4f39af5deba9540e/drivers/tee/tee_core.c#L543

https://github.com/torvalds/linux/blob/9b59ec8d50a1f28747ceff9a4f39af5deba9540e/drivers/tee/tee_core.c#L582

ctx->teedev->desc->ops->invoke_func(ctx, &arg, params);

https://github.com/torvalds/linux/blob/9b59ec8d50a1f28747ceff9a4f39af5deba9540e/drivers/tee/optee/smc_abi.c#L1094
optee_invoke_func

https://github.com/torvalds/linux/blob/baf86ac1c9ccbde281df55a4daeefadec6d2e581/drivers/tee/optee/call.c#L402

https://github.com/torvalds/linux/blob/16477cdfefdb494235a675cc80563d736991d833/drivers/tee/optee/optee_private.h#L124

* @do_call_with_arg: enters OP-TEE in secure world

4. Now it enters OP-TEE. Here I missed some code flow. Now in OP-TEE this is the code it executes:

https://github.com/OP-TEE/optee_os/blob/203ee23d005b2cec2f21b5de334c5a246be32599/core/arch/arm/sm/sm_a32.S#L169
If SCR_NS this bit set (1) means cpu runs non secure software.
If SCR_NS this bit clear (1) means cpu runs secure software.

So they cleared that bit as now we are moving from non secure Linux to secure OP-TEE. When this bit clear CPU virtually act like secure and allows to run trusted/secure software. So it's called trusted execution environment.

You can get document from ARM to know about SCR_NS bit.

https://developer.arm.com/documentation/ddi0406/b/System-Level-Architecture/Virtual-Memory-System-Architecture--VMSA-/CP15-registers-for-a-VMSA-implementation/c1--Secure-Configuration-Register--SCR-?lang=en

(Refer proper documentation this may not)

5. Now your tee-supplicant will load the required .ta (trusted applications) which is stored in /lib/optee_armtz
Here i missed some flow. Actually open session flow we should check, that time OP-TEE will request to tee-supplicant to open required trusted applications based on the UUID.

In tee-supplicant loading will happen here

https://github.com/OP-TEE/optee_client/blob/f7ed8e3d3955e0b7a7d3ff77ab2abcfd8fb1cdb9/tee-supplicant/src/tee_supplicant.c#L303
It will copy the trusted applications to shared memory and OP-TEE will load that to secure memory TA_RAM

TEECI_LoadSecureModule(supplicant_params.ta_dir, &uuid, shm_ta.buffer, &size);
This will happen during open session call.

Now I am just explaining invoke command flow.
Now come to our invoke command flow.

6. TA invoke command will execute, based on command id it will execute function.

https://github.com/linaro-swg/optee_examples/blob/master/hello_world/ta/hello_world_ta.c#L141
From CA in invoke command api we passed the command: TA_HELLO_WORLD_CMD_INC_VALUE
So inc_value(param_types, params) will execute.
https://github.com/linaro-swg/optee_examples/blob/master/hello_world/ta/hello_world_ta.c#L97

Here they not called PTA. That is optional.

Inside inc_value() you can call your driver api.

Example if this is your driver:
https://github.com/OP-TEE/optee_os/blob/203ee23d005b2cec2f21b5de334c5a246be32599/core/drivers/imx_i2c.c#L521

Then you can call those api like this:

inc_val()
{
imx_i2c_read()
imx_i2c_write()
}

Here PTA is optional.

7. Let's consider example with PTA.

https://github.com/OP-TEE/optee_os/tree/203ee23d005b2cec2f21b5de334c5a246be32599/core/pta

PTA are under core/pta folder in OP-TEE.
PTA will built along with OP-TEE binary.

Let's take example of PTA which I implemented:
https://github.com/DevendraDevadiga/optee_os/blob/optee-secure-bootlog/core/pta/boot_log.c

Here i added command ID's:
https://github.com/DevendraDevadiga/optee_os/blob/optee-secure-bootlog/core/pta/boot_log.c#L154

From my TA application I called PTA invoke command:
https://github.com/DevendraDevadiga/optee-secure-bootlog/blob/master/tmesg/ta/tmesg_ta.c#L118


