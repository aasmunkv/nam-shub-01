# Setup & usage of nam-shub-01 on private laptop

## 1 Setup

1. Make sure you have a hidden directory called `.ssh` in your home folder
    by typing `ls -a`. If not, make one.
2. Next, we need to create private/public rsa key pair. This is done by typing
    the following (changeusernameto your uio username).
    ```
    ssh-keygen -t rsa -b 4096 -C "[username]@uio.no"
    ```
    ```
    When terminal asks you
    ```
    ```
    Enter file in which to save the key(...):
    ```
    ```
    click [ENTER]. When terminal asks you
    ```
    ```
    Enter passphrase (empty for no passphrase):
    ```
    ```
    choose a password (minimum 5 characters). You also need to repeat
    the password. By typinglsin your terminal you will at this point ob-
    serve that you have two new items in your directory, namelyid_rsaand
    id_rsa.pub, which is your private and public key, respectively. By typing
    cat id_rsa[.pub]you may inspect these items.
    ```
3. Next, you may create a config file for your own convenience. Write
    [EDITOR NAME] configto open such a file (within.sshdirectory). Write
    the following into theconfigfile and save (changeusernameto your uio
    username).
    ```
    Host *
    ServerAliveInterval 60
    ServerAliveCountMax 1440
    ```

    ```
    Host uio
    Hostname login.math.uio.no
    user [username]
    ```
    ```
    Host nam-shub-
    Hostname nam-shub-
    ProxyJump [username]@login.math.uio.no
    user [username]
    ```
    ```
    The first three lines is to make sure that a session don’t terminate if user
    is not active. It tells the host ping a small package every 60 second. This
    is done a total of 1440 times. Thus, we are made sure that our program
    runs for at least 24 hours. The next three lines makes our life easier when
    we log in to the host calledlogin.math.uio.no. Instead of typing
    ```
    ```
    ssh [username]@login.math.uio.no
    ```
    ```
    to log in we now only need to typessh uio. NB: Do not run complex
    computations on this computer, as it only serves as a passage for logging
    intonam-shub-01. The last four lines is for easier login tonam-shub-01.
    The wordProxyJumpsimply meansuse this computer as passage to the
    nam-shub-01computer.
    ```
4. Lastly, we need to copy thepublicrsa key from our private laptop to the
    uio computer. Navigate to.sshfolder and type the following.
    ```
    ssh-copy-id -i id_rsa.pub uio
    ```
    ```
    Note that we now use the alias of the machine, created in theconfigfile.
    ```
5. Now we are ready for usage of these computers. Simply type
    ```
    ssh nam-shub-
    ```
    ```
    to log into the computer. Enter password if credentials are needed. To
    exit the computer you may typeCTRL+d.
    ```

## 2 Useful stuff

### 2.1 Moving files

You may want to move (relatively small) files back and forth between private
laptop and uio computer. To move a file from private laptop to uio computer,
type the following.

```
scp [private-location/filename] uio:[uio-location]
```

To move a file from uio computer to private laptop, type the following.

```
scp uio:[uio-location/filename] [private-location]
```
### 2.2 Module handling

When you are logged into one of UiOs many computers you need to load the
modules you are going to use when developing. To see complete list of available
modules and their versions, type the following.

```
module avail
```
To load one of the available modules, type the following.

```
module load [module name]
```
Example ofmodule namecould bePyTorch. Then it would load the default
version of PyTorch. If specific version for your code is required, this needs to
be specified. To list the loaded modules, type the following.

```
module list
```
To remove a loaded module, type the following.

```
module rm [module name]
```
If your are in need of a module which is not listed among the available modules,
install it by typing the following.

```
pip3 install --user [module name]
```
The command line argument--useris to avoid admin login for installation.
It is in this case a module for your use only and will not be available for the
outside world.

### 2.3 Handling GPUs using Python

A neat way of only using e.g. GPU 1 and 2, if one does not want (or need) to
use all four at the same time, is by adding the following three lines of code at
the start of your Python script.

```python
import os
os.environ["CUDA_DEVICE_ORDER"]="PCI_BUS_ID"
os.environ["CUDA_VISIBLE_DEVICES"]="1,2"
```
GPUs 1 and 2 will now be recognized by indices 0 and 1 for the rest of the
session, while GPUs 0 and 3 ’does not exist’. The following code illustrates this
(using both Tensorflow and PyTorch).

