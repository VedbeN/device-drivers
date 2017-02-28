
< ## Разработка драйверов для смарт-терминала Эвотор >

#### 3. SDK для Банковских Терминалов;

Для написания приложения-драйвера для Эвотор, требуется выполнить несколько простых шагов.

> Здесь и далее по тексту все имена констант указаны из ru.evotor.devices.drivers.Constants.

### 1. Подключить к своему проекту библиотеку для работы с оборудованием.

Для этого в `build.gradle` проекта добавьте ссылку репозиторий jitpack:

```
allprojects {
    repositories {
        jcenter()
        maven { url 'https://jitpack.io' }
    }
}
```

и в модуле `build.gradle` добавьте зависимость следующим образом:

```
dependencies {
    compile 'com.github.evotor:device-drivers:+'
}
```

### 2. Определите внешний сервис в `AndroidManifest.xml` приложения.

Для сервиса должен быть указан хотя бы один из интент-фильтров `INTENT_FILTER_DRIVER_MANAGER` или `INTENT_FILTER_VIRTUAL_DRIVER_MANAGER`.

Пример объявленного сервиса:

```
<service
    android:name="ru.mycompany.drivers.MyDeviceService"
    android:enabled="true"
    android:exported="true"
    android:icon="@drawable/logo"
    android:label="@string/service_name">
    <intent-filter>
        <action android:name="ru.evotor.devices.drivers.DriverManager" />
        <action android:name="ru.evotor.devices.drivers.ScalesService" />
    </intent-filter>
    <meta-data
        android:name="vendor_name"
        android:value="CAS" />
    <meta-data
        android:name="model_name"
        android:value="AD" />
    <meta-data
        android:name="usb_device"
        android:value="VID_1659PID_8963" />
    <meta-data
        android:name="settings_activity"
        android:value="" />
    <meta-data
        android:name="device_categories"
        android:value="SCALES" />
</service>
```  

`INTENT_FILTER_DRIVER_MANAGER` - используется для драйверов, которые требуют для работы подключенное USB-оборудование. Вместе с этим необходимо указать для сервиса в `meta-data` характеристики VendorID и ProductID целевого устройства (десятичными числами):

```
<meta-data
    android:name="usb_device"
    android:value="VID_1659PID_8963" />
```

При необходимости, можно указать несколько устройств следующим образом: `"VID_1659PID_8963|VID_123PID_456|VID_1659PID_8964"`.

Экземпляр драйвера будет автоматически создан/удалён при подключении/отключении указанного оборудования к смарт-терминалу. При наличии нескольких подходящих драйверов, пользователю будет предложен выбор.

`INTENT_FILTER_VIRTUAL_DRIVER_MANAGER` - используется для драйверов, не требующих USB-оборудования (сетевое, bluetooth и др. оборудование). Вместе с этим необходимо указать в `meta-data`, что драйвер является виртуальным:

```
<meta-data
    android:name="virtual_device"
    android:value="true" />
```

Такой драйвер может быть создан только пользователем вручную через меню настройки оборудования. В этом случае все работы по подключению к нужному устройству берёт на себя производитель драйвера.

Следующие интент-фильтры используются для реализации ролей устройства:

`INTENT_FILTER_PAY_SYSTEM` - для банковских терминалов;

Вместе с этим необходимо указать в `meta-data` категорию устройства:

```
<meta-data
    android:name="device_categories"
    android:value="SCALES" />
```

`PAYSYSTEM` - для банковских терминалов;

Можно указать сразу несколько категорий устройств следующим образом: `"PAYSYSTEM|PRICEPRINTER|CASHDRAWER"`.

В манифесте приложения у сервиса должны быть указаны `android:icon` и `android:label` - картинка и имя драйвера (показывается пользователю). Картинку желательно делать квадратной, png без фона.

  // TODO: нужен пример с банковским терминалом.
  
