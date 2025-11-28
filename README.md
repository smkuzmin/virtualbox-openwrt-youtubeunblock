Создаем в [VirualBox](https://www.virtualbox.org/) виртуальную машину [OpenWrt 24.10.3](https://downloads.openwrt.org/releases/24.10.3/targets/x86/64/openwrt-24.10.3-x86-64-generic-ext4-combined.img.gz) с пакетом [youtubeUnblock 1.1.0-2](https://github.com/Waujito/youtubeUnblock/releases/download/v1.1.0/youtubeUnblock-1.1.0-2-2d579d5-x86_64-openwrt-23.05.ipk). Ее можно использовать, например, для тестирования обхода блокировок.

### Описание виртуальной машины

- Веб-интерфейс: **http://<IP_WAN_интерфейса>/**
- Учетные данные: **root** / **без пароля**
- WAN (eth1): **DHCP-клиент** - получает адрес от физического роутера
- LAN (br-lan): **DHCP-сервер 192.168.1.1/24** - раздает адреса во **внутреннюю сеть (intnet)** другим виртульным машинам

### Создание виртуальной машины

1. Скачиваем **generic-ext4-combined.img.gz** [отсюда](https://downloads.openwrt.org/releases/24.10.3/targets/x86/64/) и конвертируем его в **OpenWrt.vdi** с помощью скрипта **vdi-build.bat**:
```powershell
@echo off
path %ProgramFiles%\Oracle\VirtualBox;%ProgramFiles%\7-Zip;%PATH%
cls
echo.
echo === Downloading ===
echo.
curl -O "https://downloads.openwrt.org/releases/24.10.3/targets/x86/64/openwrt-24.10.3-x86-64-generic-ext4-combined.img.gz"
echo.
echo === Unpacking ===
7z x -y openwrt*.img.gz
echo.
echo === Converting ===
echo.
del>nul 2>nul /A /F /Q "OpenWrt.vdi"
for /R "%~dp0" %%i in ("openwrt*.img") do set OPENWRT_IMAGE=%%~nxi
vboxmanage convertfromraw "%OPENWRT_IMAGE%" "OpenWrt.vdi" --format VDI
del>nul 2>nul /A /F /Q "openwrt*.img"
echo.
echo === Resizing ===
echo.
vboxmanage modifyhd --resize 512 "OpenWrt.vdi"
echo.
echo === UUID changing ===
echo.
vboxmanage internalcommands sethduuid "OpenWrt.vdi"
```

2. Создаем виртуальную машину: **VirtualBox** -> **Машина** -> **Создать**:
   - **Имя и тип ОС**
      - Имя: **OpenWrt**
      - Тип: **Linux**
      - Подтип: **Other Linux**
      - Версия: **Other Linux (64-bit)**
   - **Оборудование**
      - Основная память: **256 МБ** (минимум для OpenWrt)
      - Процессоры: **1**
   - **Жесткий диск**
      - **\[x\]** Использовать существующий виртуальный жесткий диск -> **OpenWrt.vdi**
    и жмем **Готово**.

3. Настраиваем виртуальную машину: **VirtualBox** -> **OpenWrt** -> **Настроить**:
   - **Общие**
      - **Дополнительно**
         - Общий буфер обмена: **Двунаправленный**
   - **Система**
      - **Порядок загрузки**
         - **\[ \]** Гибкий диск
         - **\[ \]** Оптический диск
   - **Дисплей**
      - Видеопамять: **7 МБ**
   - **Аудио**
      - **\[ \]** Включить аудио
   - **Сеть**
      - **Адаптер 1** (LAN)
      - Тип подключения: **Внутренняя сеть**
      - Имя: **intnet**
   - **Адаптер 2** (WAN)
      - **\[X\]** Включить сетевой адаптер
      - Тип подключения: **Сетевой мост**
      - Имя: **Выберите ваш физический сетевой адаптер**
   - **USB**
      - **\[ \]** Включить контроллер USB
    и жмем **OK** для окончания настройки.

4. Запускаем созданную виртуальную машину: **VirtualBox** -> **OpenWrt** -> **Запустить**.

5. Еще создаем или используем существующую виртуальную машину **Windows**, у которой в настройках указываем:
    - **Сеть**
      - **Адаптер 1** (LAN)
      - Тип подключения: **Внутренняя сеть**
      - Имя: **intnet**
    и запускаем ее: **VirtualBox** -> **Windows** -> **Запустить**.

6. Из виртуальной машины **Windows** подключаемся к **OpenWrt**: http://192.168.1.1/ (**root** / **без пароля**) и разрешаем подключения на **WAN**-интерфейсе:
   - **Network** -> **Firewall** -> **Zones: wan x Input = accept**

7. Заходим в терминал виртуальной машины **OpenWrt** (просто жмем **ENTER**) и устанавливаем зависимости для **youtubeUnblock**:
   ```bash
      opkg update && opkg install kmod-nfnetlink-queue kmod-nft-queue kmod-nf-conntrack
   ```

8. На ПК скачиваем пакеты **youtubeUnblock**:
   - [youtubeUnblock-1.1.0-2-2d579d5-x86_64-openwrt-23.05.ipk](https://github.com/Waujito/youtubeUnblock/releases/download/v1.1.0/youtubeUnblock-1.1.0-2-2d579d5-x86_64-openwrt-23.05.ipk)
   - [luci-app-youtubeUnblock-1.1.0-1-473af29.ipk](https://github.com/Waujito/youtubeUnblock/releases/download/v1.1.0/luci-app-youtubeUnblock-1.1.0-1-473af29.ipk)

9. Затем устанавливаем их через Web-интерфейс **OpenWrt**:
   - **System** -> **Software** -> **Upload package** -> выбираем **youtubeUnblock** -> **Upload** -> **Install** -> **Dismiss**.
   - **System** -> **Software** -> **Upload package** -> выбираем **luci-app-youtubeUnblock** -> **Upload** -> **Install** -> **Dismiss**.

10. Для быстрого старта/остановки виртуальной машины можно создать скрипты и поместить их на рабочий стол:
   - **WrtON.vbs**
```vbscript
Option Explicit
Dim objShell, objEnv, strPath, strCommand
Set objShell = CreateObject ("WScript.Shell")
Set objEnv = objShell.Environment ("PROCESS")
strPath = objEnv ("PATH")
objEnv ("PATH") = objEnv ("ProgramFiles") & "\Oracle\VirtualBox;" & strPath
strCommand = "vboxmanage startvm --type headless OpenWrt"
objShell.Run strCommand, 0, False
objShell.Popup "OpenWrt is starting...", 1, "Start", 64
```
   - **WrtOFF.bat**
```vbscript
Option Explicit
Dim objShell, objEnv, strPath, strCommand
Set objShell = CreateObject ("WScript.Shell")
Set objEnv = objShell.Environment ("PROCESS")
strPath = objEnv ("PATH")
objEnv ("PATH") = objEnv ("ProgramFiles") & "\Oracle\VirtualBox;" & strPath
strCommand = "vboxmanage controlvm OpenWrt poweroff"
objShell.Run strCommand, 0, True
objShell.Popup "OpenWrt stops...", 1, "Stop", 64
```

### Ссылки

- [OpenWRT как виртуальный роутер: настройка DHCP и интернет-доступа](https://www.youtube.com/watch?v=xbyBIu8Gy8w)
