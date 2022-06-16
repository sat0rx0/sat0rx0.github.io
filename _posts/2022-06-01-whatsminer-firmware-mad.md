---
layout: post
title:  "Модификация прошивки WhatsMiner M3x"
date:   2022-06-03 11:56:28
categories: asic whatsminer firmware
---

Предлагаю свое решение по модификации бинарного файла прошивки Whatsminer.
## Цели и задачи.
В стандартной прошивке WhatsMiner M3x заблокирован SSH с целью обеспечения безопасности. Даде если SSH разблокирован с помощью WhatsMinerTool, то вход в систему невозможен.
Поставлена задача - можифицировать прошивку так, чтобы демон SSH запускался автоматически и был возможен вход для пользователя root  в систему удаленно.
## Исходные данные.
Прошика ASIC представляет из себя слегка модифицированный OpenWRT дистрибутив GNU/Linux. На официальном сайте распросттраняется образ системы для прошивки через SDCard. Давайте взглянем, что это за файл:

    [sat0r@sat0r-book WM]$ file WhatsMiner-SDCard-Burn-Image-h6os-20220422.18.img 
    WhatsMiner-SDCard-Burn-Image-h6os-20220422.18.img: data

Так мы поняли, что это бинарник - просто смонтировать не выйдет. Дальше в дело вступает утилита binwalk:

    [sat0r@sat0r-book WM]$ binwalk WhatsMiner-SDCard-Burn-Image-h6os-20220422.18.img 

    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    169984        0x29800         device tree image (dtb)
    878080        0xD6600         SHA256 hash constants, little endian
    899256        0xDB8B8         device tree image (dtb)
    980589        0xEF66D         Certificate in DER format (x509 v3), header length: 4, sequence length: 5416
    1133180       0x114A7C        CRC32 polynomial table, little endian
    1134636       0x11502C        Intel x86 or x64 microcode, sig 0x0000000c, pf_mask 0x01, 2000-07-02, rev 0x4a0cec49, size 2
    1169976       0x11DA38        Android bootimg, kernel size: 2037543936 bytes, kernel addr: 0x206F7420, ramdisk size: 1684104562 bytes, ramdisk addr: 0x72617020, product name: "ash read :offset %x, %d bytes %s"
    1171482       0x11E01A        Copyright string: "Copyright (C) 2010 Charles Cazabon."
    2275840       0x22BA00        SHA256 hash constants, little endian
    2297016       0x230CB8        device tree image (dtb)
    2378349       0x244A6D        Certificate in DER format (x509 v3), header length: 4, sequence length: 5416
    2530940       0x269E7C        CRC32 polynomial table, little endian
    2532396       0x26A42C        Intel x86 or x64 microcode, sig 0x0000000c, pf_mask 0x01, 2000-07-02, rev 0x4a0cec49, size 2
    2567736       0x272E38        Android bootimg, kernel size: 2037543936 bytes, kernel addr: 0x206F7420, ramdisk size: 1684104562 bytes, ramdisk addr: 0x72617020, product name: "ash read :offset %x, %d bytes %s"
    2569242       0x27341A        Copyright string: "Copyright (C) 2010 Charles Cazabon."
    3347456       0x331400        device tree image (dtb)
    3486720       0x353400        LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: -1 bytes
    3522560       0x35C000        Microsoft executable, portable (PE)
    3653384       0x37BF08        Intel x86 or x64 microcode, pf_mask 0x00, 1918-10-02, rev 0x10021910, size 2048
    3654692       0x37C424        Intel x86 or x64 microcode, sig 0x19930522, pf_mask 0x01, 1DFC-10-02, rev 0x0001, size 2048
    3671040       0x380400        Microsoft executable, portable (PE)
    3951623       0x3C4C07        Unix path: /home/user/lichee/tools/pack/out_android/
    3952482       0x3C4F62        Unix path: /home/user/lichee/tools/pack/out_android/boot.img  /home/user/imgdata/boot
    4105215       0x3EA3FF        Unix path: /home/user/lichee/tools/pack/out_android/
    4105934       0x3EA6CE        Unix path: /home/user/lichee/tools/pack/out_android/boot.img  /home/user/imgdata/boot
    4116480       0x3ED000        Microsoft executable, portable (PE)
    4164356       0x3F8B04        Intel x86 or x64 microcode, sig 0xffffffe0, pf_mask 0x01, 1E2C-10-01, size 1
    4192332       0x3FF84C        LZMA compressed data, properties: 0x65, dictionary size: 0 bytes, uncompressed size: 128 bytes
    4208716       0x40384C        LZMA compressed data, properties: 0x65, dictionary size: 0 bytes, uncompressed size: 128 bytes
    4225100       0x40784C        LZMA compressed data, properties: 0x65, dictionary size: 0 bytes, uncompressed size: 128 bytes
    4241484       0x40B84C        LZMA compressed data, properties: 0x65, dictionary size: 0 bytes, uncompressed size: 128 bytes
    4275200       0x413C00        bzip2 compressed data, block size = 900k
    9538048       0x918A00        PC bitmap, Windows 3.x format,, 1280 x 720 x 32
    13572608      0xCF1A00        PC bitmap, Windows 3.x format,, 246 x 257 x 24    
    14255104      0xD98400        device tree image (dtb)
    14255324      0xD984DC        gzip compressed data, maximum compression, from Unix, last modified: 1970-01-01 00:00:00 (null date)
    18856960      0x11FBC00       LUKS_MAGIC version 0x1 aes sha256
    51151762      0x30C8392       End of Zip archive, footer length: -1900
    63454229      0x3C83C15       MySQL MISAM index file Version 6
    71286784      0x43FC000       device tree image (dtb)
    71287004      0x43FC0DC       gzip compressed data, maximum compression, from Unix, last modified: 1970-01-01 00:00:00 (null date)
    75888640      0x485F800       Linux EXT filesystem, blocks count: 8192, image size: 8388608, rev 2.0, ext4 filesystem data, UUID=57f8f4bc-abf4-655f-bf67-946fc0f9c0f9
    84278272      0x505FC00       Linux EXT filesystem, blocks count: 8192, image size: 8388608, rev 2.0, ext4 filesystem data, UUID=57f8f4bc-abf4-655f-bf67-946fc0f9c0f9

