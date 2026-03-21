# EasyCheatDetector (eCD) v3.13 реверс (В ознакомительных целях, для разработчиков чекеров, или читов.)

> Бесплатный античит-сканер от FUNGUN для CS 1.6 / CS:GO / CS2 / Minecraft / RUST и других Source/GoldSrc игр.
> Репозиторий: https://github.com/UnrealKaraulov/EasyCheatDetector

```
SHA256  : bbc34ecedfdbc3f2efe65d50ccff69995e54cdd0f75bac66fbe5a26285983151
MD5    : f744b4759949c77347aca84706255081
Тип    : PE32 (x86), GUI, MSVC 2022 v17.6, подпись Authenticode
Размер  : 9.63 MB
Автор  : FUNGUN (ska@fungun.net)
Исходник : D:\a\eCD\eCD\eCD\src\eCD.cpp (сборка на Azure DevOps CI/CD)
Build  : ETO_ZAPRETNAYA_INFORMATIA\openssl\...
Версия  : 3.13 (20.03.2026)
```

Заявленные показатели обнаружения: CS 1.6 95%, Minecraft 60%, CS2 50%, CS:Source 30%.
Шифрование трафика: AES-256 + RSA-2048. Техническая информация удаляется через 24 часа.

## Инструменты анализа

IDA Pro 9.3, Detect It Easy, x64dbg кастомные PowerShell/Python скрипты для расшифровки строк.

## Статически влинкованные библиотеки

OpenSSL 3.5.0-dev, SQLite 3.50.1, cpp-httplib 0.16.3, tao::json/JAXN, blackbone, asmjit, GDI+, minizip/zlib.

## Обфускация строк

662+ строки зашифрованы на этапе компиляции. Три схемы XOR:

**Тип 1** - позиционный XOR по int16 массивам на стеке (+400 функций-обёрток, диапазон `0x450000`-`0x660000`):
```c
// ключ = адрес кода, лежит в переменной сразу после массива на стеке
decoded[i] = ((key - i - 150) & 0xFFFF) ^ (array[i] & 0xFFFF)

// пример: sub_64DF90, ключ = 0x6DDDBB, 101 элемент
// -> "To start PC analyze please press SCAN..."
```

**Тип 2** - побайтовый XOR через identity lookup-таблицу:
```c
decoded[i] = encrypted_byte[i] ^ key
// byte_AE9164 ключ 0xAC -> "ShellExecuteW"
// byte_AE9198 ключ 0x63 -> "CreateCompatibleDC"
// byte_AE918C ключ 0x7B -> "BitBlt"
// byte_AE91B0 ключ 0x08 -> "[SELF PROTECTION] Error 5..."
// доп. ключи: 0x0C, 0x10, 0x17
```

**Тип 3** - word lookup-таблица + XOR с адресом кода (замаскирован как `~(A & B) & (A | B)` = `A ^ B`):
```c
result = word_table[encrypted_value] ^ code_address_key
// word_AE9224  ключ &loc_6DFC7E+1 -> "EasyCheatDetectorIsAlive"
// word_AE8678  ключ &loc_6DDE39   -> "Software\Valve\Steam\ActiveProcess"
```

<details>
<summary>Все расшифрованные строки Типа 3</summary>

```
sub_652910  EasyCheatDetectorIsAlive
sub_5FDD60  Software\Valve\Steam\ActiveProcess
sub_5FCD10  engine.dll
sub_5FCF00  .text
sub_5FD0B0  engine.dll
sub_5FD2A0  .text
sub_5FD450  GSClient
sub_60BDE0  INJECTED
sub_618A00  INJECTED
sub_5F8B80  SteamPath
sub_5F1A30  Users
sub_61DD50  MENYA LOMAT'!
sub_5E24F0  [SELPROTECTION] Found non exiting registry key!
sub_5E60E0  No parent!
sub_62C1F0  Stage: Cheat
```
Заметка: "non exiting" опечатка бомжа не знающего базового английского (должно быть "non existing")
</details>

## Порядок работы

