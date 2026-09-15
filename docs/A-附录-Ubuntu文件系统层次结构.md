# 附录 A：Ubuntu 文件系统层次结构（Ubuntu Filesystem Hierarchy）

本附录整理 Ubuntu 文件系统层次结构，内容来自 `man hier` 手册页；该手册页描述的是典型 Linux 系统的目录布局。日常排查环境问题时，常常需要回答「某个库到底被安装到了系统的哪一层」「/usr/local 下的第三方软件应归入哪个目录」「编译报错中出现的头文件和库文件路径分别属于哪个层次」这类问题，本附录可以作为对照表使用。文中先给出手册页的目录树总览，再按原文顺序逐条列出每个目录的用途，并为每条英文说明附上中文译注。例如找不到 libcudnn 被装到了哪里、`/usr/local/cuda` 属于哪一层，都可以回到这里核对目录的职责与归属。目录说明部分保留了手册页的英文原文，以便直接与系统上的实际路径对照。

## 查看手册页（Viewing the Manual Page）

在终端中执行以下命令即可查看该手册页：

```bash
man hier
```

本附录的内容即来自该手册页。

## 名称（NAME）

手册页给出的名称与说明如下：

```text
hier - description of the filesystem hierarchy
```

> 译：hier —— 文件系统层次结构的说明。

## 描述（DESCRIPTION）

A typical Linux system has, among others, the following directories:

> 译：典型的 Linux 系统除其他目录外，还包含以下目录：

## 目录树总览（Directory Tree）

手册页列出的目录树总览如下（原样保留）：

```text
/
/bin
/boot
/dev
/etc
  /etc/opt
  /etc/sgml
  /etc/skel
  /etc/X11
  /etc/xml
/home
/lib
/lib<qual>
  /lib/modules
/lost+found
/media
  /media/floppy[1-9]
  /media/cdrom[1-9]
  /media/cdrecorder[1-9]
  /media/zip[1-9]
  /media/usb[1-9]
/mnt
/opt
/proc
/root
/sbin
/srv
/sys
/tmp
/usr
  /usr/X11R6
    /usr/X11R6/bin
    /usr/X11R6/lib
      /usr/X11R6/lib/X11
      /usr/X11R6/include/X11
  /usr/bin
    /usr/bin/mh
    /usr/bin/X11
  /usr/dict
  /usr/doc
  /usr/etc
  /usr/games
  /usr/include
    /usr/include/bsd
    /usr/include/X11
    /usr/include/asm
    /usr/include/linux
    /usr/include/g++
  /usr/lib
  /usr/lib<qual>
    /usr/lib/X11
    /usr/lib/gcc-lib
    /usr/lib/groff
    /usr/lib/uucp
  /usr/local
    /usr/local/bin
    /usr/local/doc
    /usr/local/etc
    /usr/local/games
    /usr/local/lib
    /usr/local/lib<qual>
    /usr/local/include
    /usr/local/info
    /usr/local/man
    /usr/local/sbin
    /usr/local/share
    /usr/local/src
  /usr/man
  /usr/sbin
  /usr/share
    /usr/share/dict
      /usr/share/dict/words
    /usr/share/doc
    /usr/share/games
    /usr/share/info
    /usr/share/locale
    /usr/share/man
        /usr/share/man/<locale>/man[1-9]
    /usr/share/misc
    /usr/share/nls
    /usr/share/sgml
      /usr/share/sgml/docbook
      /usr/share/sgml/tei
      /usr/share/sgml/html
      /usr/share/sgml/mathtml
    /usr/share/terminfo
    /usr/share/tmac
    /usr/share/xml
      /usr/share/xml/docbook
      /usr/share/xml/xhtml
      /usr/share/xml/mathml
    /usr/share/zoneinfo
  /usr/src
    /usr/src/linux
  /usr/tmp
/var
  /var/account
  /var/adm
  /var/backups
  /var/cache
    /var/cache/fonts
    /var/cache/man
    /var/cache/www
    /var/cache/<package>
    /var/catman/cat[1-9] or /var/cache/man/cat[1-9]
  /var/crash
  /var/cron
  /var/games
  /var/lib
    /var/lib/hwclock
    /var/lib/misc
    /var/lib/xdm
    /var/lib/<editor>
    /var/lib/<name>
    /var/lib/<package>
    /var/lib/<pkgtool>
  /var/local
  /var/lock
  /var/log
  /var/opt
  /var/mail
  /var/msgs
  /var/preserve
  /var/run
  /var/spool
    /var/spool/at
    /var/spool/cron
    /var/spool/lpd
      /var/spool/lpd/printer
    /var/spool/mail
    /var/spool/mqueue
    /var/spool/news
    /var/spool/rwho
    /var/spool/smail
    /var/spool/uucp
  /var/tmp
  /var/yp
```

