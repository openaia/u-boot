.. SPDX-License-Identifier: GPL-2.0+

U-Boot Verified Boot
====================

Introduction
------------

Verified boot here means the verification of all software loaded into a
machine during the boot process to ensure that it is authorised and correct
for that machine.

Verified boot extends from the moment of system reset to as far as you wish
into the boot process. An example might be loading U-Boot from read-only
memory, then loading a signed kernel, then using the kernel's dm-verity
driver to mount a signed root filesystem.

A key point is that it is possible to field-upgrade the software on machines
which use verified boot. Since the machine will only run software that has
been correctly signed, it is safe to read software from an updatable medium.
It is also possible to add a secondary signed firmware image, in read-write
memory, so that firmware can easily be upgraded in a secure manner.


Signing
-------

Verified boot uses cryptographic algorithms to 'sign' software images.
Images are signed using a private key known only to the signer, but can
be verified using a public key. As its name suggests the public key can be
made available without risk to the verification process. The private and
public keys are mathematically related. For more information on how this
works look up "public key cryptography" and "RSA" (a particular algorithm).

The signing and verification process looks something like this::


          Signing                                      Verification
          =======                                      ============

      +--------------+                 *
      | RSA key pair |                 *             +---------------+
      | .key  .crt   |                 *             | Public key in |
      +--------------+     +------> public key ----->| trusted place |
            |              |           *             +---------------+
            |              |           *                    |
            v              |           *                    v
       +---------+         |           *             +--------------+
       |         |---------+           *             |              |
       | signer  |                     *             |    U-Boot    |
       |         |---------+           *             |  signature   |--> yes/no
       +---------+         |           *             | verification |
            ^              |           *             |              |
            |              |           *             +--------------+
            |              |           *                    ^
       +----------+        |           *                    |
       | Software |        +----> signed image -------------+
       |  image   |                    *
       +----------+                    *


The signature algorithm relies only on the public key to do its work. Using
this key it checks the signature that it finds in the image. If it verifies
then we know that the image is OK.

The public key from the signer allows us to verify and therefore trust
software from updatable memory.

It is critical that the public key be secure and cannot be tampered with.
It can be stored in read-only memory, or perhaps protected by other on-chip
crypto provided by some modern SOCs. If the public key can be changed, then
the verification is worthless.


Chaining Images
---------------

The above method works for a signer providing images to a run-time U-Boot.
It is also possible to extend this scheme to a second level, like this:

#. Master private key is used by the signer to sign a first-stage image.
#. Master public key is placed in read-only memory.
#. Secondary private key is created and used to sign second-stage images.
#. Secondary public key is placed in first stage images
#. We use the master public key to verify the first-stage image. We then
   use the secondary public key in the first-stage image to verify the second-
   state image.
#. This chaining process can go on indefinitely. It is recommended to use a
   different key at each stage, so that a compromise in one place will not
   affect the whole change.


Flattened Image Tree (FIT)
--------------------------

The FIT format is already widely used in U-Boot. It is a flattened device
tree (FDT) in a particular format, with images contained within. FITs
include hashes to verify images, so it is relatively straightforward to
add signatures as well.

The public key can be stored in U-Boot's CONFIG_OF_CONTROL device tree in
a standard place. Then when a FIT is loaded it can be verified using that
public key. Multiple keys and multiple signatures are supported.

See :doc:`signature` for more information.

.. sectionauthor:: Simon Glass <sjg@chromium.org> 1-1-13