```
 1. Проверка ОС (>= Vista, иначе предлагает WinXP версию)
 2. CRC32 целостность: .text, .rdata, overlay
 3. Диалог EULA (EULA_22_02_2026 из реестра)
 4. SeDebugPrivilege (активация + верификация через GetTokenInformation)
 5. Приоритет ABOVE_NORMAL_PRIORITY_CLASS
 6. Инициализация GDI+, UI-анимация из ZIP в PE-ресурсах (ID 2001-2004)
 7. Мьютекс (5 попыток: 500/2000/3000/5000 мс)
 8. Запуск потоков:
    - UI: перерисовка каждые 20мс
    - Watchdog: при заморозке потоков эскалирует приоритет до REALTIME
 9. Проверка GameGuard (сервисы acdrv/ggsvc)
10. Поиск игрового процесса
11. DebugActiveProcess(game_PID)
12. Правила файрвола через COM INetFwPolicy2
13. Установка корневого CA из PFX-ресурса #111
14. WMI-отпечаток железа
15. Загрузка/обновление БД читов
16. 14 СТАДИЙ СКАНИРОВАНИЯ
17. Сборка JSON-отчёта, POST на сервер
18. Вывод ReportID
```

## Поддерживаемые игры

| Игра | Процессы | Модули | Папки |
|------|-----------|-------------|-------------|
| CS 1.6 | hl.exe, cstrike.exe, cs.exe, cs_new.exe, cstrike_new.exe, game_start.exe | hw.dll, sw.dll, engine.dll, hl_res.dll, v8.dll, dsound.dll, opengl32.dll, wdmaud.drv, demoplayer.dll, core.dll, rclient.dll | cstrike/, cstrike_addons/, cstrike_hd/, valve/, platform/ |
| CS 1.6 NextClient | cstrike.exe | voice_miles.dll, GameUI.dll, dsound.dll, opengl32.dll, wdmaud.drv | |
| CS:Source | hl2.exe, cstrike_win64.exe, cstrike.exe | client.dll | bin/ |
| CS:GO | csgo.exe, csgo_legacy_app.exe | client.dll, bin/panorama.dll | csgo, platform |
| CS2 | cs2.exe | client.dll, engine2.dll, panorama.dll | subtools, qt5_plugins |
| CS:Zero | czero.exe | | valve, czero |
| Minecraft | java.exe, javaw.exe, minecraft.windows.exe, MinecraftJava.exe | lwjgl_opengl.dll, glfw.dll, lwjgl.dll, jvm.dll, xsapi.dll | |
| RUST | rustclient.exe | gameassembly.dll, unityplayer.dll | RustClient_Data |
| L4D2 | left4dead2.exe | | |
| DoD | dod.exe | hw.dll, sw.dll, dsound.dll, opengl32.dll, wdmaud.drv | valve |
| TFC | tfc.exe | hw.dll, sw.dll, dsound.dll, opengl32.dll, wdmaud.drv | valve |
| Sven Co-op | svencoop.exe | hw.dll, sw.dll, dsound.dll, opengl32.dll, wdmaud.drv | svencoop |
| Xash | xash.exe | hw.dll, sw.dll, dsound.dll, opengl32.dll, wdmaud.drv | |

Фильтры файлов: `*.mix`, `*.flt`, `*.m3d`

## Белый список процессов

steam.exe, audiodg.exe, svchost.exe, csrss.exe, lsass.exe, conhost.exe, cmd.exe, nvcontainer.exe, crashpad_handler.exe, AMDRSServ.exe, RadeonSoftware.exe, Discord.exe, Overwolf.exe, OverwolfHelper.exe, OverwolfHelper64.exe, CurseForge.exe, ProcessLasso.exe, ProcessGovernor.exe, PresentMon-x64.exe, PresentMon_x64.exe, NahimicSvc64.exe, NahimicSvc32.exe, EarTrumpet.exe, SECOCL64.exe, WarGods Cheat Defender Win7/8/XP.exe, iot_driver_v190.exe, anti_ransomware_service.exe, GameManagerService.exe, GameManagerService3.exe, KillerNetworkService.exe, RAVBg64.exe, AppMonitorPlugIn.exe, NGenuity2Helper.exe, RtkAudUService64.exe, ksdeui.exe

## Подсистемы сканирования

### Перечисление процессов
`CreateToolhelp32Snapshot` + `Process32FirstW/NextW`, потом `K32EnumProcessModulesEx` + `K32GetModuleFileNameExW`. Три прохода: все процессы, все модули, модули конкретно игрового процесса. Для каждого модуля: путь, размер, хеш, цифровая подпись.