以上为手册页列出的目录树总览。

## 目录说明（Directory Descriptions）

以下 137 条说明为手册页对每个目录的描述，按原文顺序排列，缩进层级与目录树一致。

本附录用表格重排这些说明：「目录」列是路径，「说明（英文原文）」列保留手册页条目原文（含路径与破折号），「中文译注」列为对应的中文翻译。表格按原文顺序排列，原来的缩进层级改由上方目录树体现。

| 目录 | 说明（英文原文） | 中文译注 |
| --- | --- | --- |
| `/` | - **`/`** — This is the root directory. This is where the whole tree starts. | 这是根目录。整棵目录树从这里开始。 |
| `/bin` | - **`/bin`** — This directory contains executable programs which are needed in single user mode and to bring the system up or repair it. | 该目录包含单用户模式下以及启动或修复系统时所需的可执行程序。 |
| `/boot` | - **`/boot`** — Contains static files for the boot loader. This directory holds only the files which are needed during the boot process. The map installer and configuration files should go to /sbin and /etc. The operating system kernel (initrd for example) must be located in either / or /boot. | 包含引导加载程序的静态文件。该目录只保存引导过程中需要的文件。映射安装程序和配置文件应放到 /sbin 和 /etc 中。操作系统内核（例如 initrd）必须位于 / 或 /boot 中。 |
| `/dev` | - **`/dev`** — Special or device files, which refer to physical devices. See mknod(1). | 特殊文件或设备文件，指向物理设备。参见 mknod(1)。 |
| `/etc` | - **`/etc`** — Contains configuration files which are local to the machine. Some larger software packages, like X11, can have their own subdirectories below /etc. Site-wide configuration files may be placed here or in /usr/etc. Nevertheless, programs should always look for these files in /etc and you may have links for these files to /usr/etc. | 包含本机专有的配置文件。一些较大的软件包（如 X11）可以在 /etc 下拥有自己的子目录。站点级配置文件可以放在这里或 /usr/etc 中。不过，程序应始终在 /etc 中查找这些文件，而你可以为这些文件建立指向 /usr/etc 的链接。 |
| `/etc/opt` | - **`/etc/opt`** — Host-specific configuration files for add-on applications installed in /opt. | 为安装在 /opt 中的附加应用程序提供的、与主机相关的配置文件。 |
| `/etc/sgml` | - **`/etc/sgml`** — This directory contains the configuration files for SGML (optional). | 该目录包含 SGML 的配置文件（可选）。 |
| `/etc/skel` | - **`/etc/skel`** — When a new user account is created, files from this directory are usually copied into the user's home directory. | 创建新用户账户时，通常会把该目录中的文件复制到用户的主目录中。 |
| `/etc/X11` | - **`/etc/X11`** — Configuration files for the X11 window system (optional). | X11 窗口系统的配置文件（可选）。 |
| `/etc/xml` | - **`/etc/xml`** — This directory contains the configuration files for XML (optional). | 该目录包含 XML 的配置文件（可选）。 |
| `/home` | - **`/home`** — On machines with home directories for users, these are usually beneath this directory, directly or not. The structure of this directory depends on local administration decisions (optional). | 在为用户提供主目录的机器上，这些主目录通常位于该目录之下，可能直接位于其下，也可能不直接位于其下。该目录的结构取决于本地管理决策（可选）。 |
| `/lib` | - **`/lib`** — This directory should hold those shared libraries that are necessary to boot the system and to run the commands in the root filesystem. | 该目录应存放启动系统以及运行根文件系统中的命令所必需的共享库。 |
| `/lib<qual>` | ``- **`/lib<qual>`** — These directories are variants of /lib on system which support more than one binary format requiring separate libraries (optional).`` | 在支持多种二进制格式、需要各自独立库的系统上，这些目录是 `/lib` 的变体（可选）。 |
| `/lib/modules` | - **`/lib/modules`** — Loadable kernel modules (optional). | 可加载内核模块（可选）。 |
| `/lost+found` | - **`/lost+found`** — This directory contains items lost in the filesystem. These items are usually chunks of files mangled as a consequence of a faulty disk or a system crash. | 该目录包含文件系统中丢失的条目。这些条目通常是磁盘故障或系统崩溃后损坏的文件片段。 |
| `/media` | - **`/media`** — This directory contains mount points for removable media such as CD and DVD disks or USB sticks. On systems where more than one device exists for mounting a certain type of media, mount directories can be created by appending a digit to the name of those available above starting with '0', but the unqualified name must also exist. | 该目录包含可移动介质（如 CD、DVD 光盘或 USB 存储设备）的挂载点。在某种介质存在多个设备需要挂载的系统上，可以在上述可用名称后追加一个数字来创建挂载目录，数字从 '0' 开始，但未加限定的名称也必须存在。 |
| `/media/floppy[1-9]` | - **`/media/floppy[1-9]`** — Floppy drive (optional). | 软盘驱动器（可选）。 |
| `/media/cdrom[1-9]` | - **`/media/cdrom[1-9]`** — CD-ROM drive (optional). | CD-ROM 驱动器（可选）。 |
| `/media/cdrecorder[1-9]` | - **`/media/cdrecorder[1-9]`** — CD writer (optional). | CD 刻录机（可选）。 |
| `/media/zip[1-9]` | - **`/media/zip[1-9]`** — Zip drive (optional). | Zip 驱动器（可选）。 |
| `/media/usb[1-9]` | - **`/media/usb[1-9]`** — USB drive (optional). | USB 驱动器（可选）。 |
| `/mnt` | - **`/mnt`** — This directory is a mount point for a temporarily mounted filesystem. In some distributions, /mnt contains subdirectories intended to be used as mount points for several temporary filesystems. | 该目录是临时挂载文件系统的挂载点。在某些发行版中，/mnt 包含若干子目录，用作多个临时文件系统的挂载点。 |
| `/opt` | - **`/opt`** — This directory should contain add-on packages that contain static files. | 该目录应存放包含静态文件的附加软件包。 |
| `/proc` | - **`/proc`** — This is a mount point for the proc filesystem, which provides information about running processes and the kernel. This pseudo-filesystem is described in more detail in proc(5). | 这是 proc 文件系统的挂载点，该文件系统提供关于运行中进程和内核的信息。这个伪文件系统在 proc(5) 中有更详细的描述。 |
| `/root` | - **`/root`** — This directory is usually the home directory for the root user (optional). | 该目录通常是 root 用户的主目录（可选）。 |
| `/sbin` | - **`/sbin`** — Like /bin, this directory holds commands needed to boot the system, but which are usually not executed by normal users. | 与 /bin 类似，该目录保存启动系统所需的命令，但这些命令通常不由普通用户执行。 |
| `/srv` | - **`/srv`** — This directory contains site-specific data that is served by this system. | 该目录包含由本系统提供服务的站点特定数据。 |
| `/sys` | - **`/sys`** — This is a mount point for the sysfs filesystem, which provides information about the kernel like /proc, but better structured, following the formalism of kobject infrastructure. | 这是 sysfs 文件系统的挂载点，它像 /proc 一样提供内核信息，但结构更好，遵循 kobject 基础设施的形式化规范。 |
| `/tmp` | - **`/tmp`** — This directory contains temporary files which may be deleted with no notice, such as by a regular job or at system boot up. | 该目录包含临时文件，这些文件可能在没有通知的情况下被删除，例如由定期任务删除或在系统启动时删除。 |
| `/usr` | - **`/usr`** — This directory is usually mounted from a separate partition. It should hold only sharable, read-only data, so that it can be mounted by various machines running Linux. | 该目录通常从单独的分区挂载。它应只保存可共享的只读数据，以便运行 Linux 的各类机器都能挂载它。 |
| `/usr/X11R6` | - **`/usr/X11R6`** — The X-Window system, version 11 release 6 (optional). | X-Window 系统，版本 11 第 6 版（可选）。 |
| `/usr/X11R6/bin` | - **`/usr/X11R6/bin`** — Binaries which belong to the X-Window system; often, there is a symbolic link from the more traditional /usr/bin/X11 to here. | 属于 X-Window 系统的二进制文件；通常有一个符号链接，从更传统的 /usr/bin/X11 指向这里。 |
| `/usr/X11R6/lib` | - **`/usr/X11R6/lib`** — Data files associated with the X-Window system. | 与 X-Window 系统相关的数据文件。 |
| `/usr/X11R6/lib/X11` | - **`/usr/X11R6/lib/X11`** — These contain miscellaneous files needed to run X; Often, there is a symbolic link from /usr/lib/X11 to this directory. | 这些目录包含运行 X 所需的各类文件；通常有一个符号链接，从 /usr/lib/X11 指向该目录。 |
| `/usr/X11R6/include/X11` | - **`/usr/X11R6/include/X11`** — Contains include files needed for compiling programs using the X11 window system. Often, there is a symbolic link from /usr/include/X11 to this directory. | 包含编译使用 X11 窗口系统的程序所需的头文件。通常有一个符号链接，从 /usr/include/X11 指向该目录。 |
| `/usr/bin` | - **`/usr/bin`** — This is the primary directory for executable programs. Most programs executed by normal users which are not needed for booting or for repairing the system and which are not installed locally should be placed in this directory. | 这是可执行程序的主要目录。普通用户执行的大多数程序，凡是不用于引导系统或修复系统、也不是本地安装的，都应放在该目录中。 |
| `/usr/bin/mh` | - **`/usr/bin/mh`** — Commands for the MH mail handling system (optional). | MH 邮件处理系统的命令（可选）。 |
| `/usr/bin/X11` | - **`/usr/bin/X11`** — is the traditional place to look for X11 executables; on Linux, it usually is a symbolic link to /usr/X11R6/bin. | 是查找 X11 可执行文件的传统位置；在 Linux 上，它通常是指向 /usr/X11R6/bin 的符号链接。 |
| `/usr/dict` | - **`/usr/dict`** — Replaced by /usr/share/dict. | 已被 /usr/share/dict 取代。 |
| `/usr/doc` | - **`/usr/doc`** — Replaced by /usr/share/doc. | 已被 /usr/share/doc 取代。 |
| `/usr/etc` | - **`/usr/etc`** — Site-wide configuration files to be shared between several machines may be stored in this directory. However, commands should always reference those files using the /etc directory. Links from files in /etc should point to the appropriate files in /usr/etc. | 需要在多台机器之间共享的站点级配置文件可以存放在该目录中。不过，命令应始终通过 /etc 目录引用这些文件。/etc 中文件的链接应指向 /usr/etc 中相应的文件。 |
| `/usr/games` | - **`/usr/games`** — Binaries for games and educational programs (optional). | 游戏和教育程序的二进制文件（可选）。 |
| `/usr/include` | - **`/usr/include`** — Include files for the C compiler. | C 编译器的头文件。 |
| `/usr/include/bsd` | - **`/usr/include/bsd`** — BSD compatibility include files (optional). | BSD 兼容头文件（可选）。 |
| `/usr/include/X11` | - **`/usr/include/X11`** — Include files for the C compiler and the X-Window system. This is usually a symbolic link to /usr/X11R6/include/X11. | C 编译器和 X-Window 系统的头文件。这通常是指向 /usr/X11R6/include/X11 的符号链接。 |
| `/usr/include/asm` | - **`/usr/include/asm`** — Include files which declare some assembler functions. This used to be a symbolic link to /usr/src/linux/include/asm. | 声明某些汇编器函数的头文件。这以前是指向 /usr/src/linux/include/asm 的符号链接。 |
| `/usr/include/linux` | - **`/usr/include/linux`** — This contains information which may change from system release to system release and used to be a symbolic link to /usr/src/linux/include/linux to get at operating-system-specific information. | 包含可能随系统发行版变化的信息，以前是指向 /usr/src/linux/include/linux 的符号链接，用于获取操作系统特定的信息。 |
| `/usr/include/g++` | - **`/usr/include/g++`** — Include files to use with the GNU C++ compiler. | 供 GNU C++ 编译器使用的头文件。 |
| `/usr/lib` | - **`/usr/lib`** — Object libraries, including dynamic libraries, plus some executables which usually are not invoked directly. More complicated programs may have whole subdirectories there. | 目标库，包括动态库，以及一些通常不直接调用的可执行文件。更复杂的程序可能在这里拥有完整的子目录。 |
| `/usr/lib<qual>` | ``- **`/usr/lib<qual>`** — These directories are variants of /usr/lib on system which support more than one binary format requiring separate libraries, except that the symbolic link /usr/lib<qual>/X11 is not required (optional).`` | 在支持多种二进制格式、需要各自独立库的系统上，这些目录是 /usr/lib 的变体，只是不要求提供符号链接 `/usr/lib<qual>/X11`（可选）。 |
| `/usr/lib/X11` | - **`/usr/lib/X11`** — The usual place for data files associated with X programs, and configuration files for the X system itself. On Linux, it usually is a symbolic link to /usr/X11R6/lib/X11. | 存放与 X 程序相关的数据文件以及 X 系统自身配置文件的常用位置。在 Linux 上，它通常是指向 /usr/X11R6/lib/X11 的符号链接。 |
| `/usr/lib/gcc-lib` | - **`/usr/lib/gcc-lib`** — contains executables and include files for the GNU C compiler, gcc(1). | 包含 GNU C 编译器 gcc(1) 的可执行文件和头文件。 |
| `/usr/lib/groff` | - **`/usr/lib/groff`** — Files for the GNU groff document formatting system. | GNU groff 文档格式化系统的文件。 |
| `/usr/lib/uucp` | - **`/usr/lib/uucp`** — Files for uucp(1). | uucp(1) 的文件。 |
| `/usr/local` | - **`/usr/local`** — This is where programs which are local to the site typically go. | 这是站点本地程序通常存放的位置。 |
| `/usr/local/bin` | - **`/usr/local/bin`** — Binaries for programs local to the site. | 站点本地程序的二进制文件。 |
| `/usr/local/doc` | - **`/usr/local/doc`** — Local documentation. | 本地文档。 |
| `/usr/local/etc` | - **`/usr/local/etc`** — Configuration files associated with locally installed programs. | 与本地安装程序相关的配置文件。 |
| `/usr/local/games` | - **`/usr/local/games`** — Binaries for locally installed games. | 本地安装游戏的二进制文件。 |
| `/usr/local/lib` | - **`/usr/local/lib`** — Files associated with locally installed programs. | 与本地安装程序相关的文件。 |
| `/usr/local/lib<qual>` | ``- **`/usr/local/lib<qual>`** — These directories are variants of /usr/local/lib on system which support more than one binary format requiring separate libraries (optional).`` | 在支持多种二进制格式、需要各自独立库的系统上，这些目录是 `/usr/local/lib` 的变体（可选）。 |
| `/usr/local/include` | - **`/usr/local/include`** — Header files for the local C compiler. | 本地 C 编译器的头文件。 |
| `/usr/local/info` | - **`/usr/local/info`** — Info pages associated with locally installed programs. | 与本地安装程序相关的 Info 页面。 |
| `/usr/local/man` | - **`/usr/local/man`** — Man pages associated with locally installed programs. | 与本地安装程序相关的 man 手册页。 |
| `/usr/local/sbin` | - **`/usr/local/sbin`** — Locally installed programs for system administration. | 本地安装的系统管理程序。 |
| `/usr/local/share` | - **`/usr/local/share`** — Local application data that can be shared among different architectures of the same OS. | 可以在同一操作系统的不同体系结构之间共享的本地应用数据。 |
| `/usr/local/src` | - **`/usr/local/src`** — Source code for locally installed software. | 本地安装软件的源代码。 |
| `/usr/man` | - **`/usr/man`** — Replaced by /usr/share/man. | 已被 /usr/share/man 取代。 |
| `/usr/sbin` | - **`/usr/sbin`** — This directory contains program binaries for system administration which are not essential for the boot process, for mounting /usr, or for system repair. | 该目录包含用于系统管理的程序二进制文件，这些程序对于引导过程、挂载 /usr 或系统修复并非必需。 |
| `/usr/share` | - **`/usr/share`** — This directory contains subdirectories with specific application data, that can be shared among different architectures of the same OS. Often one finds stuff here that used to live in /usr/doc or /usr/lib or /usr/man. | 该目录包含存放特定应用数据的子目录，这些数据可以在同一操作系统的不同体系结构之间共享。这里常常能找到过去位于 /usr/doc、/usr/lib 或 /usr/man 中的内容。 |
| `/usr/share/dict` | - **`/usr/share/dict`** — Contains the word lists used by spell checkers (optional). | 包含拼写检查器使用的词表（可选）。 |
| `/usr/share/dict/words` | - **`/usr/share/dict/words`** — List of English words (optional). | 英语单词列表（可选）。 |
| `/usr/share/doc` | - **`/usr/share/doc`** — Documentation about installed programs (optional). | 关于已安装程序的文档（可选）。 |
| `/usr/share/games` | - **`/usr/share/games`** — Static data files for games in /usr/games (optional). | /usr/games 中游戏的静态数据文件（可选）。 |
| `/usr/share/info` | - **`/usr/share/info`** — Info pages go here (optional). | Info 页面放在这里（可选）。 |
| `/usr/share/locale` | - **`/usr/share/locale`** — Locale information goes here (optional). | 区域设置（locale）信息放在这里（可选）。 |
| `/usr/share/man` | - **`/usr/share/man`** — Manual pages go here in subdirectories according to the man page sections. | man 手册页按手册章节放在这里的子目录中。 |
| `/usr/share/man/<locale>/man[1-9]` | ``- **`/usr/share/man/<locale>/man[1-9]`** — These directories contain manual pages for the specific locale in source code form. Systems which use a unique language and code set for all manual pages may omit the <locale> substring.`` | 这些目录包含特定区域设置的、以源代码形式提供的手册页。如果系统对所有手册页使用统一的语言和字符集，则可以省略 `<locale>` 子串。 |
| `/usr/share/misc` | - **`/usr/share/misc`** — Miscellaneous data that can be shared among different architectures of the same OS. | 可以在同一操作系统的不同体系结构之间共享的杂项数据。 |
| `/usr/share/nls` | - **`/usr/share/nls`** — The message catalogs for native language support go here (optional). | 本地语言支持（native language support）的消息目录放在这里（可选）。 |
| `/usr/share/sgml` | - **`/usr/share/sgml`** — Files for SGML (optional). | SGML 的文件（可选）。 |
| `/usr/share/sgml/docbook` | - **`/usr/share/sgml/docbook`** — DocBook DTD (optional). | DocBook DTD（可选）。 |
| `/usr/share/sgml/tei` | - **`/usr/share/sgml/tei`** — TEI DTD (optional). | TEI DTD（可选）。 |
| `/usr/share/sgml/html` | - **`/usr/share/sgml/html`** — HTML DTD (optional). | HTML DTD（可选）。 |
| `/usr/share/sgml/mathtml` | - **`/usr/share/sgml/mathtml`** — MathML DTD (optional). | MathML DTD（可选）。 |
| `/usr/share/terminfo` | - **`/usr/share/terminfo`** — The database for terminfo (optional). | terminfo 数据库（可选）。 |
| `/usr/share/tmac` | - **`/usr/share/tmac`** — Troff macros that are not distributed with groff (optional). | 不随 groff 分发的 troff 宏（可选）。 |
| `/usr/share/xml` | - **`/usr/share/xml`** — Files for XML (optional). | XML 的文件（可选）。 |
| `/usr/share/xml/docbook` | - **`/usr/share/xml/docbook`** — DocBook DTD (optional). | DocBook DTD（可选）。 |
| `/usr/share/xml/xhtml` | - **`/usr/share/xml/xhtml`** — XHTML DTD (optional). | XHTML DTD（可选）。 |
| `/usr/share/xml/mathml` | - **`/usr/share/xml/mathml`** — MathML DTD (optional). | MathML DTD（可选）。 |
| `/usr/share/zoneinfo` | - **`/usr/share/zoneinfo`** — Files for timezone information (optional). | 时区信息文件（可选）。 |
| `/usr/src` | - **`/usr/src`** — Source files for different parts of the system, included with some packages for reference purposes. Don't work here with your own projects, as files below /usr should be read-only except when installing software (optional). | 系统不同部分的源代码文件，随某些软件包提供以供参考。不要在这里处理你自己的项目，因为 /usr 下的文件除安装软件时外应保持只读（可选）。 |
| `/usr/src/linux` | - **`/usr/src/linux`** — This was the traditional place for the kernel source. Some distributions put here the source for the default kernel they ship. You should probably use another directory when building your own kernel. | 这曾是内核源代码的传统位置。某些发行版会把它们所发布默认内核的源代码放在这里。构建自己的内核时，你大概应该使用另一个目录。 |
| `/usr/tmp` | - **`/usr/tmp`** — Obsolete. This should be a link to /var/tmp. This link is present only for compatibility reasons and shouldn't be used. | 已过时。它应是指向 /var/tmp 的链接。该链接仅出于兼容性原因存在，不应使用。 |
| `/var` | - **`/var`** — This directory contains files which may change in size, such as spool and log files. | 该目录包含大小可能变化的文件，例如假脱机文件和日志文件。 |
| `/var/account` | - **`/var/account`** — Process accounting logs (optional). | 进程记账日志（可选）。 |
| `/var/adm` | - **`/var/adm`** — This directory is superseded by /var/log and should be a symbolic link to /var/log. | 该目录已被 /var/log 取代，应是指向 /var/log 的符号链接。 |
| `/var/backups` | - **`/var/backups`** — Reserved for historical reasons. | 出于历史原因保留。 |
| `/var/cache` | - **`/var/cache`** — Data cached for programs. | 为程序缓存的数据。 |
| `/var/cache/fonts` | - **`/var/cache/fonts`** — Locally-generated fonts (optional). | 本地生成的字体（可选）。 |
| `/var/cache/man` | - **`/var/cache/man`** — Locally-formatted man pages (optional). | 本地格式化的 man 手册页（可选）。 |
| `/var/cache/www` | - **`/var/cache/www`** — WWW proxy or cache data (optional). | WWW 代理或缓存数据（可选）。 |
| `/var/cache/<package>` | - **`/var/cache/<package>`** — Package specific cache data (optional). | 特定于软件包的缓存数据（可选）。 |
| `/var/catman/cat[1-9] or /var/cache/man/cat[1-9]` | - **`/var/catman/cat[1-9] or /var/cache/man/cat[1-9]`** — These directories contain preformatted manual pages according to their man page section. (The use of preformatted manual pages is deprecated.) | 这些目录包含按手册章节划分的预格式化手册页。（使用预格式化手册页的做法已废弃。） |
| `/var/crash` | - **`/var/crash`** — System crash dumps (optional). | 系统崩溃转储（可选）。 |
| `/var/cron` | - **`/var/cron`** — Reserved for historical reasons. | 出于历史原因保留。 |
| `/var/games` | - **`/var/games`** — Variable game data (optional). | 可变的游戏数据（可选）。 |
| `/var/lib` | - **`/var/lib`** — Variable state information for programs. | 程序的可变状态信息。 |
| `/var/lib/hwclock` | - **`/var/lib/hwclock`** — State directory for hwclock (optional). | hwclock 的状态目录（可选）。 |
| `/var/lib/misc` | - **`/var/lib/misc`** — Miscellaneous state data. | 杂项状态数据。 |
| `/var/lib/xdm` | - **`/var/lib/xdm`** — X display manager variable data (optional). | X 显示管理器的可变数据（可选）。 |
| `/var/lib/<editor>` | - **`/var/lib/<editor>`** — Editor backup files and state (optional). | 编辑器备份文件和状态（可选）。 |
| `/var/lib/<name>` | - **`/var/lib/<name>`** — These directories must be used for all distribution packaging support. | 这些目录必须用于所有发行版打包支持。 |
| `/var/lib/<package>` | - **`/var/lib/<package>`** — State data for packages and subsystems (optional). | 软件包和子系统的状态数据（可选）。 |
| `/var/lib/<pkgtool>` | - **`/var/lib/<pkgtool>`** — Packaging support files (optional). | 打包支持文件（可选）。 |
| `/var/local` | - **`/var/local`** — Variable data for /usr/local. | /usr/local 的可变数据。 |
| `/var/lock` | ``- **`/var/lock`** — Lock files are placed in this directory. The naming convention for device lock files is LCK..<device> where <device> is the device's name in the filesystem. The format used is that of HDU UUCP lock files, that is, lock files contain a PID as a 10-byte ASCII decimal number, followed by a newline character.`` | 锁文件放在该目录中。设备锁文件的命名约定是 `LCK..<device>`，其中 `<device>` 是设备在文件系统中的名称。使用的格式是 HDU UUCP 锁文件格式，即锁文件包含一个以 10 字节 ASCII 十进制数表示的 PID，后面跟一个换行符。 |
| `/var/log` | - **`/var/log`** — Miscellaneous log files. | 杂项日志文件。 |
| `/var/opt` | - **`/var/opt`** — Variable data for /opt. | /opt 的可变数据。 |
| `/var/mail` | - **`/var/mail`** — Users' mailboxes. Replaces /var/spool/mail. | 用户的邮箱。取代 /var/spool/mail。 |
| `/var/msgs` | - **`/var/msgs`** — Reserved for historical reasons. | 出于历史原因保留。 |
| `/var/preserve` | - **`/var/preserve`** — Reserved for historical reasons. | 出于历史原因保留。 |
| `/var/run` | - **`/var/run`** — Run-time variable files, like files holding process identifiers (PIDs) and logged user information (utmp). Files in this directory are usually cleared when the system boots. | 运行时可变文件，例如保存进程标识符（PID）和已登录用户信息（utmp）的文件。该目录中的文件通常在系统启动时被清除。 |
| `/var/spool` | - **`/var/spool`** — Spooled (or queued) files for various programs. | 各种程序的假脱机（或排队）文件。 |
| `/var/spool/at` | - **`/var/spool/at`** — Spooled jobs for at(1). | at(1) 的假脱机作业。 |
| `/var/spool/cron` | - **`/var/spool/cron`** — Spooled jobs for cron(8). | cron(8) 的假脱机作业。 |
| `/var/spool/lpd` | - **`/var/spool/lpd`** — Spooled files for printing (optional). | 打印用的假脱机文件（可选）。 |
| `/var/spool/lpd/printer` | - **`/var/spool/lpd/printer`** — Spools for a specific printer (optional). | 特定打印机的假脱机目录（可选）。 |
| `/var/spool/mail` | - **`/var/spool/mail`** — Replaced by /var/mail. | 已被 /var/mail 取代。 |
| `/var/spool/mqueue` | - **`/var/spool/mqueue`** — Queued outgoing mail (optional). | 排队的待发出邮件（可选）。 |
| `/var/spool/news` | - **`/var/spool/news`** — Spool directory for news (optional). | 新闻的假脱机目录（可选）。 |
| `/var/spool/rwho` | - **`/var/spool/rwho`** — Spooled files for rwhod(8) (optional). | rwhod(8) 的假脱机文件（可选）。 |
| `/var/spool/smail` | - **`/var/spool/smail`** — Spooled files for the smail(1) mail delivery program. | smail(1) 邮件投递程序的假脱机文件。 |
| `/var/spool/uucp` | - **`/var/spool/uucp`** — Spooled files for uucp(1) (optional). | uucp(1) 的假脱机文件（可选）。 |
| `/var/tmp` | - **`/var/tmp`** — Like /tmp, this directory holds temporary files stored for an unspecified duration. | 与 /tmp 类似，该目录保存存放时间不确定的临时文件。 |
| `/var/yp` | - **`/var/yp`** — Database files for NIS, formerly known as the Sun Yellow Pages (YP). | NIS 的数据库文件，NIS 以前称为 Sun Yellow Pages（YP）。 |

`/usr/include/linux` 条目在手册页原文中还附有下面这段说明：

(Note that one should have include files there that work correctly with the current libc and in user space. However, Linux kernel source is not designed to be used with user programs and does not know anything about the libc you are using. It is very likely that things will break if you let /usr/include/asm and /usr/include/linux point at a random kernel tree. Debian systems don't do this and use headers from a known good kernel version, provided in the libc*-dev package.)

> 译：（注意，那里应放置能与当前 libc 以及用户空间正确配合的头文件。然而，Linux 内核源代码并非为供用户程序使用而设计，它对你正在使用的 libc 一无所知。如果你让 /usr/include/asm 和 /usr/include/linux 指向任意一个内核源码树，事情很可能会出问题。Debian 系统不这样做，而是使用来自已知良好内核版本的头文件，由 libc*-dev 软件包提供。）
