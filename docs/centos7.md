### CentOS 7

- Install prerequisites for Homebrew. It requires kernel>=3.2 and glibc>=2.13 and CentOS 7 has kernel=3.10 and glibc=2.17. As of Feb 4, 2025, the installation also requires git>=2.7.0 and curl>=7.41.0.
```
# sudo yum install epel-release centos-release-scl
# sudo yum install rh-git227-git httpd24-curl
# echo >> ~/.bashrc
# echo 'export HOMEBREW_CURL_PATH=/opt/rh/httpd24/root/bin/curl' >> ~/.bashrc
# echo 'export HOMEBREW_GIT_PATH=/opt/rh/rh-git227/root/bin/git' >> ~/.bashrc
# echo 'source /opt/rh/httpd24/enable' >> ~/.bashrc
# echo 'source /opt/rh/rh-git227/enable' >> ~/.bashrc
# source ~/.bashrc
```

- There are 2 issues in Homebrew: First, the installation script filters out `LD_LIBRARY_PATH` added above. Second, the `brew` script hardcoded curl path to be `/usr/bin/curl` when downloading files, which leads to failure download from https urls. To make it use the updated curl, run:
```
# sudo mv /usr/bin/curl /usr/bin/curl.bak
# sudo ln -sf /opt/rh/httpd24/root/usr/bin/curl /usr/bin/curl
# sudo ln -sf /opt/rh/httpd24/root/usr/lib64/libcurl-httpd24.so.4 /usr/lib64/libcurl-httpd24.so.4
# sudo ln -sf /opt/rh/httpd24/root/usr/lib64/libnghttp2-httpd24.so.14 /usr/lib64/libnghttp2-httpd24.so.14
```

- Now, install Homebrew:
```
# /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
# source ~/.bashrc
```

