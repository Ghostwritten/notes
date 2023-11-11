We all know the password importance. It’s a best practices to use complex password.
Also, I recommend you to use the different password for each services such as email, ftp, ssh, etc.,
On top of that you should change the password frequently to avoid any unnecessary hacking attempt.By default RHEL and it’s clone uses cracklib module to check password strength.
We are going to discuss, how to check the password strength using cracklib module.
If you would like to check the password score which you have created then use the pwscore package.
If you would like to create a good password, basically it should have minimum 12-15 characters length.It should be created in the following combinations like, Alphabets (Lower case & Upper case), Numbers and Special Characters.
There are many utilities are available in Linux to check a password complexity and we are going to discuss about cracklib module today.
How to install cracklib module in Linux?
The cracklib module is available in most of the distribution repository so, use the distribution official package manager to install it.
For Fedora system, use DNF Command to install cracklib.

```powershell
$ sudo dnf install cracklib
```

For Debian/Ubuntu systems, use APT-GET Command or APT Command to install libcrack2.

```powershell
$ sudo apt install libcrack2
```

For Arch Linux based systems, use Pacman Command to install cracklib.

```powershell
$ sudo pacman -S cracklib
```

For RHEL/CentOS systems, use YUM Command to install cracklib.

```powershell
$ sudo yum install cracklib
```

For openSUSE Leap system, use Zypper Command to install cracklib.

```powershell
$ sudo zypper install cracklib
```

How to use cracklib module in Linux to check Password complexity?
I have added few example in this article to make you understand better about this module.
If you use words like, person’s name or place name or common word then you will be getting an message “it is based on a dictionary word”.

```powershell
$ echo "password" | cracklib-check
password: it is based on a dictionary word
```

The default password length in Linux is Seven characters. If you give any password less than seven characters then you will be getting an message “it is WAY too short”.

```powershell
$ echo "123" | cracklib-check 
123: it is WAY too short
```

You will be getting OK When you give good password like us.

```powershell
$ echo "ME$2w!@fgty6723" | cracklib-check
ME!@fgty6723: OK
```

How to install pwscore in Linux?
The pwscore package is available in most of the distribution official repository so, use the distribution package manager to install it.
For Fedora system, use DNF Command to install libpwquality.

```powershell
$ sudo dnf install libpwquality
```

For Debian/Ubuntu systems, use APT-GET Command or APT Command to install libpwquality.

```powershell
$ sudo apt install libpwquality
```

For Arch Linux based systems, use Pacman Command to install libpwquality.

```powershell
$ sudo pacman -S libpwquality
```

For RHEL/CentOS systems, use YUM Command to install libpwquality.

```powershell
$ sudo yum install libpwquality
```

For openSUSE Leap system, use Zypper Command to install libpwquality.

```powershell
$ sudo zypper install libpwquality
```

If you are given any words like, person name or place name or common word then you will be getting a message “it is based on a dictionary word”.

```powershell
$ echo "password" | pwscore
Password quality check failed:
 The password fails the dictionary check - it is based on a dictionary word
```

The default password length in Linux is Seven characters. If you give any password less than seven characters then you will be getting an message “it is WAY too short”.

```powershell
$ echo "123" | pwscore
Password quality check failed:
 The password is shorter than 8 characters
```

You will be getting password score When you give good password like us.

```powershell
$ echo "ME!@fgty6723" | pwscore
90
```

