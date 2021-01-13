# RTL8723BS-linux-module
Modulo de Wifi para el tablet SW1-011 (RTL8723bs)

Para compilar el modulo deber realizar lo siguente:

### Obtener las dependencies
sudo apt-get install git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex
### Crea una carpeta para realizar todo el procedimiento. En mi caso use la carpeta personal.
mkdir linux
### Obtener la fuente del Kernel. En mi caso use el repositorio de "Hans de Goede" ya que dispone de los branch por orden de version.
git clone -b v5.4-footrail https://github.com/jwrdegoede/linux-sunxi.git # El comando -b (version) es del branch de la version de kernel
### Copia la configuracion existente de tu Kernel del sistema
cp /boot/config-`uname -r` .config
### Copia el archivo "symbol" de tu kernel al direcctorio de la fuente de kernel que usaras. Este archivo guarda que nombre de las funciones existen en el kernel que estas usando, asi que nuestro nuevo modulo estara ahí.
cp /usr/src/linux-headers-5.4.0-050400-generic/Module.symvers ./          #El direcctorio cambiara segun la version del kernel que uses
### Carga tu configuracion original de tu kernel. En el caso de tu kernel sea significativamente nuevo que el kernel existente en el arhivo de configurancion, vas a toda la nueva configuracion no existente en tu archivo de configuracion. Puedes solamente sentarte y darle a enter para hacer todo por default (generalmente seguro), o puedes solamente escribir lo siguente
yes '' | make oldconfig
### Desde que estamos usando una version generica del kernel, debes modifical el archivo "Makefile" de la fuente del kernel descargado y en donde dice EXTRAVERSION = -050400-generic. Para que asi coincida con la version del modulo en uso. Puedes usar el comando "uname -r"
### Añade el Wifi a los ids de dispositivos
Ve a drivers/staging/rtl8723bs/os_dep/sdio_intf.c: y añade esta linea " { SDIO_DEVICE(0x024c, 0x0627), }, " y guarda el archivo
### Construye el modulo
make scripts prepare modules_prepare
make -C . M=drivers/staging/rtl8723bs
### Instala el modulo
sudo cp drivers/staging/rtl8723bs/rtl8723bs.ko /lib/modules/5.4.0-050400-generic/kernel/drivers/staging/rtl8723bs/
sudo depmod
### Recarga el driver del controlador
echo '80860F14:01' > /sys/bus/platform/drivers/sdhci-acpi/unbind
echo '80860F14:01' > /sys/bus/platform/drivers/sdhci-acpi/bind
### Verifica que funciona
dmesg
iwconfig