### Сканер виртуальной памяти (`sub_7B25E0`)
Обходит всё адресное пространство через `VirtualQueryEx`. Пропускает `MEM_FREE`. Разбивает большие регионы на чанки с **перекрытием 1024 байт** на границах. Читает через `ReadProcessMemory`. Помечает инжекции как `Injected_0x%I64x`. (Представили ебало типа который запустил это с VAC? Когда у типа даже нету базовых шифрований обращение к процессу и памяти...)

### Детект inline-хуков (`sub_7BEAD0`)
Идёт по коду игры с шагом `sizeof(pointer)`. Для каждого значения в диапазоне `0x10000`-`0x7FFFFFFEFFFF` вызывает `VirtualQueryEx` для проверки `PAGE_EXECUTE*`. Если да - читает 6 байт перед адресом через `ReadProcessMemory`. Флагует как хук если `prefix[0]==0xFF` или `prefix[1]==0xE8` или `prefix[4]==0xFF` (паттерны JMP/CALL трамплинов). Записывает `hooked`/`hookedaddr` в отчёт. 

### Менеджер аппаратных брейкпоинтов (`sub_7BB380`)
`GetThreadContext`/`Wow64GetThreadContext` для чтения DR0-DR3/DR7. Ищет свободный слот по битам Local Enable (0, 2, 4, 6) в DR7. Записывает адрес, конфигурирует тип/длину, применяет через `SetThreadContext`, перечитывает для верификации. Поддержка x64 и WOW64.

### Движок самоотладки (`sub_7BCDF0`)
```
DebugActiveProcess(game_PID)         // занимает единственный слот отладчика
DebugSetProcessKillOnExit(FALSE)     // игра переживёт краш ECD

while running:
    WaitForDebugEvent(timeout=100ms)
    switch event:
        EXCEPTION_BREAKPOINT  -> сканирует хуки, восстанавливает байт, ставит TF
        SINGLE_STEP           -> обрабатывает HW BP, переставляет INT3/HW BP
        ACCESS_VIOLATION      -> анализ защищённых страниц
        CREATE_THREAD         -> ставит HW BP на новый поток
    CheckRemoteDebuggerPresent()     // детект отсоединения
```

### Проверка драйверов и сервисов (`sub_765F60`)
`OpenSCManagerW` -> `OpenServiceW` -> `QueryServiceStatusEx` -> проверяет `dwCurrentState == 4` (SERVICE_RUNNING). Цели: `acdrv` (драйвер nProtect), `ggsvc` (сервис nProtect). Резолв NT device paths через `QueryDosDeviceW` (перебор дисков A-Z).