![Пример отображения иконки и имени драйвера](https://github.com/VedbeN/device-drivers/blob/master/icon_xmpl.png?raw=true)

Можно задать `activity`  настроек, если это требуется:

```
<meta-data
    android:name="settings_activity"
    android:value="ru.mycompany.drivers.MySettingsActivity" />
```

Указанная `activity`  должна находиться в текущем `package` и будет вызвана при первом подключении устройства или по нажатию на строчку с оборудованием в меню настроек оборудования.

Версия драйвера (`versionCode` и `versionName`) берётся из `build.gradle`:

```
defaultConfig {
    applicationId "ru.mycompany.drivers.myscales"
    minSdkVersion 22
    targetSdkVersion 24
    versionCode 2
    versionName "1.0.1"
}
```

`MinSdkVersion` должна быть не выше версии 22!

### 3. В реализации метода подключения к сервису для всех `action` указанных в интент-фильтрах укажите соответствующий Binder'ы:

для `INTENT_FILTER_DRIVER_MANAGER` - класс наследник `ru.evotor.devices.drivers.IUsbDriverManagerService.Stub`;

для `INTENT_FILTER_VIRTUAL_DRIVER_MANAGER` - класс наследник `ru.evotor.devices.drivers.IVirtualDriverManagerService.Stub`;

для `INTENT_FILTER_PAY_SYSTEM` - класс наследник `ru.evotor.devices.drivers.IPaySystemDriverService.Stub`;

Например:

  // TODO: нужен пример с банковским терминалом.
  
```

```

В этом же сервисе удобно определить `Map` для хранения списка активных экземпляров драйверов (а их, потенциально, может быть больше, чем 1 в системе одновременно), т.к. обращаться к нему придётся из всех указанных Stub'ов.

  // TODO: нужна ссылка на описание, что такое `Map`.
  
### 4. Опишите указанные Binder'ы.

  // TODO: нужен пример с банковским терминалом.
  
Для всех описываемых методов в случае невозможности выполнить требуемое действие (например, взвесить для метода `getWeight`) следует задействовать поддерживаемый `Exception` тип (с текстовым человекочитаемым описанием проблемы).

Поддерживаемые `Exception` типы:
`BadParcelableException`;
`IllegalArgumentException`;
`IllegalStateException`;
`NullPointerException`;
`SecurityException`;
`NetworkOnMainThreadException`.

Proof: https://developer.android.com/reference/android/os/Parcel.html#writeException%28java.lang.Exception%29

#### `IUsbDriverManagerService.Stub` - класс для управления драйверами usb-устройств: подключение и отключение устройств происходят здесь.  Требуется реализовать методы `addUsbDevice` и `destroy`.

```
import ru.evotor.devices.drivers.IUsbDriverManagerService;

public class MyDriverManagerStub extends IUsbDriverManagerService.Stub {

    private MyDeviceService myDeviceService;

    public MyDriverManagerStub(MyDeviceService myDeviceService) {
        this.myDeviceService = myDeviceService;
    }

    @Override
    public int addUsbDevice(UsbDevice usbDevice, String usbPortPath) throws RemoteException {
        return myDeviceService.createNewDevice(usbDevice);
    }

    @Override
    public void destroy(int instanceId) throws RemoteException {
        myDeviceService.destroy(instanceId);
    }
}
```

Метод `addUsbDevice` в `IUsbDriverManagerService` принимает на вход:

1) `UsbDevice`, для которого он создан;

2) некоторый строковый идентификатор номера физического usb-порта (может потребоваться, например, если  требуется сохранить какие-либо настройки оборудования и восстановить их после перезагрузки терминала). В этот момент у приложения-драйвера уже есть `permission` для работы с этим устройством.

Метод `addUsbDevice` возвращает номер экземпляра драйвера внутри приложения. По этому номеру будет происходить обращение к конкретному драйверу.

Метод `destroy` в `IUsbDriverManagerService` принимает на вход номер экземпляра драйвера. Вызов этого метода уведомляет приложение об отключении от устройства. В этот момент у приложения-драйвера уже нет `permission` для работы с этим устройством, само устройство уже может быть удалено из смарт-терминала.


#### `IVirtualDriverManagerService.Stub` - класс для управления драйверами виртуальных устройств:
подключение и отключение устройств происходят здесь. Требуется реализовать методы `addNewVirtualDevice`, `recreateNewVirtualDevice` и `destroy`.

