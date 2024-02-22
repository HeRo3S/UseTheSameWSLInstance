# UseTheSameWSLInstance

Original source from [here](https://superuser.com/a/1708681)

This method seems a bit hacky, but it should work. I haven't verified it since I don't dual-boot Win10/Win11, but I have a very high level of confidence in it. It's a modification of a technique I've suggested in the past, and because of the dual-access, it has to be done twice. While it appears quite involved, realize that I'm a bit of an "over-explainer" :-).

From your question, it sounds like you may have already done part of it, but I'm going to start "from scratch" just to make sure.

Also, you don't mentioned the distribution, so I'll assume Ubuntu, since it's the default and most common.
From the Windows that has the WSL installation you want to share

First, start from the Windows version where you already have your distribution installed. I believe that this is Windows 10 for you, so I'll use that as reference.

Start by exiting WSL and heading to PowerShell.

Run the following, modifying any path or names you want:

```sh
# Modify this next line if you'd like to use a different location
$WSL_BASE = "C:\WSL"
wsl -l -v # Confirm the distribution name and adjust next line if needed
$WSL_ORIGINAL_NAME = "Ubuntu"
$WSL_NEW_NAME = "Ubuntu-Shared"

wsl --shutdown

mkdir "$WSL_BASE\instances\$WSL_NEW_NAME"
mkdir "$WSL_BASE\images"
cd "$WSL_BASE"

wsl --export "$WSL_ORIGINAL_NAME" "$WSL_BASE\images\$WSL_ORIGINAL_NAME.tar"
wsl --import "$WSL_NEW_NAME" "$WSL_BASE\instances\$WSL_NEW_NAME" "$WSL_BASE\images\$WSL_ORIGINAL_NAME.tar" --version 2
wsl --set-default $WSL_NEW_NAME
wsl ~
```

You should be in the copied WSL instance, but as your root user. We'll fix that under Windows 11 in a moment, since we just going to "throw away" this installation anyway. Exit WSL, and from PowerShell again:

`rm "$WSL_BASE\instances\$WSL_NEW_NAME"`

With that, we've copied and then removed a distribution in the shared location. This also (safely) creates the registry entries needed for it. With the instance files removed however, we can't (yet) use this distribution until we "fix it" from Win11.
From the Windows that doesn't have the WSL installation you want to share

We're going to do a similar process in Windows 11, using the tar/image file that you created in the first step. It should go without saying, but use the same $WSL_BASE that you used in Windows 11.

From PowerShell:

```sh
# Use the same WSL_BASE that you used in Win10
$WSL_BASE = "C:\WSL"
# The WSL_ORIGINAL_NAME must also match what 
# was used in Windows 10
$WSL_ORIGINAL_NAME = "Ubuntu"
$WSL_NEW_NAME = "Ubuntu-Shared"

mkdir "$WSL_BASE\instances\$WSL_NEW_NAME"
cd "$WSL_BASE"

wsl --import "$WSL_NEW_NAME" "$WSL_BASE\instances\$WSL_NEW_NAME" "$WSL_BASE\images\$WSL_ORIGINAL_NAME.tar" --version 2

wsl ~ -d $WSL_NEW_NAME
```

As before, you should be root, since WSL doesn't "remember" the default username in --imported distributions. To fix this, use the following command to create a /etc/wsl.conf setting your default user:

```sh
read -p "Your default WSL username: " DEFAULT_USER

# One-liner - Must be copied/pasted intact:
sudo sh -c "( cat << EOF
[user]
default=$DEFAULT_USER
EOF
) >> /etc/wsl.conf"

exit

Exit your WSL distribution back to PowerShell:

wsl --shutdown
wsl --set-default $WSL_NEW_NAME
wsl ~
```
You should now have a shared, default Ubuntu-Shared distribution that works properly on both Windows 10 and Windows 11.

Once you've verified that everything is working properly, make another backup of it using wsl --export, then uninstall the original distribution from Windows 10. You can do this from the Start menu by right-clicking the "Ubuntu" (or distribution name) and selecting "Uninstall", of course.

You can also remove the original image file we create in D:\WSL\images.

Let me know if you run into any issues. I've verified each command above on my own system, but didn't actually do it from two different dual-boot Windows, of course. There's a slim chance I might have missed something on the copy/paste, and if so I apologize! We can fix it if needed, but it may require doing the whole thing over again.