### Сканирование реестра (2 прохода)
| Ключ | Назначение |
|------|-----------|
| `HKLM\SYSTEM\ControlSet001\Services\bam\State\UserSettings` | История запуска EXE через BAM (сохраняется после удаления файла) |
| `HKCU\Software\Microsoft\DirectInput\` | История устройств ввода |
| `HKCU\...\Explorer\FeatureUsage\AppSwitched` | История Alt-Tab |
| `HKCR\Local Settings\...\Shell\MuiCache` | Кэш имён запускавшихся приложений |
| `HKCU\Software\Valve\Steam\ActiveProcess` | Информация о процессе Steam |
| `HKCU\...\SteamPath` | Путь установки Steam |

Фильтры: `.exe`, `.bat`, `.cmd`. Также делает снимки реестра (`SaveRegSnapshotToFile`) и сравнивает их (`CompareRegWithFile`) для обнаружения изменений.

### Браузерная история
Копирует БД во временную папку, открывает через SQLite:

**Chrome** (`History`):
```sql
SELECT current_path, referrer, tab_url, start_time FROM downloads
```

**Firefox** (`places.sqlite`):
```sql
SELECT p.url, a.content, a.lastModified
FROM moz_annos AS a JOIN moz_places AS p ON a.place_id = p.id
WHERE a.anno_attribute_id = (
  SELECT id FROM moz_anno_attributes WHERE name = 'downloads/destinationFileURI'
)
```

### Анализ игровых файлов
- Фейковые спрайты: `valve/sprites/muzzIeflash5.spr`
- Regex для обфусцированных имён: `.*?\\[a-z][a-z]t[a-z]s[a-z]h[a-z]e[a-z]d[a-z][a-z]$`
- Проверка секции `.text` в `engine.dll` на патчи (расшифровано из `sub_5FCD10`/`sub_5FCF00`)
- Проверка хука `glVertex3fv` (wallhack в CS 1.6, расшифровано из `sub_60B080`)
- Рекурсивное хеширование `cstrike/`, `valve/`, `platform/`

### Сетевой мониторинг
`GetExtendedTcpTable`/`GetExtendedUdpTable` (IPHLPAPI) с привязкой к PID. Параллельная проверка IP (`checkIPsParallel` из RTTI). Фильтрация `0.0.0.0`/`127.0.0.1`. Проверка именованных каналов (`\\.\pipe\`).

### Прочие проверки
- **Буфер обмена**: `OpenClipboard` -> `CF_HDROP` -> `DragQueryFileW` (до 512 файлов)
- **Самоудаляющиеся процессы**: поиск `exe.todel1`, `.delete-later-` в `%TEMP%`
- **Цифровые подписи**: `WinVerifyTrust` + `CertGetCertificateChain` для каждого модуля (Который кстати не будет работать, если у типа кастм винда)
- **Детект DLL-инъекций** (`sub_764070`): `GetModuleHandleW` + `GetProcAddress` с зашифрованными именами
- **Поиск строк в памяти**: `StringSearcher::Search` (из RTTI)
- **Аттестация драйверов**: "Windows Hardware Driver Attested Verification"
- **Проверка родительского процесса** (`sub_741430`): кто запустил ECD

## База данных читов (ECD_DB)

SQLite 3, указатель `dword_BB35F0`. 4 уровня вложенности:
```
Игра -> Процесс -> Модуль/DLL -> Сигнатура (хеш, паттерн, путь)
```

Каждая запись 24 байта, зашифрована. 100+ уникальных функций-дешифраторов (`sub_72C390`-`sub_736120`).

**Движок матчинга** (`sub_722550`):
1. Расшифровывает путь из записи БД
2. Проверяет существование файла через `sub_6FFC60` (`GetFileAttributesW`, при `ACCESS_DENIED`/`SHARING_VIOLATION` тоже считает что файл есть, fallback `CreateFileW` + `Wow64DisableWow64FsRedirection`)
3. Хеширует содержимое файла
4. Сравнивает хеш с эталоном из БД
5. При совпадении добавляет в результаты

**Известные детекты**: orpheu_amxx.dll, NextClient, GSClient/GSClient+, particleman.dll, demoheader.dmf, safemon.dll (360 Total Security, выгружается через `FreeLibrary`....)

**ID категорий детекции** (из анализа контекста xrefs):
- `32891` = сканирование antishield (regex `.*antishield.*` -> "path" -> "32891" -> "started")
- `33625` = обход драйвера GameGuard ("gameguard driver" -> "33625" -> "injected" -> "bypassed" -> "system_country")

## Самозащита

**CRC32** (3 слота): `.text`, `.rdata`, overlay. При несовпадении: "[N] Failed to compare crc32..."

**Anti-tamper** (7 точек: `0x581AF0`, `0x5D5D30`, `0x5D8930`, `0x5F6070`, `0x6217E0`, `0x6310E0`, `0x6357B0`):
1. `SetThreadPriority(THREAD_PRIORITY_HIGHEST)`
2. Рандомизированный цикл `rand()%50+15` итераций
3. Генерация мусора на стеке (anti-dump)
4. Хеширование + побайтовое сравнение
5. Замер времени: дельта `GetTickCount()` 2000-40000мс -> `byte_B50F3D = 1`

Сообщение при срабатывании: "MENYA LOMAT' SUKING HACKER!" (А кому сейчас легко??)

**Anti-suspend watchdog** (`sub_647160`): поток с приоритетом LOWEST следит за приостановкой. Эскалация: ABOVE_NORMAL -> HIGH_PRIORITY -> REALTIME.

**Пасхалки** (не используются в логике):
- "THIS CUTE BEAST CAN PROTECT ANTIHACK EXECUTABLE FROM BAD HACKERS" (мёртвая переменная в WinMain)
- "THIS ADORABLE THE CUTEST BEAST DEFENDS YOUR ANTIHACK EXECUTABLE FROM CYBER VILLAINS!" (маркер в MARK)

## Сеть и отчёты

**Серверы**: `https://fungun.net/ecd/` (основной), `https://fungun.top/` (зеркало), `https://fungun.net/ecd/list/` (отчёты), `https://fungun.net/ecd/faq/` (Ну и прост FaQ)

