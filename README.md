
Создаем в [VirualBox](https://www.virtualbox.org/) виртуальную машину [OpenWrt 24.10.3](https://downloads.openwrt.org/releases/24.10.3/targets/x86/64/openwrt-24.10.3-x86-64-generic-ext4-combined.img.gz) с пакетом [youtubeUnblock 1.1.0-2](https://github.com/Waujito/youtubeUnblock/releases/download/v1.1.0/youtubeUnblock-1.1.0-2-2d579d5-x86_64-openwrt-23.05.ipk). Такую виртуальную машину можно использовать, например, для тестирования обхода блокировок.

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

2. Создаем виртуальную машину: Выберите **Машина** -> **Создать**:
   - **Имя и тип ОС**
      - Имя: **OpenWrt**
      - Тип: **Linux**
      - Подтип: **Other Linux**
      - Версия: **Other Linux (64-bit)**
   - **Оборудование**
      - Основная память: **256 МБ** (минимум для OpenWRT)
      - Процессоры: **1**
   - **Жесткий диск**
      **\[x\]** Использовать существующий виртуальный жесткий диск -> **OpenWrt.vdi** -> **Готово**

3. Настраиваем виртуальную машину: **OpenWrt** -> **Настроить**:
   - **Общие**
      - **Порядок загрузки**
         - **\[ \]** Гибкий диск
         - **\[ \]** Оптический диск
      - **Дополнительно**
         - Общий буфер обмена: **Двунаправленный**
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

4. Запускаем еще виртуальную машину **Windows**, у которой в настройках выбрана **Внутренняя сеть (intnet)**.

5. Подключаемся к **OpenWrt** для ее дальнейшей настройки из виртуальной машины **Windows** по **Web**:
   - Веб-интерфейс: http://192.168.1.1/
   - Учетные данные: **root** / **без пароля**

6. Разрешаем подключение к **Web**-интерфейсу **OpenWrt** на **WAN**-интерфейсе (чтобы можно было управлять **OpenWrt** не только из виртуальной машины):
   **Network** -> **Firewall** -> **Zones: wan x Input = accept**

7. Подключаемся к роутеру по SSH через [PuTTY](https://the.earth.li/~sgtatham/putty/latest/w32/putty.exe) (при запросе пароля просто жмем **ENTER**):
   ```powershell
      putty.exe 192.168.1.1
   ```
8. Устанавливаем зависимости для **youtubeUnblock** - в окне **PuTTY** выполняем команду:
   ```bash
      opkg update && opkg install kmod-nfnetlink-queue kmod-nft-queue kmod-nf-conntrack
   ```

9. Скачиваем пакеты **youtubeUnblock**
   - [youtubeUnblock-1.1.0-2-2d579d5-x86_64-openwrt-23.05.ipk](https://github.com/Waujito/youtubeUnblock/releases/download/v1.1.0/youtubeUnblock-1.1.0-2-2d579d5-x86_64-openwrt-23.05.ipk)
   - [luci-app-youtubeUnblock-1.1.0-1-473af29.ipk](https://github.com/Waujito/youtubeUnblock/releases/download/v1.1.0/luci-app-youtubeUnblock-1.1.0-1-473af29.ipk)

10. Устанавливаем их через Web-интерфейс:
   - **System** -> **Software**, жмем **Upload package** -> выбираем **youtubeUnblock** -> жмем **Upload** -> **Install** -> **Dismiss**.
   - **System** -> **Software**, жмем **Upload package** -> выбираем **luci-app-youtubeUnblock** -> жмем **Upload** -> **Install** -> **Dismiss**.

11. Для быстрого старта/остановки виртуальной машины можно создать скрипты:
   - **OpenWrt-start.vbs**
```vbscript
Option Explicit
Dim objShell, objEnv, strPath, strCommand, intReturn
Set objShell = CreateObject ("WScript.Shell")
Set objEnv = objShell.Environment ("PROCESS")
strPath = objEnv ("PATH")
objEnv ("PATH") = objEnv ("ProgramFiles") & "\Oracle\VirtualBox;" & strPath
strCommand = "vboxmanage startvm --type headless OpenWrt"
On Error Resume Next
intReturn = objShell.Run (strCommand, 0, False)
If Err.Number <> 0 Then
  WScript.Echo "Ошибка при запуске VM: " & Err.Description
  WScript.Quit 1
End If
On Error GoTo 0
WScript.Quit 0
```
   - **OpenWrt-stop.bat**
```vbscript
Option Explicit
Dim objShell, objEnv, strPath, strCommand, intReturn
Set objShell = CreateObject ("WScript.Shell")
Set objEnv = objShell.Environment ("PROCESS")
strPath = objEnv ("PATH")
objEnv("PATH") = objEnv ("ProgramFiles") & "\Oracle\VirtualBox;" & strPath
On Error Resume Next
intReturn = objShell.Run ("vboxmanage showvminfo OpenWrt --machinereadable", 0, True)
If Err.Number <> 0 Or intReturn <> 0 Then
  WScript.Echo "Виртуальная машина OpenWrt не найдена или не запущена."
  WScript.Quit 1
End If
On Error GoTo 0
strCommand = "vboxmanage controlvm OpenWrt poweroff"
On Error Resume Next
intReturn = objShell.Run (strCommand, 0, True)
If Err.Number <> 0 Then
  WScript.Echo "Ошибка при остановке VM: " & Err.Description
  WScript.Quit 1
End If
On Error GoTo 0
WScript.Sleep 2000
WScript.Quit 0
```

### Ссылки

- [OpenWRT как виртуальный роутер: настройка DHCP и интернет-доступа](https://www.youtube.com/watch?v=xbyBIu8Gy8w)
