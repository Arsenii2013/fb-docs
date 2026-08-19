# Blink

На данном этапе еще не надо ставить процессор.  

- Создаем проект в Vivado. Выбираем RTL project.
- Пишем blink.sv. Создаем через File - Add Sources... - Add source file - Create file. Затем надо указать в настройках проекта что blink — top модуль. Tools - Settings... - General - Top module name.  
- Пишем тестбенч blinkTB.sv. Создаем через File - Add Sources... - Add simulation file - Create file.
    - Для симуляции Flow navigator - Run simelation - Run behavioral simulation. Правда так симулируется не в questa а во встроеном симуляторе.  
- Затем пишем констрэины,  File - Add Sources... - Add constrain file - Create file, файлы xdc.  
    - Pin assignment:  
    ```
    set_property -dict { PACKAGE_PIN P14   IOSTANDARD LVCMOS33 } [get_ports { gpio[0] }];
    ```
    - Клок:  
    ```
    set_property -dict { PACKAGE_PIN H16   IOSTANDARD LVCMOS33 } [get_ports { PL_clk }];  
    create_clock -add -name sys_clk_pin -period 8.00 -waveform {0 4} [get_ports { PL_clk }];
    ```  
- Протыкиваем все шаги в Flow Navigator. Загружаем в ПЛИС через Hardware Manager.

## Материалы

- [Pin assignmemt для PINQ Z1](PYNQ-Z1_C.xdc)