**HTTP стек**: cpp-httplib 0.16.3 поверх OpenSSL. User-Agent `cpp-httplib/0.16.3`. Digest auth до 5 попыток. Поддержка прокси. Follow redirects.

**API ключ**: `xWey5BsRh1e36` (расшифрован из `sub_63B390`/`sub_63F820` через `sub_662A20`). Вставляется как часть URL пути при запросах обновлений БД через `sub_75C8F0`.

**Формат отчёта** (JSON через tao::json):
```json
{
  "injected":   "[list] ReadProcessMemory + VirtualQueryEx",
  "loaded":     "[list] K32EnumProcessModulesEx",
  "started":    "[list] CreateToolhelp32Snapshot",
  "installed":  "[list] registry BAM/MuiCache/AppSwitched",
  "downloaded": "[list] Chrome/Firefox SQLite",
  "changer":    "[list] file hash mismatches vs ECD_DB",
  "hooked":     "bool",
  "hookedaddr": "address",
  "gamefile":   "process name",
  "demopath":   "demo file path",
  "path":       "[list] suspicious paths",
  "browser":    "[list] download URLs",
  "trash_path": "[list] recycle bin findings",
  "time":       "timestamp"
}
```

**Уровни серьёзности**: `"Potencial"` (из контекста xrefs: `founds_diagnostic` + `scan_memreg_name`, неподтверждённые находки в памяти)

**Оффлайн режим**: сохраняет `%Y%m%d_%H%M%S.ecd` с ключом `xWey5BsRh1e36` (xrefs `0x5D26D7`, `0x5D281D`, `0x5D3089` последовательно в `sub_58AD50`)

## Установка сертификата (`sub_764AB0`)

```
1. FindResourceW(#111, RT_RCDATA)      // PFX from PE resources
2. sub_748B50()                         // build obfuscated password
3. PFXImportCertStore(&pPFX, pwd, 1)
4. CertOpenStore(CERT_STORE_PROV_SYSTEM, flags, L"Root")
   admin:  CERT_SYSTEM_STORE_LOCAL_MACHINE  (0x20000)
   user:   CERT_SYSTEM_STORE_CURRENT_USER   (0x10000)
5. CertAddCertificateContextToStore(hStore, pCert, REPLACE_EXISTING)
6. Registry: rootcert2=0x01 or usercert2=0x01
```

Ставит свой корневой CA в Trusted Root Certification Authorities.

## WMI-запросы (all via ROOT\CIMV2)
(использовать WMI в 2025-2026 году для HWID, это прям СИЛЬНО... ведь не обойдёт не какой дефолт free spoofer xd)
| Function | Query | Purpose |
|----------|-------|---------|
| sub_6C0810 | `SELECT * FROM Win32_SystemDriver` | Cheat driver detection or AC |
| sub_7434A0 | `SELECT Name FROM Win32_ComputerSystem` | Machine name |
| sub_7434A0 | `SELECT Version,Name FROM Win32_OperatingSystem` | OS info |
| sub_74B530 | `SELECT Model,SerialNumber FROM Win32_DiskDrive` | Disk fingerprint |
| sub_74BE10 | `SELECT UniqueId,ProcessorId,Name FROM Win32_Processor` | CPU fingerprint |
| sub_74CA60 | `SELECT Manufacturer,SerialNumber FROM Win32_BIOS` | BIOS fingerprint |
| sub_74D350 | `SELECT Product,Model,Manufacturer,SerialNumber FROM Win32_BaseBoard` | Motherboard fingerprint |

Используется для привязки бана к железу (CPU + диск + BIOS + материнская плата).

## Взаимодействие с GameGuard

Имена сервисов (расшифровал вручную):