```python
1 import os
2 os.environ["CUDA_DEVICE_ORDER"]="PCI_BUS_ID"
3 os.environ["CUDA_VISIBLE_DEVICES"]="1,2"
4
5 import torch
6 import tensorflow as tf
7
8 avail = torch.cuda.is_available()
9 count = torch.cuda.device_count()
10 print("Available:",avail,", Count:",count)
11
12 gpus = tf.config.list_physical_devices(’GPU’)
13 for gpu in gpus:
14 print("Name:",gpu.name,", Type:",gpu.device_type)
```
Running the code WITH lines 1-3 yields the following output.

```
Available: True , Count: 4
Name: /physical_device:GPU:0 , Type: GPU
Name: /physical_device:GPU:1 , Type: GPU
Name: /physical_device:GPU:2 , Type: GPU
Name: /physical_device:GPU:3 , Type: GPU
```
Running the code WITHOUT lines 1-3 yields the following output.

```
Available: True , Count: 2
Name: /physical_device:GPU:0 , Type: GPU
Name: /physical_device:GPU:1 , Type: GPU
```
### 2.4 Jupyter notebook/lab

The following is a description on how to use Jupyter notebook/lab on UiOs
computers.

1. Open two terminals, hereby calledterm1andterm2.

2. Interm1, log intonam-shub-01by typing the following.
    ```
    ssh -Y nam-shub-
    ```
    ```
    The flag-Yenables trusted X11 forwarding.
    ```

3. Still being interm1, start Jupyter notebook by typing the following.
    ```
    jupyter notebook --no-browser --port=
    ```
    ```
    Jupyter lab may be started in similar fashion. The port chosen (=8081) to
    transfer data is somewhat arbitrarily chosen, however we need to ensure
    that we choose a port which is not used for anything else.
    ```

4. Interm2 we now want to start forwarding the ssh signal. Do this by
    typing the following.
    ```
    ssh -N -L 8081:localhost:8081 nam-shub-
    ```
    ```
    The flag-Nstates not to execute a remote command. This is useful for
    just forwarding ports (protocol version 2 only). The flag-Lspecifies that
    the given port on the local (client) host is to be forwarded to the given
    host and port on the remote side.
    ```

5. Now that you have obtained connection through given port, go back to
    term1where you can find some links which can be copied to your browser,
    to be able to view the notebook.

### 2.5 Anaconda support

Many users will be familiar with Anaconda as a Python virtual environment
and package manager. The server has Anaconda support, and you can create
your own personal environment as opposed to using the base environment on
the server. To create your own environment, first load the Anaconda module.

```
module load Anaconda
```
You then need to initialize the Anaconda environment.

```
module load Anaconda
```
We now have an active base Anaconda environment. However, to fully utilize
Anaconda, we would often like to create our own environment where we can add
packages to serve our specific purposes. This environment will be stored in your
homefolder on the UiO servers. Anaconda has a specific command for adding
the relevant executables to your.bashrcfile.


```
conda init bash
```
This will append the necessary lines. However, the.bashrcfile does not run
automatically when you login to the server. Thus you will have to manually run
these lines, by typing in
```
. .bashrc
```
After this has been completed, you can create a new Anaconda environment by
typing

```
conda create --name [name_of_environment]
```
You can then activate your environment by typing

```
conda activate [name_of_environment]
```
You can now install any relevant packages you might need in your very own
virtual Python environment. Remember that every time you login to the server,
you will have to run the commands

```
module load Anaconda
. .bashrc
conda activate [name_of_environment]
```

to load your personal conda environment. To simplify this, you can optionally
create a bash script to do this for you. The bash script will look something like
the three lines above, but with an added

```
#!/bin/sh
```
as the first line of the script, i.e. at the top. After you have created your bash
script, and called it[name_of_script].sh, make sure you make it executable
by running

```
chmod +x [name_of_script].sh
```
Now you can run your script by typing
```
. [name_of_script].sh
```
and your Anaconda environment is ready to go.


### 2.6 VSCode support

VSCode has ssh support for working in your home environment on the UiO
servers. By using VSCode, you can easily browse, edit, and generally manage
the files on your home environment.
To initialize VSCode ssh support, you start by simply opening VSCode on
your client / computer. In the bottom right of the VSCode window there should
be a green button which you can click. If you have set up your.sshfolder
according to this guide, VSCode should find your ssh aliases and prompt you
with a menu where you can select either thenam-shub-01alias or the general
uioenvironment.
If this doesn’t show up, you will need to set these manually. This guide
(which can be found athttps://code.visualstudio.com/docs/remote/ssh)
can provide more detailed information on how to go about setting up ssh support
with VSCode.

### 2.7 Process handling

To get a list of active processes on CPU, typehtop. To see status on GPUs,
typenvidia-smi.

### 2.8 Additional

If you want to repeat a command in the terminal everyN second, type the
following.

```
watch -n [N] [command]
```

