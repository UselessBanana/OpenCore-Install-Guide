# Bulldozer(15h) y Jaguar(16h)

| Soporte | Versión |
| :--- | :--- |
| Versión de OpenCore soportada | 0.6.8 |
| Soporte inicial de macOS | macOS 10.13, High Sierra |

## Punto de partida

Hacer un config.plist puede parecer difícil, pero en realidad no lo es. Sólo toma tiempo, pero esta guía te dirá cómo configurar todo, no dejará espacios sin cubrir. Esto también significa que si tienes problemas, asegúrate de que todos los espacios de tu config estén correctos, respaldándote con esta guía. Principales cosas a tener en cuenta con OpenCore:

* **Todas las propiedades deben ser definidas**, no hay opciones predeterminadas para OpenCore, así que **no elimines secciones a menos que se te indique específicamente**. Si la guía no menciona la opción, déjala como está.
* **El sample.plist no puede ser utilizado como está**, debes configurarlo para tu sistema
* **NO USES CONFIGURADORES**, éstos raramente respetan la configuración de OpenCore y algunos como los de Mackie agregarán propiedades de Clover y generarán plists corruptas!
Ahora con todo esto, haremos un pequeño recordatorio de todo lo que necesitamos:

* [ProperTree](https://github.com/corpnewt/ProperTree)
  * Editor de plists universal
* [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
  * Para generar datos de SMBIOS
* [Sample/config.plist](https://github.com/acidanthera/OpenCorePkg/releases)
  * Dirígete a la sección anterior sobre cómo obtenerla: [Setup de tu config.plist](/config.plist/README.md)
* [Parches del kernel de AMD](https://github.com/AMD-OSX/AMD_Vanilla/tree/opencore)
  * Necesario para arrancar macOS en hardware AMD (guarda estos para luego, cubriremos cómo usarlos abajo)
  * [Bulldozer/Jaguar(15h/16h)](https://github.com/AMD-OSX/AMD_Vanilla/tree/opencore/15h_16h) (Soporta 10.13, 10.14, y 10.15)

**Lee esta guía más de una vez antes de configurar OpenCore y asegúrate de tenerlo configurado correctamente. Ten en cuenta que las imágenes no siempre serán las más actualizadas, así que lee el texto debajo de ellas, si no se menciona nada, déjalo como predeterminado.**

## ACPI

![ACPI](../images/config/AMD/acpi-fx.png)

### Add

::: tip Info

Aquí es donde agregarás los SSDTs para su sistema, estos son muy importantes para **arrancar macOS** y tienen muchos usos como [hacer USB mapping](https://dortania.github.io/OpenCore-Post-Install/usb/), [deshabilitar GPUs no compatibles](/extras/spoof.md) y demás. Y con nuestro sistema, **incluso es necesario  para bootear**. Guía sobre cómo encontrarlos aquí: [**Comenzando con ACPI**](https://dortania.github.io/Getting-Started-With-ACPI/)

| SSDTs necesarios | Descripción |
| :--- | :--- |
| **[SSDT-EC-USBX](https://dortania.github.io/Getting-Started-With-ACPI/)** | Arregla el controlador integrado (Embeeded Controller o EC en inglés) y la energía de los USBs. Dirígete a [Comenzando con ACPI](https://dortania.github.io/Getting-Started-With-ACPI/) para más información. |

Ten en cuenta que **no debes** agregar tu `DSDT.aml` generado aquí, ya está en tu firmware. Entonces, si está presente, elimina la entrada correspondiente en tu `config.plist` y en EFI/OC/ACPI.

Aquellos que deseen una explicación más profunda de cómo hacer un dump de su DSDT, y de cómo hacer estos SSDTs y compilarlos, consulten [**Comenzando con ACPI**](https://dortania.github.io/Getting-Started-With-ACPI/). Los SSDTs compilados tienen una extensión **.aml** (compilados) e irán a la carpeta `EFI/OC/ACPI` y **debe** especificarse en tu configuración en `ACPI -> Add` también.

:::

### Delete

Esto bloquea la carga de ciertas tablas ACPI, en nuestro caso podemos ignorar esto.

### Patch

Esta sección nos permite modificar dinámicamente partes del ACPI (DSDT, SSDTs, etc.) a través de OpenCore. En nuestro caso, los parches son manejados por nuestros SSDTs. Esta es una solución mucho más limpia, ya que nos permitirá iniciar Windows y otros sistemas operativos con OpenCore

### Quirks

Configuración relacionada con ACPI: deja todo aquí como predeterminado, ya que no tenemos uso para estas quirks.

## Booter

![Booter](../images/config/config-universal/aptio-iv-booter.png)

Esta sección está dedicada a los quirks relacionados con el parche del boot.efi con OpenRuntime, el reemplazo de AptioMemoryFix.efi

### MmioWhitelist

Esta sección permite pasar dispositivos a macOS que generalmente se ignoran. Esto puede ser útil cuando es emparejado con `DevirtualiseMmio`

### Quirks

::: tip Información
Configuraciones relacionadas a el parcheo de boot.efi y arreglos en el firmware. En nuestro caso dejarlo como predeterminado
:::
::: details Información más detallada

* **AvoidRuntimeDefrag**: YES
  * Corrige runtime services de UEFI como fecha, hora, NVRAM, control de energía, etc.
* **EnableSafeModeSlide**: YES
   * Habilita las variables de Slide en el arranque seguro
* **EnableWriteUnprotector**: YES
  * Necesario para eliminar la protección de escritura del registro CRO.
* **SetupVirtualMap**: YES
  * Corrige las llamadas de `SetVirtualAddresses` a `virtual addresses`, requerido para resolver kernel panics tempranos en placas madre Gigabyte
* **ProvideCustomSlide**: YES
  * Usado para el cálculo de la variable Slide. Sin embargo la necesidad de este es determinada por el mensaje `OCABC: Only N/256 slide values are usable!` en tu registro de depuración. Si aparece el mensaje `OCABC: All slides are usable! You can disable ProvideCustomSlide!` en tu registro, puedes deshabilitar `ProvideCustomSlide`.
  
:::

## DeviceProperties

![DeviceProperties](../images/config/config-universal/DP-no-igpu.png)

### Add

Configura las device properties desde un mapa.

Por defecto, el Sample.plist tiene esta sección configurada para la GPU integrada y el audio. En nuestro caso no tenemos un iGPU, por lo que el PciRoot `PciRoot(0x0)/Pci(0x2,0x0)` puede ser eliminado de la sección `Add`. Para el audio, estaremos configurando el layout en la sección de los boot args, por lo que la eliminación de `PciRoot(0x0)/Pci(0x1b,0x0)` es recomendada para las secciones `Add` y `Block` 

Si no leiste todo esto: Elimina todos los PciRoots aquí ya que no utilizaremos esta sección.

### Delete

Elimina las propiedades del dispositivo del mapa. En nuestro caso podemos ignorar esto.

## Kernel

| Kernel | Parches del kernel |
| :--- | :--- |
| ![Kernel](../images/config/AMD/kernel.png) | ![](../images/config/AMD/kernel-patch.png) |

### Add

Aquí es donde especificamos qué kexts cargar, en qué orden y para qué arquitectura es el kext en cuestión. Por defecto recomendamos que dejes lo que hace ProperTree, pero si tienes una CPU de 32 bits presta atención a la información de abajo:

::: details Información más detallada

Lo principal que debes tener en cuenta es:

* El orden de carga
  * Recuerda que cualquier plugin debe ser cargado *luego* del kext principal.
  * Esto significa que kexts como Lilu **deben** ir cargados antes de VirtualSMC, AppleALC, WhateverGreen, etc. 

Un dato a tener en cuenta es que los usuarios de [ProperTree](https://github.com/corpnewt/ProperTree) pueden ejecutar **Cmd/Ctrl+Shift+R** para agregar todos sus kexts en el orden correcto sin escribir manualmente cada kext.

* **Arch**
  * Las arquitecturas soportadas por este kextArchitectures supported by this kext
  * Los valores soportados actualmente son `Any`, `i386`(32-bits) y `x86_64`(64-bits)
* **BundlePath**
  * Nombre del kext
  * por ejemplo: `Lilu.kext`
* **Enabled**
  * Se explica por sí solo, define si el kext está habilitado o no.
* **ExecutablePath**
  * La ruta al ejecutable real está oculta dentro del kext, puedes ver qué ruta tiene tu kext haciendo clic derecho y seleccionando `Mostrar contenido del paquete`. En general, será `Contents/MacOS/Kext` pero algunos tienen kexts ocultos dentro de la carpeta `Plugin`. Ten en cuenta que los kexts con sólo plists no necesitan que esto sea completado.
  * ejemplo: `Contents/MacOS/Lilu`
* **MinKernel**
  * La versión más baja del kernel en la que se inyectará tu kext, mira la tabla de abajo para ver valores posibles.
  * Ejemplo: `12.00.00` para OS X 10.8
* **MaxKernel**
  * La versión más alta del kernel en la que se inyectará tu kext, mira la tabla de abajo para ver valores posibles
  * Ejemplo: `11.99.99` para OS X 10.7
* **PlistPath**
  * Ruta al `info.plist`, oculto dentro del kext
  * ejemplo: `Contents/Info.plist`

::: details Tabla de soporte del Kernel

| Versión de OS X | MinKernel | MaxKernel |
| :--- | :--- | :--- |
| 10.4 | 8.0.0 | 8.99.99 |
| 10.5 | 9.0.0 | 9.99.99 |
| 10.6 | 10.0.0 | 10.99.99 |
| 10.7 | 11.0.0 | 11.99.99 |
| 10.8 | 12.0.0 | 12.99.99 |
| 10.9 | 13.0.0 | 13.99.99 |
| 10.10 | 14.0.0 | 14.99.99 |
| 10.11 | 15.0.0 | 15.99.99 |
| 10.12 | 16.0.0 | 16.99.99 |
| 10.13 | 17.0.0 | 17.99.99 |
| 10.14 | 18.0.0 | 18.99.99 |
| 10.15 | 19.0.0 | 19.99.99 |
| 11 | 20.0.0 | 20.99.99 |

:::

### Emulate

::: tip Info

Necesario para falsificar CPUs no soportadas como Pentiums y Celeros y para deshabilitar la administración de energía de la CPU en procesadores no soportados (como los de AMD)

| Quirk | Habilitado |
| :--- | :--- |
| DummyPowerManagement | YES |

:::

::: details Información más detallada

* **CpuidMask**: Deja esto en blanco
  * Máscara para CPUID falsa
* **CpuidData**: Deja esto en blanco
  * Entrada falsa del CPUID
* **DummyPowerManagement**: YES
  * Nueva alternativa a NullCPUPowerManagement; requerido para todos los sistemas basados en CPUS AMD debido a que no hay administración de energía nativo. Intel puede ignorar esto. 
* **MinKernel**: Deja esto en blanco
  * La versión más baja del kernel en la que se inyectará tu kext, mira la tabla de abajo para ver valores posibles.
  * Ejemplo: `12.00.00` para OS X 10.8
* **MaxKernel**: Deja esto en blanco
  * La versión más alta del kernel en la que se inyectará tu kext, mira la tabla de abajo para ver valores posibles
  * Ejemplo: `11.99.99` para OS X 10.7

::: details Tabla de soporte del kernel

| Versión de OS X | MinKernel | MaxKernel |
| :--- | :--- | :--- |
| 10.4 | 8.0.0 | 8.99.99 |
| 10.5 | 9.0.0 | 9.99.99 |
| 10.6 | 10.0.0 | 10.99.99 |
| 10.7 | 11.0.0 | 11.99.99 |
| 10.8 | 12.0.0 | 12.99.99 |
| 10.9 | 13.0.0 | 13.99.99 |
| 10.10 | 14.0.0 | 14.99.99 |
| 10.11 | 15.0.0 | 15.99.99 |
| 10.12 | 16.0.0 | 16.99.99 |
| 10.13 | 17.0.0 | 17.99.99 |
| 10.14 | 18.0.0 | 18.99.99 |
| 10.15 | 19.0.0 | 19.99.99 |
| 11 | 20.0.0 | 20.99.99 |

:::

### Force

Utilizado para cargar kexts desde el volumen del sistema, esto es únicamente relevante para sistemas operativos más antiguos donde ciertos kexts no están presentes en el cache (como IONetworkingFamily en 10.6).

En nuestro caso podemos ignorar esto. 

### Block

Bloquea la carga de ciertos kexts. No es relevante para nosotros.

### Patch

Aquí es cuando sucede la magia de los parches del kernel para AMD. Por favor ten en cuenta que `KernelToPatch` y `MatchOS` de Clover se convierten en `Kernel` y `MinKernel`/ `MaxKernel` en OpenCore. Puedes encontrar parches pre hechos por [AlGrey](https://amd-osx.com/forum/memberlist.php?mode=viewprofile&u=10918&sid=e0feb8a14a97be482d2fd68dbc268f97)(algrey#9303).

Parches del kernel:

* [Bulldozer/Jaguar(15h/16h)](https://github.com/AMD-OSX/AMD_Vanilla/tree/opencore/15h_16h) (10.13, 10.14, y 10.15)

Para juntarlos:

* Abre ambos archivos,
* Elimina la sección `Kernel -> Patch` de tu config.plist
* Copia la sección `Kernel -> Patch` de patches.plist
* Pégalos en el lugar donde estaban los parches antiguos en tu config.plist

![](../images/config/AMD/kernel.gif)

### Quirks

::: tip Info

Ajustes relacionados a el kernel, en nuestro caso habilitaremos lo siguiente:

| Quirk | Habilitado |
| :--- | :--- |
| PanicNoKextDump | YES |
| PowerTimeoutKernelPanic | YES |
| XhciPortLimit | YES |

:::

::: details Información más detallada

* **AppleCpuPmCfgLock**: NO
  * Solo es necesario cuando CFG-Lock no se puede deshabilitar en BIOS, la contraparte de Clover sería AppleIntelCPUPM. Los usuarios de AMD pueden ignorar esto.

* **AppleXcpmCfgLock**: NO
  * Solo es necesario cuando CFG-Lock no se puede deshabilitar en BIOS, la contraparte de Clover sería KernelPM. Los usuarios de AMD pueden ignorar esto.

* **AppleXcpmExtraMsrs**: NO
  * Deshabilita múltiples accesos MSR necesarios para ciertas CPUs no soportadas como Pentiums y Xeons.

* **CustomSMBIOSGuid**: NO
  * Hace parches de GUID para el modo `Custom` de UpdateSMBIOSMode. Usualmente relevante para laptops Dell
  * La habilitación de este quirk con el modo `Custom` de UpdateSMBIOSMode también puede deshabilitar la inyección de SMBIOS en sistemas operativos que no son de Apple, aunque no recomendamos esto ya que rompe la compatibilildad con BootCamp. Úsalo bajo tu propio riesgo; debe ser utilizado en conjunto con `PlatformInfo -> UpdateSMBIOSMode -> Custom`.

* **DisableIoMapper**: NO
  * AMD no soporta DMAR o VT-D, por lo que es irrelevante.

* **DisableLinkeditJettison**: YES
  * Permite que Lilu y otros tengan rendimiento más estable sin el quirk `keepsyms=1`

* **DisableRtcChecksum**: NO
  * Impide que AppleRTC escriba en la suma de comprobación primaria (0x58-0x59), necesaria para los usuarios que reciben el reinicio del BIOS o son enviados al modo seguro después del reinicio/apagado.

* **ExtendBTFeatureFlags** NO
  * Útil para aquellos que tengan problemas con continuity con tarjetas que no sean de Apple/Fenvi.

* **LapicKernelPanic**: NO
  * Inhabilita el kernel panic en la interrupción del kernel de AP, generalmente necesaria para los sistemas HP. El equivalente de Clover es `Kernel LAPIC`.

* **LegacyCommpage**: NO
  * Resuelve el requerimiento de SSS3 para CPUS de 64 bits en macOS, por lo que es principalmente relevanete para CPUs como Pentium 4 de 64 bits (como Prescott).

* **PanicNoKextDump**: YES
  * Permite leer los registros de kernel panics cuando éstos ocurren.

* **PowerTimeoutKernelPanic**: YES
  * Ayuda a solucionar kernel panics relacionados con los cambios de energía con los drivers de Apple en macOS Catalina, especialmente con audio digital.

* **SetApfsTrimTimeout**: `-1`
  * Establece el tiempo de espera en microsegundos para el sistema de archivos APFS en SSDs, únicamente aplicable para macOS 10.14 y posterior con SSDs problemáticas.

* **XhciPortLimit**: YES
  * Este es en realidad el parche de límite de 15 puertos, no confíes en él, ya que no es una solución garantizada para reparar USBs. Una mejor solución para AMD puede ser encontrada aquí: [USB Mapping para AMD](https://inyextciones.github.io/OpenCore-Post-Install/usb/)

:::

### Scheme

Configuraciones relacionadas al arranque legacy (10.4-10.6). La mayoría de los usuarios pueden saltearse esto, pero si planeas en arrancar versiones legacy puedes ver la información abajo:

::: details Información más detallada

* **FuzzyMatch**: True
  * Usado para ignorar sumas de comprobación (checksums) con kernelcache, optando con el último cache disponible. Puede ayudar a mejorar el rendimiendo en muchas computadoras en 10.6
* **KernelArch**: x86_64
  * Configura la arquitectura del kernel; puedes elegir entre `Auto`, `i386` (32-bit), y `x86_64` (64-bit).
  * Si estás planeando en arrancar versiones antiguas que requieren un kernel de 32 bits (como 10.4 y 10.5), recomendamos que lo configures a `Auto` y dejes que macOS decida basado en tu SMBIOS. Mira la tabla de abajo para ver los valores soportados
    * 10.4-10.5 — `x86_64`, `i386` o `i386-user32`
      * `i386-user32` refiere a el espacio de usuario de 32 bits, por lo que las CPUs de 32 bits deben usar esto (o CPUs que no tengan SSSE3)(or CPUs missing SSSE3)
      * `x86_64` tendrá un espacio de kernel de 32 bits, pero asegurará un espacio de usuario de 64 bits en 10.4/5
    * 10.6 — `i386`, `i386-user32`, o `x86_64`
    * 10.7 — `i386` o `x86_64`
    * 10.8 y posterior — `x86_64`

* **KernelCache**: Auto
  * Configura el tipo de cache del kernel, principalmente relevante para depuración, por lo que recomendamos `Auto` para obtener el mejor soporte.

:::

## Misc

![Misc](../images/config/config-universal/misc.png)

### Boot

Configuración para la pantalla de inicio (Deja todo como predeterminado).

### Debug

::: tip Info

Útil para depurar problemas de arranque de OpenCore (cambiaremos todo *menos* `DisplayDelay`):

| Quirk | Enabled |
| :--- | :--- |
| AppleDebug | YES |
| ApplePanic | YES |
| DisableWatchDog | YES |
| Target | 67 |

:::

::: details Información más detallada

* **AppleDebug**: YES
  * Habilita el logging de boot.efi. Esto es útil para hacer depuración. Ten en cuenta que esto es soportado en 10.15.4 y posterior.
* **ApplePanic**: YES
  * Intenta registrar kernel panics en el disco
* **DisableWatchDog**: YES
  * Deshabilita el watchdog UEFI, puede ayudar con problemas de arranque temprano
* **DisplayLevel**: `2147483650`
  * Muestra aún más información de depuración, requiere la versión DEBUG de OpenCore
* **SerialInit**: NO
  * Necesario para configurar salida de seriales con OpenCore
* **SysReport**: NO
  * Útil para depurar y otros aspectos como volcar tablas ACPI
  * Ten en cuenta que esto es limitado a las versiones DEBUG de OpenCore.
* **Target**: `67`
  * Muestra más información de depuración, requiere la versión de depuración de OpenCore

These values are based of those calculated in [OpenCore debugging](../troubleshooting/debug.md)

:::

### Security

::: tip Info

Security se explica por sí sola, **no te lo saltes**. Vamos a cambiar lo siguiente:

| Quirk | Habilitado | Comentario |
| :--- | :--- | :--- |
| AllowNvramReset | YES | |
| AllowSetDefault | YES | |
| BlacklistAppleUpdate | YES | |
| ScanPolicy | 0 | |
| SecureBootModel | Default |  Esta es una palabra la cual distingue entre mayúsculas y minúsculas, configúrala a `Disabled` si no quieres arranque seguro (por ejemplo en el caso de que requieras los Web Drivers de Nvidia) |
| Vault | Optional | Esta es una palabra, no es opcional omitir esta configuración. Lo lamentarás si no lo configuras en `Optional`, ten en cuenta que distingue entre mayúsculas y minúsculas |

:::

::: details More in-depth Info

* **AllowNvramReset**: YES
  * Permite restablecer NVRAM tanto en el selector de arranque como al presionar `Cmd+Opt+P+R`

* **AllowSetDefault**: YES
  * Permite que `CTRL+Enter` y `CTRL+Index` configuren el dispositivo de arranque predeterminado en el selector

* **ApECID**: 0
  * Usado para compensar identificadores de arranque seguro, actualmente este quirk es faltoso debido a un bug en el instalador de macOS así que te recomendamos que lo dejes como está por defecto.

* **AuthRestart**: NO
  * Habilita el reinicio autenticado para FileVault 2, por lo que no se requiere contraseña al reiniciar. Puede considerarse un riesgo de seguridad así que es opcional

* **BlacklistAppleUpdate**: YES
  * Utilizado para bloquear actualizaciones del firmware como un nivel extra de protección ya que macOS Big Sur dejo de utilizar la variable `run-efi-updater`

* **DmgLoading**: Signed
  * Asegura la carga únicamente de DMGs firmados

* **ExposeSensitiveData**: `6`
  * Muestra más información de depuración, requiere la versión de depuración de OpenCore

* **Vault**: `Optional`
  * No trataremos el Vaulting, por lo que podemos ignorar esto, ten en cuenta que **no podrás bootear si esto está configurado en `Secure`**
  * Esta es una palabra, no es opcional omitir esta configuración. Lo lamentarás si no lo configuras en `Optional`, ten en cuenta que distingue entre mayúsculas y minúsculas

* **ScanPolicy**: `0`
  * `0` te permite ver todas las unidades disponibles, consulta la sección [Seguridad](https://dortania.github.io/OpenCore-Post-Install/universal/security.html) para obtener más detalles. **No arrancará dispositivos USB con este ajuste predeterminado**
  
* **SecureBootModel**: Default
  * Habilita la funcionalidad del arranque seguro de Apple en macOS, por favor refiérete a [Seguridad](https://inyextciones.github.io/OpenCore-Post-Install/universal/security.html) para más información.
   * Nota: Los usuarios pueden encontrar fallas de arranque cuando actualizen OpenCore en un sistema que ya contenga una instalación. Para resolver esto, dirígete aquí: [Trancado en OCB: LoadImage failed - Security Violation](/troubleshooting/extended/kernel-issues.md#trancado-en-ocb-loadimage-failed-security-violation)

:::

### Tools

Utilizado para ejecutar herramientas de depuración OC como el shell, la función de "Snapshot" de ProperTree las agregará por tí.

### Entries

Se usa para especificar rutas de inicio irregulares que no se pueden encontrar naturalmente con OpenCore.

No se cubrirá aquí, consulte 8.6 de [Configuration.pdf](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf) para obtener más información

## NVRAM

![NVRAM](../images/config/config-universal/nvram.png)

### Add

::: tip 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14

Utilizado para el escalado de la interfaz de usuario de OpenCore, el valor predeterminado funcionará para nosotros. Consulta la sección detallada para obtener más información.

:::

::: details Información más detallada

Ruta del Booter, utilizada principalmente para escalar la interfaz de usuario

* **UIScale**:
  * `01`: Resolución estándar
  * `02`: HiDPI (generalmente requerido para FileVault para funcionar correctamente en pantallas más pequeñas)

* **DefaultBackgroundColor**: Color de fondo utilizado por boot.efi
  * `00000000`: Syrah Negro
  * `BFBFBF00`: Gris Claro

:::

::: tip 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102

GUID de NVRAM de OpenCore, principalmente relevante para usuarios de RTCMemoryFixup

:::

::: details Información más detallada

* **rtc-blacklist**: <>
  * Para ser usado en conjunto con RTCMemoryFixup, dirígete aquí para obtener más información: [Arreglando errores de escritura del RTC](https://dortania.github.io/OpenCore-Post-Install/misc/rtc.html#finding-our-bad-rtc-region)
  * La mayoría de usuarios pueden ignorar esta sección

:::

::: tip 7C436110-AB2A-4BBB-A880-FE41995C9F82

Máscara de bits de Protección de integridad del sistema

* **General Purpose boot-args**:

| boot-args | Descripción |
| :--- | :--- |
| **-v** | Esto habilita el modo detallado, que muestra todo el texto detrás de escena que se desplaza al iniciar, en lugar del logotipo de Apple y la barra de progreso. Es invaluable para cualquier Hackintosher, ya que le da una mirada interna al proceso de arranque y puede ayudarlo a identificar problemas, problemas con kexts, etc. |
| **debug=0x100** | Esto deshabilita el watchdog de macOS que ayuda a evitar el reinicio en caso de un kernel panic. De esa manera, puedes *con suerte* obtener información útil y usarla para solucionar este problema. |
| **keepsyms=1** | Esta es una configuración complementaria para debug=0x100 que le dice al sistema operativo que también imprima los símbolos en un kernel panic. Eso puede dar una idea más útil de lo que está causando el pánico en sí. |
| **alcid=1** | Utilizado para configurar el layout-id para AppleALC, consulta [códecs compatibles](https://github.com/acidanthera/applealc/wiki/supported-codecs) para averiguar qué layout usar para su sistema específico. Más información sobre esto está cubierta en la [Página de post instalación](https://dortania.github.io/OpenCore-Post-Install/) |

* **GPU-Specific boot-args**:

| boot-args | Description |
| :--- | :--- |
| **agdpmod=pikera** | Se usa para deshabilitar el BoardID en las GPUs Navi (series RX 5000). Sin esto obtendrás una pantalla negra. **No lo uses si no tienes Navi** (es decir, las tarjetas Polaris y Vega no deberían usar esto) |
| **nvda_drv_vrl=1** | Se utiliza para habilitar los Web Drivers de Nvidia en las tarjetas Maxwell y Pascal en Sierra y High Sierra |


* **csr-active-config**: `00000000` 

  * Configuración de la 'Protección de integridad del sistema' (SIP). En general, se recomienda cambiar esto con `csrutil` a través de la partición de recuperación.
  * csr-active-config de forma predeterminada se establece en `00000000` que habilita la Protección de integridad del sistema. Puedes elegir una cantidad de valores diferentes, pero en general recomendamos mantener esto habilitado para tener mayor seguridad. Puedes encontrar más información en nuestra página de solución de problemas: [Desactivar el SIP](../troubleshooting/extended/post-issues.md#deshabilitando-el-sip)

* **run-efi-updater**: `No`
  * Esto es utilizado para prevenir la instalación de los paquetes de firmware de Apple. Esto es especialmente importante para hardware de PCs de escritorio debido a que estas actualizaciones de firmware se crashearán en las actualizaciones.

* **prev-lang:kbd**: <>
  * Necesario para teclados no latinos en el formato de `lang-COUNTRY:keyboard`, se recomienda mantener en blanco aunque puede especificarlo (**El valor predeterminado en el sample.plist es ruso**):
  * Inglés de Estados Unidos: `en-US:0`(`656e2d55533a30` en HEX)
  * Una lista completa puede ser encontrada en [AppleKeyboardLayouts.txt](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt)
  * Sugerencia: `prev-lang:kbd` se puede cambiar a un string para que puedas ingresar `en-US: 0` directamente en lugar de convertir a HEX

| Key | Type | Value |
| :--- | :--- | :--- |
| prev-lang:kbd | String | en-US:0 |

:::

### Delete

::: tip Información

Vuelve a escribir a la fuerza las variables de NVRAM. Ten en cuenta que `Add` **no sobrescribirá** los valores ya presentes en NVRAM, por lo que  valores como `boot-args` deben dejarse solos. En nuestro caso, cambiaremos lo siguiente:

| Quirk | Habilitado |
| :--- | :--- |
| WriteFlash | YES |

:::

::: details Información más detallada

**LegacyEnable**: NO
  * Permite que la NVRAM se almacene en nvram.plist, necesario para sistemas sin NVRAM nativa

**LegacyOverwrite**: NO
  * Permite sobrescribir variables de firmware de la nvram.plist, solo necesario para sistemas sin NVRAM nativa

**LegacySchema**
  * Se usa para asignar variables NVRAM, se usa con LegacyEnable establecido en YES

**WriteFlash**: YES
  * Permite escribir en la memoria flash para todas las variables agregadas.

:::

## PlatformInfo

![PlatformInfo](../images/config/config-universal/iMacPro-smbios.png)

::: tip Info

Para configurar la información del SMBIOS, usaremos la aplicación [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) hecha por [CorpNewt](https://github.com/corpnewt/).

Para este ejemplo elegiremos el SMBIOS iMacPro1,1, pero otras SMBIOS pueden funcionar mejor que otras con ciertas GPUs:

* iMacPro1,1: AMD RX Polaris y posterior
* MacPro7,1: AMD RX Polaris y posterior
  * Ten en cuenta que MacPro7,1 es exclusiva para macOS 10.15, Catalina y posterior
* MacPro6,1: AMD R5/R7/R9 y más viejos
* iMac14,2: Nvidia Kepler y posterior
  * Nota: iMac14,2 es soportada desde 10.8 a 10.15, así que para macOS 11 y posterior por favor usa MacPro7,1

Ejecuta GenSMBIOS, elije la opción 1 para descargar MacSerial y la Opción 3 para seleccionar la SMBIOS que deseas. Esto nos dará una salida **similar** a la siguiente:

```sh
  #######################################################
 #              iMacPro1,1 SMBIOS Info                 #
#######################################################

Type:         iMacPro1,1
Serial:       C02YX0TZHX87
Board Serial: C029269024NJG36CB
SmUUID:       DEA17B2D-2F9F-4955-B266-A74C47678AD3
```

El orden es `Product | Serial | Board Serial (MLB)`

La parte `Type` se copia en Generic -> SystemProductName.

La parte `Serial` se copia en Generic -> SystemSerialNumber.

La parte `Board Serial` se copia en Generic -> MLB.

La parte `SmUUID` se copia en Generic -> SystemUUID.

Configuramos Generic -> ROM a una ROM de Apple (extraída de una Mac real), la dirección MAC de tu NIC o cualquier dirección MAC aleatoria (podría ser solo 6 bytes aleatorios Para esta guía usaremos `11223300 0000`. Después de instalar dirígete a la página [Arreglando iServices](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html) para saber cómo encontrar tu dirección MAC real)

**Recuerda que quieres o una serie no válida o números de serie válidos, pero NO quieres series válidas que están en uso, quieres recibir un mensaje de regreso como: "Serie no válida" o "Fecha de compra no validada"**

[Página de verificación de cobertura de Apple](https://checkcoverage.apple.com)

**Automatic**: YES

* Genera PlatformInfo basada en la sección Generic en lugar de las secciones DataHub, NVRAM y SMBIOS

:::

### Generic

::: details Información más detallada

* **AdviseWindows**: NO
  * Se usa cuando la partición EFI no es la primera en la unidad de Windows

* **MaxBIOSVersion**: NO
  * Establece la versión de la BIOS a la máxima para prevenir actualizaciones de firmware en Big Sur y posterior. Esto es principalmente aplicable para Macs genuinas. 

* **ProcessorType**: `0`
  * Ponlo en `0` para la detección automática del tipo de procesador. Sin embargo, este valor puede ser cambiado si lo deseas. Mira [AppleSmBios.h](https://github.com/acidanthera/OpenCorePkg/blob/master/Include/Apple/IndustryStandard/AppleSmBios.h) para ver valores posibles.

* **SpoofVendor**: YES
  * Intercambia el campo del proveedor por Acidanthera. Generalmente no es seguro usar Apple como proveedor.

* **SystemMemoryStatus**: Auto
  * Establece si la memoria está soldada o no en la información del SMBIOS, esto es puramente cosmético, por lo que recomendamos "Auto"

* **UpdateDataHub**: YES
  * Actualiza campos de Data Hub

* **UpdateNVRAM**: YES
  * Actualiza campos de NVRAM

* **UpdateSMBIOS**: YES
  * Actualiza campos SMBIOS

* **UpdateSMBIOSMode**: Create
  * Reemplaza las tablas con EfiReservedMemoryType recientemente asignadas, usar `Custom` en las laptops Dell que requieren el quirk CustomSMBIOS
  * Configurando esto a `Custom` con el quirk CustomSMBIOSGuid habilitado también puede deshabilitar la inyección de SMBIOS en sistemas operativos que no son de Apple, aunque no recomendamos esto ya que rompe la compatibilildad con BootCamp. Úsalo bajo tu propio riesgo.


:::

## UEFI

![UEFI](../images/config/config-universal/aptio-v-uefi.png)

**ConnectDrivers**: YES

* Fuerza drivers `.efi`. El cambio a `NO` conectará automáticamente los drivers UEFI agregados. Esto puede hacer que el arranque sea un poco más rápido, pero no todos los drivers se conectan. (ej. ciertos drivers del sistema pueden no cargarse).

### Drivers

Agrega tus drivers .efi aquí

Los únicos drivers presentes aquí deberían ser:

* HfsPlus.efi
* OpenRuntime.efi

### APFS

Settings related to the APFS driver, leave everything here as default.

### Audio

En relación con la configuración de AudioDxe, por ahora ignoraremos esto (dejar como predeterminado). Esto no está relacionado con el soporte de audio en macOS.

* Para un más información sobre uso de AudioDxe y la sección Audio, consulta la página de post instalación: [Agregar la GUI y el chime](https://dortania.github.io/OpenCore-Post-Install/)

### Input

Related to boot.efi keyboard passthrough used for FileVault and Hotkey support, leave everything here as default as we have no use for these quirks. See here for more details: [Security and FileVault](https://inyextciones.github.io/OpenCore-Post-Install/)

### Output

Relacionado a la salida visual de OpenCore, deja todo aquí como predeterminado, ya que no tenemos uso para estos quirks.

### ProtocolOverrides

Principalmente relevante para máquinas virtuales, Macs Legacy y usuarios de FileVault. Consulta aquí para obtener más detalles: [Seguridad y FileVault](https://inyextciones.github.io/OpenCore-Post-Install/)

### Quirks

::: tip Info
En relación con los quirks con el entorno UEFI, cambiaremos lo siguiente:

| Quirk | Enabled | Comment |
| :--- | :--- | :--- |
| UnblockFsConnect | NO | Necesario principalmente para placas madre HP  |

:::

::: details Información más detallada


* **RequestBootVarRouting**: YES
  * Redirige AptioMemoryFix desde `EFI_GLOBAL_VARIABLE_GUID` a `OC_VENDOR_VARIABLE_GUID`. Necesario para cuando el firmware intenta eliminar las entradas de arranque y se recomienda que esté habilitado en todos los sistemas para la instalación correcta de las actualizaciones, el funcionamiento del panel de control del disco de inicio, etc.

* **DisableSecurityPolicy**: NO
  * Deshabilita la Política de Seguridad de la Plataforma (Platform Security Policy) en el firmware, recomendado para firmwares con errores donde la deshabilitación del arranque seguro no permite la carga de drivers en firmwares de terceros.
  * Si tienes un dispositivo Microsoft Surface , te recomendamos que habilites esta opción.

* **UnblockFsConnect**: NO
  * Algunos firmwares bloquean a las llamadas "partition handles" abriéndolas en modo de controlador, lo que hace que los protocolos del sistema de archivos no puedan instalarse. Principalmente relevante para sistemas HP cuando no se ven discos enumerados

:::

### ReservedMemory

Se utiliza para eximir ciertas regiones de memoria de los sistemas operativos a utilizar, principalmente relevante para iGPUs o sistemas Sandy Bridge con memoria defectuosa. El uso de este quirk no está cubierto en esta guía

## Cleaning up

Y ahora estás listo para guardar y colocarlo en tu EFI bajo EFI/OC.

Para aquellos que tienen problemas de booteo, asegúrense de leer primero la [Sección de solución de problemas](../troubleshooting/troubleshooting.md) y si tus preguntas siguen sin respuesta, tenemos muchos recursos a tu disposición:

* [Subreddit de Hackintosh](https://www.reddit.com/r/hackintosh/)
* [Nuestro servidor de Discord](https://discord.gg/2QYd7ZT)

* **NOTA IMPORTANTE: El Subreddit y el servidor de Discord sólo admiten posts en inglés, así que asegúrate de hacerlos en este idioma para evitar inconvenientes.**

**Sanity check**:

Gracias a los esfuerzos de Ramus, también tenemos una herramienta increíble para ayudar a verificar su configuración para aquellos que pueden haberse perdido algo:

* [**Sanity Checker**](https://opencore.slowgeek.com)

Ten en cuenta que esta herramienta no está hecha ni mantenida por Dortania, todos los problemas con este sitio deben enviarse aquí: [Repositorio del Sanity Checker](https://github.com/rlerdorf/OCSanity)

## Configuración de la BIOS para AMD

* Nota: La mayoría de las opciones pueden no llegar a estar presentes en tu firmware. Recomendamos que lo hagas coincidir lo más que puedas, pero no te preocupes demasiado si hay opciones que no están disponibles en tu BIOS.

### Deshabilitar

* Fast Boot
* Secure Boot
* Serial/COM Port
* Parallel Port
* Compatibility Support Module (CSM)(**Debe estar deshabilitado, errores de GPU como `gIO` son comunes cuando esta opción está habilitada**)

### Habilitar

* Above 4G decoding(**Esto debe estar habilitado, si no puedes encontrar esta opción agrega `npci=0x2000` a tus boot-args. No tengas esta opción y el boot arg npci habilitadas al mismo tiempo**)
* EHCI/XHCI Hand-off
* OS type: Windows 8.1/10 UEFI Mode
* SATA Mode: AHCI

# Ahora con todo esto hecho, dirígete a la [página de instalación](/installation/installation-process)
