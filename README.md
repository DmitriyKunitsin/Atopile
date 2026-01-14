# ```Краткая памятка```:

1. Логику компонента пишем в .ato (resistor.ato, button.ato).
2. Чтобы KiCad увидел «картинку» на схеме → .kicad_sym.
3. Чтобы KiCad посадил корпус на плату → .kicad_mod (+ .step для 3-D).
4. 90 % задачи закрывается тем, что берём готовые символ и footprint из стандартных библиотек KiCad, а в .ato только связываем пины и прописываем параметры.
5. Есть еще мой репозиторий, заранее извините, там не убрано, но можете попробовать взять от туда, что-то полезное [GIT](https://github.com/DmitriyKunitsin/Atopile)
6. Если этого мало, то читаем ниже, удачи.

# Разработка печатных плат с помощью кода.
## Разрабатывайте аппаратное решение как программное обеспечение 
## ```Atopile``` - это язык, компилятора и набор инструментов для электроники

- # ```Установка``` - Для работы рекомендуется использовать IDE ```VS Code```, достаточно установить расширение с названием **atopile**, либо перейти по [ссылке](https://marketplace.visualstudio.com/items?itemName=atopile.atopile). Инструкцию по установке можете найти по [ссылке](https://docs.atopile.io/atopile/guides/install). Так же для работы потребуется установить на компьютер ПО ```KiCAD``` [ссылка](https://www.kicad.org/download/)


- # ```Быстрый старт```
    1. Устанавливаем расширение на **VS Code**
    2. Зажимаем набор клавишь **Ctrl + P** и вводе в поле **>atopile: Open Example Project**
    3. Выбираем любой понравившийся

- # ```Через коммандную строку```
- Если после установки ato команда не находится – добавьте %LOCALAPPDATA%\pipx\bin в PATH вручную; pipx не всегда прописывает.
    1. Переходите по нужному пути, где желаете создать проект
    2. Выполняете команду ```ato create```
    3. Заполняте опросник, при выборе лицензии, рекомендуется выбирать 1 **MIT license**
    4. Создастся дефолтный проект с пустым файлом **main.ato**


- # ```Структура проекта```
### Проект состоит из нескольких основных папок и файлом, в их число входит:
1. **layouts** - тут лежат необходимые для работы KiCAD файлы по каждому указанному в **ato.yaml** профилю
2. **parts** - папка, в которой лежат все добавленный личные или подгруженные пакеты с элементами
3. **ato.yaml** - файл конфигурации
4. **main.ato** - основной файл, который компилируется и подтягивает все связанные файлы

### Расширения файлов в папке **parts**, кто за что отвечает
|Расширение|Что внутри|Кто читает|Зачем|
|-------|------|------|--------|
|**.step**|3-D-модель корпуса.|KiCad 3-D Viewer, сторонние MCAD|Показывает «кубик» при просмотре платы в KiCad → «3D-Viewer» и при экспорте в FreeCAD/Step. ato пока не трогает, но кладёт рядом, чтобы путь прописать в .kicad_mod.|
|**.kicad_mod**|Файл одной посадочной площадки (footprint). 3-D-ссылка, центр, слои и т.д.|KiCad PCB Editor + ato (через footprint = "parts/mylib.pretty/TQFP-32.kicad_mod")|Скрипт сборки ato копирует *.pretty в build/kicad/libs, KiCad подхватывает и ставит именно этот footprint на плату.|
|**.kicad_sym**|Библиотека символов (один файл = много символов). S-expr: выводы, графика, ссылки на footprint/SPICE|KiCad Schematic Editor|Чтобы на схеме KiCad resistor показывался как «R» с двумя выводами. ato сам рисовать символы не умеет, поэтому: 1) берём готовый .kicad_sym из официальных библиотек KiCad; 2) импортируем в проект; 3) в ato.yaml указываем kicad_symbol: Device:R. ato при сборке подсунет KiCad нужный символ.|
- ```Лайфхацк```В целом если делаете, что-то уникальное для **atopile** , то 90% эти файлы можно взять в KiCAD, для отображения элемента на схеме, а логику его прописать в файле **.ato**

# Для добавления готового элемента из **Package** atopile, надо произвести манипуляции
1. Выбрать небходимый
2. в командной строке ввести **ato install [имя нужного пакета]**
- как пример 
    ```
    ato add atopile/buttons
    ```
3. Подключить в своём проекте ( в файле main.ato)
- Пример
    ```
    """Brand new atopile project!"""

    #pragma experiment("BRIDGE_CONNECT")

    import Electrical

    from "atopile/buttons/buttons.ato" import VerticalButton
    from "atopile/buttons/buttons.ato" import HorizontalButton

    module App:
        btn_bridge = new VerticalButton
        signal_a = new Electrical
        signal_b = new Electrical
        # Если кнопка нажата, signal_a соединен с signal_b
        signal_a ~> btn_bridge ~> signal_b
    ```

# Можно задать размеры платы прямо в .ato:
```
module MyBoard:
    signal gnd
    signal vcc
    layout = Board(
        outline = Rect(width=50 mm, height=30 mm),
        layers = 2,
        via_style = "th"
    )
```
- При ato build автоматически сгенерируется *.kicad_pcb с нужным контуром – не трогайте руками, иначе на следующий билд изменения сотрутся.