Самые интересные для нас записи - последние две. Это образы файловых систем. Давайте "отрежем" последнюю.

    [sat0r@sat0r-book WM]$ dd if=WhatsMiner-SDCard-Burn-Image-h6os-20220422.18.img of=fs1.ext4 skip=84278272 bs=1

Получился файл с образом файловой системы. Его можно просто смонтировать.

    [sat0r@sat0r-book WM]$ sudo mount fs1.ext4 /mnt/IMG-1/

Проверяем что там у нас лежит:

    [sat0r@sat0r-book WM]$ ls /mnt/IMG-1/
    etc  lost+found
    [sat0r@sat0r-book WM]$ ls /mnt/IMG-1/etc/
    board.d   e2fsck.conf        hosts                 inittab          modules.d  protocols  resolv.conf     ssl              uboot.md5
    config    factory_test_step  hotplug.d             iproute2         mtab       rc.button  services        sysctl.conf      uci-defaults
    crontabs  firewall.user      hotplug.json          localtime        passwd     rc.common  shadow          sysctl.d
    diag.sh   fstab              hotplug-preinit.json  luci-uploads     preinit    rc.d       shadow.default  sysupgrade.conf
    dropbear  group              init.d                microbt_release  profile    rc.local   shells          TZ

Ура! Это каталок /etc нашей системы OpenWrt. Можно отредактировать необходимое, а затем собрать навый образ, "отрезав" заголовок от официального и подклеив наш, отредактированный.