| Function | Key | Result |
|----------|-----|--------|
| sub_651680 | 0x6DFDB3 | acdrv |
| sub_651720 | 0x6DFDB3 | 'acdrv' |
| sub_6517D0 | 0x6DFDB3 | ggsvc |
| sub_651870 | 0x6DFDBB | 'acdrv' |
| sub_651920 | 0x6DFDBB | acdrv |
| sub_6519C0 | 0x6DFDBB | ggsvc |

Два рандомизированных пути проверки через `GetTickCount()%1000 <= 50` (5%/95%). Тройная проверка целостности/подписи драйвера, инкрементирует `dword_B50EFC` (0-3).

## Ключевые функции

| Address | Description |
|---------|-------------|
| 0x58AD50 | Main orchestrator (all scan + report logic) |
| 0x722550 | Cheat DB matching engine |
| 0x56BC10 | Filesystem scanner |
| 0x75C8F0 | HTTP download manager |
| 0x654110 | WinMain |
| 0x64FB00 | Startup wrapper (mutex, HTTP init, scan launch) |
| 0x7BCDF0 | Self-debugging engine |
| 0x7BEAD0 | Inline hook scanner |
| 0x7B25E0 | Memory region scanner |
| 0x7BB380 | HW breakpoint manager |
| 0x7BD420 | Debug exception dispatcher |
| 0x765F60 | Service status checker |
| 0x764AB0 | Certificate installer |
| 0x767AC0 | Firewall rule creator |
| 0x766070 | Clipboard checker |
| 0x6FFC60 | File existence checker |
| 0x755000 | PE parser + network monitor |
| 0x7B1230 | PE header parser (.NET CLR detection) |
| 0x456A50 | JSON field mapper (enum -> key) |
| 0x754720 | Random token generator |

## Глобальные переменные

| Address | Type | Description |
|---------|------|-------------|
| dword_BB35F0 | ptr | Cheat database |
| dword_BB3538 | ptr | File check cache |
| dword_B50EBC | int | Match counter |
| dword_B50EC0 | int | Checked files counter |
| byte_B50F3D | bool | Tamper detected flag |
| byte_B50E85 | bool | GameGuard detected flag |
| dword_B50EFC | int | GameGuard integrity score (0-3) |

## NT API (via blackbone)

NtReadVirtualMemory, NtWriteVirtualMemory, NtAllocateVirtualMemory, NtFreeVirtualMemory, NtProtectVirtualMemory, NtMapViewOfSection, NtUnmapViewOfSection, NtQueryVirtualMemory, NtQuerySystemInformation, NtQueryObject, NtDuplicateObject, NtSuspendProcess, NtResumeProcess, NtSetInformationProcess, NtQueryInformationProcess, NtQueryInformationThread, NtGetContextThread, NtQueueApcThread, NtWow64ReadVirtualMemory64, NtWow64WriteVirtualMemory64, NtWow64QueryInformationProcess64, LdrLoadDll, LdrUnloadDll, LdrGetProcedureAddress, RtlQueueApcWow64Thread, NtYieldExecution, NtQuerySection, RtlGetVersion

## asmjit (статически слинкован, 0x7C0000-0x7F0000)

`X86CpuUtil::detect` (0x7EC556, 5026 байт), `CodeGen`, `X86Assembler`, `JitRuntime::add`, `VMemUtil::allocProcessMemory/releaseProcessMemory`. Генерация динамического машинного кода в runtime, вызывается из `sub_58AD50`.

## Маркеры сборки

```
EASYCHEATDETECTOR_MARK__12350
EASYCHEATDETECTOR_MARK__12399
EASYCHEATDETECTOR_MARK__FUNC12350
EASYCHEATDETECTOR_MARK__FUNC12399
```

## Ну и что я ещё не доделал, это 

есть ещё строка по адресу `0xad4eb0`: `G7@qZ!xP3#tL9$wF8^mR2&jH6*eD1%vT4@kN5!sB0#yQ3^uX8$zC7&lM2*tJ9^oF1!pR6#hV4@wT5%jK`. Единственный xref `0x5baa22` в `sub_58AD50`, рядом со строками `hooked`/`hookedaddr`/`error_exception`. Мне уже далее стало в падлу, по этому если нужно, далее сами. 