- Upgrade bash, it is required when running oh-my-posh. oh-my-posh dropped support for bash 4.x around [Aug 2024](https://github.com/JanDeDobbeleer/oh-my-posh/issues/5685). **Logout and login** your user session to make shell switch to take effect.
```
# brew install bash bash-completion@2
# echo '/home/linuxbrew/.linuxbrew/bin/bash' | sudo tee -a /etc/shells
# chsh -s /home/linuxbrew/.linuxbrew/bin/bash
```
There is a [bug](https://github.com/Homebrew/homebrew-core/issues/158667) in the updated bash, that it does not wrap input commands(it does wrap output). It should be caused by the local implementation of termcap library in bash. Ubuntu passes `TERMCAP_LIB=" -ltinfo"`(terminfo library in `ncurses`) when building bash. If you cannot bear the bug, build bash yourself with `ncurses` in deps. The stock `vim` and `manpage` also complains about term features. Install `vim` and `less` brew package to resolve. The bug is fixed as of Feb 11, 2025.
Also make sure the `$SHELL` env prints the Homebrew bash path. Some terminal app like gnome-terminal set `$SHELL` to `bash` by default. Go to `preferences` -> `profiles` -> `command` -> check `run command as a login shell`.

- If you cannot login remotely, you are probably blocked by SELinux. Check with SETroubleshoot GUI or command line:
```
# sudo sealert -a /var/log/audit/audit.log
...
SELinux is preventing /usr/sbin/sshd from getattr access on the file /home/linuxbrew/.linuxbrew/Cellar/bash/5.2.37/bin/bash.
...
If you want to fix the label.
/home/linuxbrew/.linuxbrew/Cellar/bash/5.2.37/bin/bash default label should be user_home_t.
Then you can run restorecon. The access attempt may have been stopped due to insufficient permissions to access a parent directory in which case try to change the following command accordingly.
Do
# /sbin/restorecon -v /home/linuxbrew/.linuxbrew/Cellar/bash/5.2.37/bin/bash
...
```
Fix it as suggested above. Better disable SELinux, since it leads to so many issues. Disable it by editing `/etc/selinux/config`, change line to `SELINUX=disabled`. Reboot or simply disable SELinux in current session:
```
# sudo setenforce 0
```

- Install oh-my-posh and a nerd font. The font needs to be chosen in your terminal settings(e.g. `preferences` -> `profiles` -> `text` -> `custom font` using gnome terminal). Select the `non-mono`(double-width) version of the font. The icon may seems too small using the `mono`(single-width) version. See [here](https://github.com/Powerlevel9k/powerlevel9k/issues/430). If it is not possible to select the `non-mono` version, try install `font-jetbrains-mono-nerd-font` instead, or do it by setting font alias to `monospace` in fontconfig system.
```
# brew install oh-my-posh
# echo 'eval "$(oh-my-posh init bash)"' >> ~/.bashrc
# source ~/.bashrc
# brew install --cask font-dejavu-sans-mono-nerd-font
```
- Install tmux:
```
# brew install tmux
```
- Install neovim:
```
# brew install neovim
```

#### NOTE

neovim requires more recent version of glibc, which will be installed by Homebrew. So multiple versions of glibc are installed and managed. `patchelf` seems to be used to modify rpath of binary files.
It complains about missing symbols when running system `ldd`:
```
# ldd $(which nvim)
/home/linuxbrew/.linuxbrew/bin/nvim: /lib64/libc.so.6: version `GLIBC_2.33' not found (required by /home/linuxbrew/.linuxbrew/bin/nvim)
/home/linuxbrew/.linuxbrew/bin/nvim: /lib64/libc.so.6: version `GLIBC_2.34' not found (required by /home/linuxbrew/.linuxbrew/bin/nvim)
/home/linuxbrew/.linuxbrew/bin/nvim: /lib64/libc.so.6: version `GLIBC_2.32' not found (required by /home/linuxbrew/.linuxbrew/bin/nvim)
/home/linuxbrew/.linuxbrew/bin/nvim: /lib64/libm.so.6: version `GLIBC_2.29' not found (required by /home/linuxbrew/.linuxbrew/bin/nvim)
...
	linux-vdso.so.1 =>  (0x00007ffd8b7d2000)
	libluv.so.1 => /home/linuxbrew/.linuxbrew/lib/libluv.so.1 (0x00007f16ffbd1000)
	libvterm.so.0 => /home/linuxbrew/.linuxbrew/opt/libvterm/lib/libvterm.so.0 (0x00007f16ffbbc000)
	/home/linuxbrew/.linuxbrew/opt/lpeg/lib/liblpeg.so (0x00007f16ffbaa000)
	libmsgpack-c.so.2 => /home/linuxbrew/.linuxbrew/opt/msgpack/lib/libmsgpack-c.so.2 (0x00007f16ffb9f000)
	libtree-sitter.so.0.24 => /home/linuxbrew/.linuxbrew/opt/tree-sitter/lib/libtree-sitter.so.0.24 (0x00007f16ffb6c000)
	libunibilium.so.4 => /home/linuxbrew/.linuxbrew/opt/unibilium/lib/libunibilium.so.4 (0x00007f16ffb55000)
	libluajit-5.1.so.2 => /home/linuxbrew/.linuxbrew/opt/luajit/lib/libluajit-5.1.so.2 (0x00007f16ffac5000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f16ff19d000)
	libuv.so.1 => /home/linuxbrew/.linuxbrew/lib/libuv.so.1 (0x00007f16ffa75000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f16fedcf000)
	libgcc_s.so.1 => /home/linuxbrew/.linuxbrew/opt/gcc/lib/gcc/current/libgcc_s.so.1 (0x00007f16ffa44000)
	/home/linuxbrew/.linuxbrew/lib/ld.so => /lib64/ld-linux-x86-64.so.2 (0x00007f16ff9e8000)
```
But actually runs well. See Homeberw `ldd` output:
```
# $(brew --prefix glibc)/bin/ldd $(which nvim)
	linux-vdso.so.1 (0x00007ffe97579000)
	libluv.so.1 => /home/linuxbrew/.linuxbrew/lib/libluv.so.1 (0x00007f47b7654000)
	libvterm.so.0 => /home/linuxbrew/.linuxbrew/opt/libvterm/lib/libvterm.so.0 (0x00007f47b763f000)
	/home/linuxbrew/.linuxbrew/opt/lpeg/lib/liblpeg.so (0x00007f47b762c000)
	libmsgpack-c.so.2 => /home/linuxbrew/.linuxbrew/opt/msgpack/lib/libmsgpack-c.so.2 (0x00007f47b7621000)
	libtree-sitter.so.0.24 => /home/linuxbrew/.linuxbrew/opt/tree-sitter/lib/libtree-sitter.so.0.24 (0x00007f47b75ee000)
	libunibilium.so.4 => /home/linuxbrew/.linuxbrew/opt/unibilium/lib/libunibilium.so.4 (0x00007f47b75d8000)
	libluajit-5.1.so.2 => /home/linuxbrew/.linuxbrew/opt/luajit/lib/libluajit-5.1.so.2 (0x00007f47b7548000)
	libm.so.6 => /home/linuxbrew/.linuxbrew/opt/glibc/lib/libm.so.6 (0x00007f47b7464000)
	libuv.so.1 => /home/linuxbrew/.linuxbrew/lib/libuv.so.1 (0x00007f47b7428000)
	libc.so.6 => /home/linuxbrew/.linuxbrew/opt/glibc/lib/libc.so.6 (0x00007f47b7217000)
	libgcc_s.so.1 => /home/linuxbrew/.linuxbrew/opt/gcc/lib/gcc/current/libgcc_s.so.1 (0x00007f47b71e7000)
	/home/linuxbrew/.linuxbrew/lib/ld.so => /home/linuxbrew/.linuxbrew/Cellar/glibc/2.35_1/lib64/ld-linux-x86-64.so.2 (0x00007f47b7bd5000)
```