# Создание своего элемента ```74HC595```, которого по какой-то причине не нашли в доступных пакетах
1. Заимствуем в KiCAD подходящие файлы в данном примере это файл **SO-16_3.9x9.9mm_P1.27mm.step** по пути **KiCad\9.0\share\kicad\3dmodels\Package_SO.3dshapes**, данный файл необходим для отображения 3-д модели
2. Заимствует так же файл для отображения посадочной площадки элемента файл **SOIC-16_3.9x9.9mm_P1.27mm.kicad_mod** по пути **KiCad\9.0\share\kicad\demos\kit-dev-coldfire-xilinx_5213\kit-dev-coldfire.pretty**
3. Создаём в папке **patrs** папку с нашим новым элементом, в данном примере этот папка **ShiftReg**(делаем сдиговый регистр 74HC595) и кладём позаимствованые файлы туда
4. Создаём вновь созданной папке файл с расширением **.ato** с названием **PINE_SHIFT_74HC595_driver**, он будет выполнять у нас роль некого драйвера, который будет описывать скольки пинов у элемента, какая нумерация и какой footprint взять, а так же модель, что мы заранее скачали
- ```Пример файла```
    ```
    #pragma experiment("TRAITS")

    import is_atomic_part 

    component PINE_SHIFT_74HC595_package:
    # Атрибуты manufacturer и mpn вместе полностью определяют, какой это компонент. 
    # Если вы создаете класс компонента для конкретного компонента, обычно рекомендуется включить как минимум либо lcsc_id, либо mpn + manufacturer.
    
    trait is_atomic_part< manufacturer="Kunitsin", partnumber="74HC595", footprint="SOIC-16_3.9x9.9mm_P1.27mm.kicad_mod", symbol="74xx:74HC595", model="SO-16_3.9x9.9mm_P1.27mm.step">
    # pins
    pin 1
    pin 2
    pin 3
    pin 4
    pin 5
    pin 6
    pin 7
    pin 8
    pin 9
    pin 10
    pin 11
    pin 12
    pin 13
    pin 14
    pin 15
    pin 16
    ```
5. В файле проекта создаём новый файл c расширением **.ato**, либо можем добавить весь код в наш **main.ato**, в данном случае создаётся файл для вынесения этого регистра как отдельный подключаемый элемент. Создали файл с именем **ShiftReg.ato**
- ```Код файла```
    ```
    #pragma experiment("TRAITS")

    import Electrical
    from "parts\ShiftReg\PINE_SHIFT_74HC595_driver.ato" import PINE_SHIFT_74HC595_package
    from "atopile/buttons/buttons.ato" import HorizontalButton

    module ShiftRegister:
        """
        Simple shift regulator 74HC595 fpr lesson
        """
        # Inputs
        ser_d_in = new Electrical  # Serial data in
        shift_clk = new Electrical  # Shift clock
        latch = new Electrical  # Latch clock
        oe = new Electrical  # Output enable
        srclr = new Electrical  # Shift clear

        # Outputs
        q_out_data = new Electrical  # Serial out
        q0 = new Electrical  # Parallel out 0
        q1 = new Electrical
        q2 = new Electrical
        q3 = new Electrical
        q4 = new Electrical
        q5 = new Electrical
        q6 = new Electrical
        q7 = new Electrical  # Parallel out 7
        
        # Power
        vcc = new Electrical
        gnd = new Electrical

        module ShiftReg from Shift_reg_driver:
        
        
        

        #    Подключение физически описанных контактов драйвера к электрическим
        module Shift_reg_driver from ShiftRegister:
        package = new PINE_SHIFT_74HC595_package
        ser_d_in ~ package.14
        shift_clk ~ package.11
        latch ~ package.12
        oe ~ package.13
        srclr ~ package.10
        q_out_data ~ package.9
        q0 ~ package.15
        q1 ~ package.1
        q2 ~ package.2
        q3 ~ package.3
        q4 ~ package.4
        q5 ~ package.5
        q6 ~ package.6
        q7 ~ package.7
        vcc ~ package.16
        gnd ~ package.8
    ```
6. В основном файле **main.ato** просто подключаем наш сдвиговый регистр
- ```main.ato```
    ```
    #pragma experiment("BRIDGE_CONNECT")
    #pragma experiment("FOR_LOOP")

    import Electrical
    import ElectricPower
    import Resistor

    from "atopile/buttons/buttons.ato" import HorizontalButton
    from "ShiftReg.ato" import Shift_reg_driver

    module Usage:
        power = new ElectricPower

        btn_pullup = new HorizontalButton
        pullup_resistor = new Resistor
        pullup_signal = new Electrical

        pullup_resistor.resistance = 10kohm +/- 5%
        pullup_resistor.package = "0402"
        
        power.hv ~> pullup_resistor ~> pullup_signal
        pullup_signal ~> btn_pullup ~> power.lv

        shiftRegisterCustom = new Shift_reg_driver
        power.hv ~> shiftRegisterCustom.vcc
        power.lv ~> shiftRegisterCustom.gnd
    ```

# С ```#pragma``` можно ознакомиться в ```CLAUDE.md```
# Где почитать документацию
1. [ссылка](https://docs.atopile.io/atopile/quickstart)
2. [гит с пакетами](https://github.com/atopile/packages)
3. ```CLAUDE.md``` в созданном вами проекте
# Где спросить, если застряли
[ссылка](https://github.com/atopile/atopile/discussions)