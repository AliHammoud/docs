# Маршрутизация с помощью NAT-инстанса

_NAT-инстанс_ — специальная виртуальная машина с преднастроенными правилами маршрутизации и трансляции IP-адресов.

В {{ yandex-cloud }} можно настроить связь нескольких ВМ с интернетом через NAT-инстанс с помощью [статической маршрутизации](../../vpc/concepts/static-routes.md). При этом будет использован только один публичный IP-адрес — тот, который присвоен NAT-инстансу.

Чтобы настроить маршрутизацию через NAT-инстанс:

1. [Подготовьте облако к работе](#before-you-begin).
1. [Создайте тестовую виртуальную машину](#create-vm).
1. [Создайте NAT-инстанс](#create-nat-instance).
1. [Настройте статическую маршрутизацию в облачной сети](#configure-static-route).
1. [Проверьте работу NAT-инстанса](#test-nat-instance). 

Если созданные ресурсы вам больше не нужны, [удалите их](#clear-out).

## Перед началом работы {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки NAT-инстанса входят:

* плата за постоянно запущенные ВМ (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md));
* плата за использование динамического или статического внешнего IP-адреса (см. [тарифы {{ vpc-full-name }}](../../vpc/pricing.md)).

### Подготовьте инфраструктуру {#deploy-infrastructure}

1. [Создайте](../../vpc/operations/network-create.md) облачную сеть, например `my-vpc`.
1. В облачной сети [создайте](../../vpc/operations/subnet-create.md) подсети, например:
    * `public-subnet`, в которой будет размещен NAT-инстанс;
    * `private-subnet`, в которой будет размещена тестовая ВМ.

## Создайте тестовую виртуальную машину {#create-vm}

{% list tabs %}

- Консоль управления

   1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором хотите создать тестовую ВМ.
   1. В списке сервисов выберите **{{ compute-name }}**.
   1. Нажмите кнопку **Создать ВМ**.
   1. В блоке **Базовые параметры**:
         * В поле **Имя** введите имя ВМ, например `test-vm`.
         * В поле **Зона доступности** выберите зону доступности, где находится подсеть `private-subnet`.
   1. В блоке **Выбор образа/загрузочного диска** выберите один из образов и версию операционной системы на базе Linux.
   1. В блоке **Сетевые настройки**:
         * В поле **Подсеть** выберите подсеть для тестовой ВМ, например `private-subnet`.
         * В поле **Публичный адрес** выберите **Без адреса**.
         * В поле **Внутренний IPv4-адрес** выберите **Автоматически**.
   1. В блоке **Доступ**:
         * В поле **Логин** введите имя пользователя.
         * В поле **SSH-ключ** вставьте содержимое файла с публичным [SSH-ключом](../../glossary/ssh-keygen.md). Пару ключей для подключения по SSH необходимо [создать](../../compute/operations/vm-connect/ssh.md#creating-ssh-keys) самостоятельно.
   1. Нажмите кнопку **Создать ВМ**.

   Сохраните имя пользователя, закрытый SSH-ключ и внутренний IP-адрес тестовой ВМ.

{% endlist %}

## Создайте NAT-инстанс {#create-nat-instance}

{% list tabs %}

- Консоль управления

   1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором хотите создать NAT-инстанс.
   1. В списке сервисов выберите **{{ compute-name }}**.
   1. Нажмите кнопку **Создать ВМ**.
   1. В блоке **Базовые параметры**:
         * В поле **Имя** введите имя ВМ для NAT-инстанса, например `nat-instance`.
         * В поле **Зона доступности** выберите зону доступности, где находится подсеть `public-subnet`.
   1. В блоке **Выбор образа/загрузочного диска** перейдите на вкладку **{{ marketplace-name }}** и выберите образ [NAT-инстанс](/marketplace/products/yc/nat-instance-ubuntu-18-04-lts).
   1. В блоке **Сетевые настройки**:
         * В поле **Подсеть** выберите подсеть для NAT-инстанса, например `public-subnet`.
         * В поле **Публичный адрес** выберите **Автоматически**.
         * В поле **Внутренний IPv4-адрес** выберите **Автоматически**.
   1. В поле **Доступ**:
         * В поле **Логин** введите имя пользователя.
         * В поле **SSH-ключ** вставьте содержимое файла с публичным SSH-ключом. Пару ключей для подключения по SSH необходимо создать самостоятельно.
   1. Нажмите кнопку **Создать ВМ**.

   Сохраните имя пользователя, закрытый SSH-ключ, внутренний и публичный IP-адреса NAT-инстанса.

{% endlist %}

## Настройте статическую маршрутизацию {#configure-static-route}

{% note info %}

При создании NAT-инстанса автоматически настраивается только один сетевой интерфейс. Остальные интерфейсы можно включить вручную. Назначьте каждому новому интерфейсу IP-адрес и пропишите для него маршрут в таблице маршрутизации. В каждой подсети корректным шлюзом будет первый адрес — `x.x.x.1`.

{% endnote %}

{% list tabs %}

- Консоль управления

   1. Создайте таблицу маршрутизации и добавьте в нее статический маршрут: 

      1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором хотите создать статический маршрут.
      1. В списке сервисов выберите **{{ vpc-name }}**.
      1. Перейдите на вкладку ![route-tables](../../_assets/vpc/route-tables.svg) **Таблицы маршрутизации**.
      1. Нажмите кнопку **Создать таблицу маршрутизации**.
      1. Задайте имя таблицы маршрутизации, например `nat-instance-route`.

         {% include [name-format](../../_includes/name-format.md) %}

      1. Выберите сеть, например `my-vpc`.
      1. Нажмите кнопку **Добавить маршрут**.
      1. В открывшемся окне введите префикс подсети назначения `0.0.0.0/0`.
      1. В поле **Next hop** выберите **IP-адрес**.
      1. Укажите внутренний IP-адрес NAT-инстанса. Нажмите кнопку **Добавить**.
      1. Нажмите кнопку **Создать таблицу маршрутизации**.
   1. Привяжите таблицу маршрутизации к подсети, где находится тестовая ВМ, например `private-subnet`:
      1. Перейдите на вкладку ![subnets](../../_assets/vpc/subnets.svg) **Подсети**.
      1. Напротив подсети с тестовой ВМ, например `private-subnet`, нажмите кнопку ![options](../../_assets/options.svg).
      1. В открывшемся меню выберите пункт **Привязать таблицу маршрутизации**.
      1. В открывшемся окне выберите таблицу `nat-instance-route` из списка.
      1. Нажмите кнопку **Привязать**.

   Созданный маршрут можно применять и к другим подсетям в той же сети, кроме подсети, где находится NAT-инстанс.

{% endlist %}

## Проверьте работу NAT-инстанса {#test-nat-instance}

1. [Подключитесь](../../compute/operations/vm-connect/ssh.md#vm-connect) к NAT-инстансу по SSH, указав:
   * путь к файлу с закрытым SSH-ключом NAT-инстанса;
   * имя пользователя NAT-инстанса;
   * публичный IP-адрес NAT-инстанса.

   Для этого в терминале выполните команду:

   ```bash
   ssh -i <путь_к_файлу_с_закрытым_SSH-ключом_NAT-инстанса> \
     <имя_пользователя_NAT-инстанса>@<публичный_IP-адрес_NAT-инстанса>
   ```

1. Создайте на NAT-инстансе файл с закрытым SSH-ключом тестовой ВМ, например `private-key`:

    ```bash
    sudo nano private-key
    ```

    Вставьте в файл содержимое закрытого SSH-ключа тестовой ВМ.

1. Из NAT-инстанса подключитесь по SSH к тестовой ВМ в той же облачной сети, указав:
   * путь к файлу с закрытым SSH-ключом тестовой ВМ, например `private-key`;
   * имя пользователя тестовой ВМ;
   * внутренний IP-адрес тестовой ВМ.

   Для этого в терминале выполните команду:

   ```bash
   ssh -i <путь_к_файлу_с_закрытым_SSH-ключом_тестовой_ВМ> \
     <имя_пользователя_тестовой_ВМ>@<внутренний_IP-адрес_тестовой_ВМ>
   ```

1. Убедитесь, что доступ в интернет тестовая ВМ получает через публичный IP-адрес NAT-инстанса. Введите в терминале команду:

   ```bash
   curl ifconfig.co
   ```

   В результате должен быть выведен публичный IP-адрес NAT-инстанса, значит все настроено правильно.

## Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за созданные ресурсы, [удалите](../../compute/operations/vm-control/vm-delete.md) тестовую ВМ и NAT-инстанс.