```
import ru.evotor.devices.drivers.IVirtualDriverManagerService;

public class MyDriverManagerStub extends IVirtualDriverManagerService.Stub {

    private MyDeviceService myDeviceService;

    public MyDriverManagerStub(MyDeviceService myDeviceService) {
        this.myDeviceService = myDeviceService;
    }

    @Override
    public int addNewVirtualDevice() throws RemoteException {

            return myDeviceService.createNewDevice(usbDevice);
    }

    @Override
    public void recreateNewVirtualDevice(int instanceId) throws RemoteException {
        myDeviceService.recreateNewVirtualDevice(instanceId);
    }

    @Override
    public void destroy(int i) throws RemoteException {
        myDeviceService.destroy(instanceId);
    }
}
```

Метод `addNewVirtualDevice` возвращает номер экземпляра драйвера внутри приложения. По этому номеру будет происходить обращение к конкретному драйверу.

Метод `recreateNewVirtualDevice` принимает на вход номер экземпляра драйвера внутри приложения.

Метод `destroy` принимает на вход номер экземпляра драйвера. Вызов этого метода уведомляет приложение об отключении от устройства.

Для вновь созданного экземпляра драйвера (а виртуальные устройства могут создаваться только вручную пользователем через меню настроек оборудования) будет вызван метод `addNewVirtualDevice`.

Метод `recreateNewVirtualDevice` будет вызван для тех устройств, которые уже создавались пользователем ранее, но в данный момент подключения к таким драйверам нет. Например, после перезагрузки смарт-терминала, перезапуска сервиса работы с оборудованием или обновления приложения-драйвера.

Метод `destroy` будет вызван для устройства, которое пользователь вручную удалил из списка оборудования.

#### `IPaySystemDriverService.Stub` - класс для работы с конкретными экземплярами

```
import ru.evotor.devices.drivers.IPaySystemDriverService;
import ru.evotor.devices.drivers.paysystem.PayResult;
import ru.evotor.devices.drivers.paysystem.PayInfo;

public class MyPaySystemStub implements IPaySystemDriverService.Stub {

    private MyDeviceService myDeviceService;

    public MyPaySystemStub(MyDeviceService myDeviceService) {
        this.myDeviceService = myDeviceService;
    }

    @Override
    public PayResult payment(int instanceId, PayInfo payInfo) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).payment(payInfo);
    }

    @Override
    public PayResult cancelPayment(int instanceId, PayInfo payInfo, String rrn) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).cancelPayment(payInfo, rrn);
    }

    @Override
    public PayResult payback(int instanceId, PayInfo payInfo, String rrn) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).payback(payInfo, rrn);
    }

    @Override
    public PayResult cancelPayback(int instanceId, PayInfo payInfo, String rrn) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).cancelPayback(payInfo, rrn);
    }

    @Override
    public PayResult closeSession(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).closeSession();
    }

    @Override
    public void openServiceMenu(int instanceId) throws RemoteException {
        myDeviceService.getMyDevice(instanceId).openServiceMenu();
    }

    @Override
    public String getBankName(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).getBankName();
    }

    @Override
    public int getTerminalNumber(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).getTerminalNumber();
    }

    @Override
    public String getTerminalID(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).getTerminalID();
    }

    @Override
    public String getMerchNumber(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).getMerchNumber();
    }

    @Override
    public String getMerchCategoryCode(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).String();
    }

    @Override
    public String getMerchEngName(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).getMerchEngName();
    }

    @Override
    public String getCashier(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).getCashier();
    }

    @Override
    public String getServerIP(int instanceId) throws RemoteException {
        return myDeviceService.getMyDevice(instanceId).getServerIP();
    }
}
```

Все методы принимают на вход номер экземпляра драйвера.

Метод оплаты принимает на вход информацию об оплате (сумму), методы возврата и отмены дополнительно к этому принимают на вход `РРН` прошлой операции.

  // TODO: нужна ссылка на описание, что такое `РРН`. 

### 5. После того, как описаны все классы для взаимодействия с инфраструктурой смарт-терминала, можно описать сам класс работы с оборудованием:

  // TODO: нужен пример с банковским терминалом.
  
Для банковских терминалов это выглядит следующим образом:

```

```

Интерфейс для банковских терминалов - `ru.evotor.devices.drivers.paysystem.IPaySystem`.

### 6. Всё готово.

Загрузите приложение на смарт-терминал, чтобы работать с Вашим драйвером.

-----
